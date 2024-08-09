
[Windows Qt 程序崩溃重启\_qt程序异常结束自动重启-CSDN博客](https://blog.csdn.net/qq_20088499/article/details/100696867)

```cpp
#include <QApplication>
#include <QDebug>
#include <QSettings>
#include "testWindow.h"
#include "aboutwindow.h"
#include <iostream>
#include <string>
#include <QFile>

int main(int argc, char** argv)
{
    // 检查是否传递了 --restart 参数
    std::string arg;
    if (argc == 2) {
        arg = argv[1];
    }

    bool isRestart = false;

    if (arg == "--restart") {
        std::cout << "test.exe restart。\n";
        isRestart = true;
    }
    else {
        std::cout << "normal start";
    }

    QApplication a(argc, argv);

    // last window close not quit
    a.setQuitOnLastWindowClosed(false);

#if 0
    testWindow w;
	// 检查是否由守护进程启动
	if (isRestart) {
        w.showWindow();
	}
	else {
		w.showNormal();
	}
#else
    aboutWindow w;
    w.show();
#endif
    return a.exec();
}
```

```cpp
#include <QtCore/QCoreApplication>
#include <QProcess>
#include <QDebug>
#include <QTimer>
#include <QSettings>

QSettings settings("YourCompany", "daemon");

void startProcess(QProcess* process, const QStringList& arguments = QStringList()) {
    QString applicationDir = QCoreApplication::applicationDirPath();
    QString executablePath = applicationDir + "/test.exe"; // 构建 test.exe 的完整路径
    process->start(executablePath, arguments);
}

int main(int argc, char* argv[])
{
    QCoreApplication a(argc, argv);

    QProcess* process = new QProcess(&a);
    settings.setValue("restart", false);
    startProcess(process); // 首次启动不带 --restart 参数

    QObject::connect(process, static_cast<void(QProcess::*)(int, QProcess::ExitStatus)>(&QProcess::finished),
        [&process, &a](int exitCode, QProcess::ExitStatus exitStatus) {
            //if (exitStatus == QProcess::CrashExit || exitCode != 0) {
            if (1) {
                qDebug() << "test.exe abnormal exit，restarting...";
                qDebug() << "exitCode: " << exitCode;
				settings.setValue("restart", true);
                startProcess(process, { "--restart" }); // 异常退出时重启并传递 --restart 参数
            }
            else {
                qDebug() << "test.exe normal exit，daemon exiting...";
                qDebug() << "exitCode: " << exitCode;
                a.quit(); // test.exe 正常退出时，结束守护进程的事件循环
            }
        });
#if 0
    // 创建并配置 QTimer
    QTimer* timer = new QTimer(&a);
    timer->setInterval(3000); // 设置时间间隔为 3000 毫秒（3秒）
    QObject::connect(timer, &QTimer::timeout, []() {
        qDebug() << "detecting....";
        });
    timer->start(); // 启动定时器
#endif

    return a.exec();
}
```