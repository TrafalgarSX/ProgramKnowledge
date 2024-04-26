本文将对如下问题进行释疑：

1. 为何Qt应用要在`main()`函数中以`QCoreApplication app(argc, argv)`开始，并以`return app.exec()`结束？
2. 同为循环，为何`while(true)`会导致CPU满载？而Qt的事件循环不会。
3. 为何通过`QEventLoop::exec()`阻塞程序执行，可以避免程序卡死？
4. `QCoreApplication::postEvent()`与`QCoreApplication::sendEvent()`为何会有不阻塞和阻塞的差别？
5. `QCoreApplication::processEvents()`与`QCoreApplication::sendPostedEvents()`有何差别？

## 主事件循环

通常情况下，Qt应用程序的`main()`函数都会近似以下形式：

```cpp
int main(int argc, char *argv[])
{
    QCoreApplication app(argc, argv);
    ……
    return app.exec();
}
```

在`QCoreApplication app(argc, argv)`这一步，主要是创建并初始化了一个`QCoreApplication`对象。最主要的是设置一些线程相关的数据（`QThreadData`），比如事件调度器（`QAbstractEventDispatcher`）等。  
而`app.exec()`则是为了**启动一个事件循环来分发事件**。如果没有事件循环或事件循环没有启动，则对象永远不会接收到事件。  
由`QCoreApplication`启动的事件循环也叫作`主事件循环`。

在省略掉大部分代码后，我们可以清晰地看到`QCoreApplication::exec()`如何启动一个事件循环：

```cpp
int QCoreApplication::exec()
{
    ……
    QEventLoop eventLoop;
    ……
    int returnCode = eventLoop.exec();
    ……
    return returnCode;
}
```

## 事件循环

如下代码可见`QEventLoop::exec()`是通过循环不断地调用`QEventLoop::processEvents()`来分发事件队列中的事件。

```cpp
int QEventLoop::exec(ProcessEventsFlags flags = AllEvents))
{
    Q_D(QEventLoop);
    ……
    while (!d->exit.loadAcquire())
        processEvents(flags | WaitForMoreEvents | EventLoopExec);
    ……
    return d->returnCode.load();
}
```

而最终完成事件分发的是事件调度器。通过下面的代码我们可以看出，**事件调度器存在于各自线程相关数据中。也就是说每个线程都可以有、并且只使用自己专属的事件调度器。**

```cpp
bool QEventLoop::processEvents(ProcessEventsFlags flags)
{
    Q_D(QEventLoop);
    if (!d->threadData->hasEventDispatcher())
        return false;
    return d->threadData->eventDispatcher.load()->processEvents(flags);
}
```

到这里我们已经能够看出，为何在通过`QEventLoop::exec()`阻塞程序执行，程序却不会卡死了。因为`QEventLoop::exec()`开启了一个新的事件循环来分发事件，而且相同线程上的所有事件循环采用同一个事件调度器。

## 事件调度器

事件调度器就比较依赖于各个平台的实现了，各平台上的事件调度器实现都不尽相同。我们以`QEventDispatcherUNIX`为例进行分析。

```cpp
bool QEventDispatcherUNIX::processEvents(QEventLoop::ProcessEventsFlags flags)
{
    Q_D(QEventDispatcherUNIX);
    ……
    QCoreApplicationPrivate::sendPostedEvents(0, 0, d->threadData);
    ……
    timespec *tm = nullptr;
    ……
    d->pollfds.append(d->threadPipe.prepare());

    int nevents = 0;

    switch (qt_safe_poll(d->pollfds.data(), d->pollfds.size(), tm)) {
    ……
    }
    return (nevents > 0);
}

void QCoreApplicationPrivate::sendPostedEvents(QObject *receiver, int event_type,
                                               QThreadData *data)
{
    ……
    while (i < data->postEventList.size()) {
        ……
        const QPostEvent &pe = data->postEventList.at(i);
        ++i;
        ……
        QEvent *e = pe.event;
        QObject * r = pe.receiver;
        ……
        QCoreApplication::sendEvent(r, e);
    }
    ……
}
```

`QCoreApplicationPrivate::sendPostedEvents(0, 0, d->threadData)`会派发当前线程事件队列中所有的事件。那么当事件分发完毕后是否就立即进入下一个事件队列派发周期了呢？  
事实并非如此。在`QEventDispatcherUNIX`中，其成员`d（QEventDispatcherUNIXPrivate）`的成员`threadPipe（QThreadPipe）`通过`eventfd()`系统调用获取了一个文件描述符，并在`QEventDispatcherUNIX::processEvents()`中通过`qt_safe_poll()`等待其数据可读（`POLLIN`）。在此过程中，线程进入内核态等待事件发生（反正`poll()`调用怎么实现的咱也不知道，总之它就是不(那么)占CPU资源），自然不会占用CPU资源。  
实际上`qt_safe_poll()`还监控了通过`QSocketNotifier`等监控的文件描述符的状态。当这些文件描述符指定的事件发生时，也会退出内核态进入下一次事件循环。

## 清空事件队列

通过上面的分析并综合下述代码，我们知道`QCoreApplication::sendPostedEvents()`会清空事件队列并立即返回。

```cpp
void QCoreApplication::sendPostedEvents(QObject *receiver, int event_type)
{
    QThreadData *data = QThreadData::current();
    QCoreApplicationPrivate::sendPostedEvents(receiver, event_type, data);
}
```

而`QCoreApplication::processEvents()`相较于`QCoreApplication::sendPostedEvents()`相当于多了等待事件发生这一步。也就是说它在清空事件队列后不会立即返回。

```cpp
void QCoreApplication::processEvents(QEventLoop::ProcessEventsFlags flags)
{
    QThreadData *data = QThreadData::current();
    if (!data->hasEventDispatcher())
        return;
    data->eventDispatcher.load()->processEvents(flags);
}
```

## 投递事件

上面说到，`QEventDispatcherUNIX::processEvents()`会等待事件发生，从而退出内核态进入下一次事件循环。如下所示：

```cpp
void QCoreApplication::postEvent(QObject *receiver, QEvent *event, int priority)
{
    ……
    QThreadData * volatile * pdata = &receiver->d_func()->threadData;
    QThreadData *data = *pdata;
    ……
    data->postEventList.addEvent(QPostEvent(receiver, event, priority));
    ……
    QAbstractEventDispatcher* dispatcher = data->eventDispatcher.loadAcquire();
    if (dispatcher)
        dispatcher->wakeUp();
}
```

当通过`QCoreApplication::postEvent()`投递一个事件的时候，会将其加入到事件队列中，并唤醒事件调度器。具体来说，是向`eventfd()`系统调用获取的文件描述符写入数据，使其状态为数据可读（`POLLIN`）。

```cpp
void QEventDispatcherUNIX::wakeUp()
{
    Q_D(QEventDispatcherUNIX);
    d->threadPipe.wakeUp();
}

void QThreadPipe::wakeUp()
{
    if (wakeUps.testAndSetAcquire(0, 1)) {
#ifndef QT_NO_EVENTFD
        if (fds[1] == -1) {
            // eventfd
            eventfd_t value = 1;
            int ret;
            EINTR_LOOP(ret, eventfd_write(fds[0], value));
            return;
        }
#endif
        char c = 0;
        qt_safe_write(fds[1], &c, 1);
    }
}
```

## 发送事件

网上大多数教程把`postEvent()`、`sendEvent()`都叫发送事件，并分出了不阻塞和阻塞两种差别。事实上`postEvent()`、`sendEvent()`是Qt事件循环中两个不同的阶段。如果简化掉一切中间流程，可以认为`QCoreApplication::sendEvent()`是在直接调用接受者的`QObject::event()`方法（`QGuiApplication`及`QApplication`实现不尽相同）。

```cpp
inline bool QCoreApplication::sendEvent(QObject *receiver, QEvent *event)
{
    if (event)
        event->spont = false;
    return notifyInternal2(receiver, event);
}

bool QCoreApplication::notifyInternal2(QObject *receiver, QEvent *event)
{
    bool selfRequired = QCoreApplicationPrivate::threadRequiresCoreApplication();
    ……
    if (!selfRequired)
        return doNotify(receiver, event);
    return self->notify(receiver, event);
}

bool QCoreApplication::notify(QObject *receiver, QEvent *event)
{
    ……
    return doNotify(receiver, event);
}

static bool doNotify(QObject *receiver, QEvent *event)
{
    ……
    return receiver->isWidgetType() ? false : QCoreApplicationPrivate::notify_helper(receiver, event);
}

bool QCoreApplicationPrivate::notify_helper(QObject *receiver, QEvent * event)
{
    ……
    return receiver->event(event);
}
```

[Qt-事件循环\_qt的事件循环-CSDN博客](https://blog.csdn.net/mrbone11/article/details/124945582)

[Qt实用技能3-理解事件循环 - 知乎](https://zhuanlan.zhihu.com/p/72758194)