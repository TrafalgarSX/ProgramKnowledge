### QT 托盘菜单
#### 如何让托盘菜单总是在左上显示

这个弹出显示的逻辑基于以下几个关键点：

**`trayIconMenu->popup(newPos);`的作用**：这行代码实际上是在告诉菜单应该在计算出的新位置newPos处显示。QMenu::popup()函数接受一个QPoint作为参数，**这个点指定了菜单左上角应该出现的屏幕位置**。

菜单的默认展示方式：通常，当你在某个位置（如鼠标点击的位置）弹出一个菜单时，菜单会从这个点向右下方展开。这是因为大多数图形用户界面（GUI）系统的坐标原点（(0, 0)点）位于屏幕的左上角，横坐标（X轴）向右增加，纵坐标（Y轴）向下增加。

调整菜单的弹出位置：通过计算一个新的位置newPos，使得菜单的右下角而不是左上角与鼠标点击的位置basePos对齐，可以改变菜单的展开方向。具体来说，通过从basePos的横纵坐标分别减去菜单的宽度和高度，我们得到了一个新的坐标点newPos。**这个新的坐标点实际上定义了菜单左上角的新位置，从而使得菜单的右下角与鼠标点击的位置对齐。**

为什么这样可以实现向左上展示：当菜单的右下角与鼠标点击的位置对齐时，菜单自然会向左和向上展开，因为它的展开方向是从左上角向右下角。这样，无论鼠标点击的位置在屏幕的哪个部分，菜单总是尝试向屏幕的内部展开，避免超出屏幕边界。


总结来说，这种方法通过智能地调整菜单的弹出位置，改变了菜单的默认展开行为，从而实现了总是向左上方向展开的效果，这对于提升用户体验和适应特定的布局需求非常有用。
```cpp
void Window::onTrayIconActivated(QSystemTrayIcon::ActivationReason reason)
{
    if (reason == QSystemTrayIcon::Trigger || reason == QSystemTrayIcon::Context) {
        // 获取当前鼠标的位置作为基准点
        QPoint basePos = QCursor::pos();

        // 获取菜单的预期大小
        QSize menuSize = trayIconMenu->sizeHint();

        // 计算新的位置，使菜单总是向左上方向展开
        QPoint newPos = basePos - QPoint(menuSize.width(), menuSize.height());

        // 确保菜单不会显示在屏幕外
        QRect screenRect = QGuiApplication::primaryScreen()->geometry();
        newPos.setX(qMax(newPos.x(), screenRect.left()));
        newPos.setY(qMax(newPos.y(), screenRect.top()));

        // 在计算出的新位置显示菜单
        trayIconMenu->popup(newPos);
    }
}
    // 连接activated信号到自定义槽函数
    connect(trayIcon, &QSystemTrayIcon::activated, this, &Window::onTrayIconActivated);
```
### 如何自定义托盘菜单

[用Qt写软件系列四：定制个性化系统托盘菜单 - 24K纯开源 - 博客园](https://www.cnblogs.com/csuftzzk/p/CustomSystemTray.html)