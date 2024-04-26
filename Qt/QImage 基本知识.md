
灰度图像是一种图像类型，其中每个像素的颜色信息只包含灰度（亮度）信息，而不包含色彩信息。在灰度图像中，像素值的范围通常从0（黑色）到255（白色）。灰度图像通常用于处理不需要颜色信息的图像，例如文本识别或边缘检测。

除了灰度图像，还有以下几种常见的图像类型：

1. 二值图像：每个像素只有两种可能的值，通常是0（黑色）和1（白色）。二值图像通常用于处理需要明确区分前景和背景的图像，例如二维码或条形码。
    
2. 彩色图像：每个像素包含红、绿、蓝三种颜色的信息。这是最常见的图像类型，用于显示彩色照片或图像。
    
3. 索引颜色图像：每个像素的值是颜色表中的索引。索引颜色图像可以用于显示有限数量的颜色，而且文件大小比彩色图像小。
    
4. RGBA图像：每个像素包含红、绿、蓝三种颜色的信息，以及一个alpha通道，用于表示像素的透明度。
    

这些都是常见的图像类型，但还有许多其他类型的图像，例如深度图像、HDR图像等。

显示二维码的Demo
```cpp
    // qrgencode gen qr and show 
    QString username = ui->username->text();

    // 二维码有四个纠错等级，从低到高：L-%7/M-%15/Q-%25/H-%30
    const qrcodegen::QrCode::Ecc level = qrcodegen::QrCode::Ecc::MEDIUM;

    // 生成二维码

    try {
        // 使用简易接口，默认 Version 范围 min=1, max=40
        // const qrcodegen::QrCode qr = qrcodegen::QrCode::encodeText(text, level);
        // 也可以指定符号版本，越大可容纳的信息越多，如果内容超出容量范围会抛异常
        // 根据规范，Version1 是 21x21，Version2 是 25x25，每增加一个Version，就比前一版本每边增加 4 个模块
        std::vector<qrcodegen::QrSegment> segs = qrcodegen::QrSegment::makeSegments(username.toUtf8().constData());
        const qrcodegen::QrCode qr = qrcodegen::QrCode::encodeSegments(segs, level, 1, 40);
        const int size = qr.getSize();
        // 显示二维码  二值图像
        QImage im(size, size, QImage::Format_Mono);

        for (int row = 0; row < size; ++row)
        {
            for (int col = 0; col < size; ++col)
            {
                im.setPixel(row, col, qr.getModule(row, col));
            }
        }
        im = im.scaled(250, 250);

        QLabel *label = new QLabel(this);
        label->setPixmap(QPixmap::fromImage(im));

        QDialog *dialog = new QDialog(this);
        QVBoxLayout *layout = new QVBoxLayout(dialog);
        layout->addWidget(label);
        dialog->setLayout(layout);

        dialog->setWindowTitle("QRCode");
        dialog->show();
    } catch (qrcodegen::data_too_long e) {
        QMessageBox::warning(this, "QRCode", "The text is too long to encode as a QR Code");
    }

```

### 如何使用 qrcodegen 库

qrcodegen.cpp 中的 encodeSegments如何使用， 为什么它要先将 text 转成 QrSegment 数组？

`qrcodegen::QrCode::encodeSegments`函数用于生成二维码。它接受一个`QrSegment`数组作为参数，这个数组表示要编码的数据。

`QrSegment`是二维码库中的一个类，它表示二维码中的一个数据段。一个数据段可以是数字、字母、字节或日本的Kanji字符。`QrSegment::makeSegments`函数可以将一个字符串转换为一个`QrSegment`数组。

将文本转换为`QrSegment`数组的原因是，二维码可以包含多种类型的数据，而不仅仅是文本。例如，你可以在一个二维码中包含一个网址和一个电话号码。每种类型的数据都有一个对应的`QrSegment`。

在你的代码中，你首先使用`QrSegment::makeSegments`函数将用户名转换为一个`QrSegment`数组。然后，你使用`qrcodegen::QrCode::encodeSegments`函数将这个数组编码为一个二维码。

以下是一个示例，它在一个二维码中包含了一个网址和一个电话号码。每种类型的数据都有一个对应的`QrSegment`。

```cpp
#include <qrcodegen/QrCode.hpp>
#include <vector>

int main() {
    // 网址
    std::string url = "https://www.example.com";
    qrcodegen::QrSegment seg1 = qrcodegen::QrSegment::makeBytes((uint8_t*)url.c_str(), url.length());

    // 电话号码
    std::string phone = "1234567890";
    qrcodegen::QrSegment seg2 = qrcodegen::QrSegment::makeNumeric(phone.c_str(), phone.length());

    // 创建QrSegment数组
    std::vector<qrcodegen::QrSegment> segs;
    segs.push_back(seg1);
    segs.push_back(seg2);

    // 生成二维码
    const qrcodegen::QrCode qr = qrcodegen::QrCode::encodeSegments(segs, qrcodegen::QrCode::Ecc::MEDIUM);

    // 输出二维码
    std::cout << qr.toSvgString(4) << std::endl;

    return 0;
}
```



```cpp
// Define custom colors
QRgb foreground = qRgb(0, 0, 255); // Blue for QR code
QRgb background = qRgb(255, 255, 0); // Yellow for background

QImage im(size, size, QImage::Format_RGB32);

for (int row = 0; row < size; ++row)
{
    for (int col = 0; col < size; ++col)
    {
        im.setPixel(row, col, qr.getModule(row, col) ? foreground : background);
    }
}
im = im.scaled(250, 250);

QLabel *label = new QLabel(this);
label->setPixmap(QPixmap::fromImage(im));
label->setScaledContents(true);

QDialog *dialog = new QDialog(this);
QVBoxLayout *layout = new QVBoxLayout(dialog);
layout->addWidget(label);
dialog->setLayout(layout);
```