## 存储概述
Android提供了几种保存application持久化数据的选择。而开发者根据
-   数据是否为app私有
-   数据是否可以暴露给其他app
-   数据大小情况

![[Android存储分类.png]]

### 存储方式

#### 内部存储
---------
内部存储位于系统中很特殊的一个位置（类似**沙盒**），如果你想将文件存储于内部存储中，那么文件默认只能被你的应用访问到，且一个应用所创建的所有文件都在和应用包名相同的目录下。  
也就是说应用创建于内部存储的文件，与这个应用是关联起来的。当一个应用卸载之后，内部存储中的这些文件也被删除。 **内部存储空间十分有限**，因而显得可贵，另外，它也是系统本身和系统应用程序主要的数据存储所在地，一旦内部存储空间耗尽，手机也就无法使用了。所以对于内部存储空间，我们要尽量避免使用。**Shared Preferences和SQLite数据库都是存储在内部存储空间上的。内部存储一般用`Context`来获取和操作。**

##### 创建并写入内部存储
安卓还为我们提供了一个简便方法`Context`类的 `openFileOutput()`来读写应用在内部存储空间上的文件
```java
String FILENAME = "hello_file";
String string = "hello world!";
FileOutputStream fos = openFileOutput(FILENAME, Context.MODE_PRIVATE);
fos.write(string.getBytes());
fos.close();

Context.fileList()
Context.deleteFile(filename)
```

`openFileOutput()`或者`Context.getFileDir()`通过Android Device Monitor工具可以找到此目录，为`/data/data/com.xxx.xxx/files/`。

##### 内部存储总结
- 文件操作只需要向函数提供文件名，所以程序自己只需要维护文件名即可；
- 不用自己去创建文件对象和输入、输出流，提供文件名就可以返回File对象或输入输出流
- 对于路径操作返回的都是文件对象。
- 由`Context`类调用

#### 外部存储
--------
所有的安卓设备都有外部存储和内部存储，这两个名称来源于安卓的早期设备，那个时候的设备内部存储确实是固定的，而外部存储确实是可以像U盘一样移动的。但是在后来的设备中，很多中高端机器都将自己的机身存储扩展到了8G以上，他们将存储在概念上分成了"内部internal" 和"外部external" 两部分，但其实都在手机内部。所以**不管安卓手机是否有可移动的sdcard，他们总是有外部存储**和内部存储。最关键的是，**我们都是通过相同的api来访问可移动的sdcard或者手机自带的存储（外部存储）**。

##### 获取权限
如果要访问外部存储：
```xml
<manifest ...>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    
android:requestLegacyExternalStorage="true"
    ...
</manifest>
```
Android 6.0后新加入了运行时权限，运行过程中需要动态申请权限。
  
##### 检查状态
在使用外部存储之前，你必须要使用`getExternalStorageState()`方法，先检查外部存储的当前状态，以判断是否可用：
```java
/* Checks if external storage is available for read and write */
public boolean isExternalStorageWritable() {
    String state = Environment.getExternalStorageState();
    if (Environment.MEDIA_MOUNTED.equals(state)) {
        return true;
    }
    return false;
}
/* Checks if external storage is available to at least read */
public boolean isExternalStorageReadable() {
    String state = Environment.getExternalStorageState();
    if (Environment.MEDIA_MOUNTED.equals(state) ||
        Environment.MEDIA_MOUNTED_READ_ONLY.equals(state)) {
        return true;
    }
    return false;
}
```

##### 外部存储公有文件
公共文件Public files：文件是可以被自由访问，**目录任意**，且文件的数据对其他应用或者用户来说都是由意义的，当应用被卸载之后，其卸载前创建的文件**仍然保留**。
```java
public File getAlbumStorageDir(String albumName) {
    // Get the directory for the user's public pictures directory.
    File file = new File(Environment.getExternalStoragePublicDirectory(
            Environment.DIRECTORY_PICTURES), albumName);
    if (!file.mkdirs()) {
        Log.e(LOG_TAG, "Directory not created");
    }
    return file;
}
```
目录为： /sdcard/data/

##### 外部存储私有文件
其实由于是外部存储的原因即是是这种类型的文件也能被其他程序访问，只不过一个应用私有的文件对其他应用其实是没有访问价值的（恶意程序除外）。外部存储上，应用私有文件的价值在于卸载之后，这些文件**也会被删除**。类似于内部存储。  
所有应用程序的外部存储的私有文件都放在根目录的`sdcard/Android/data/`下，目录形式为`sdcard/Android/data/<package_name>/`。

##### 一些文件路径获取方法
下图中的外部存储文件路径错误，应该为`/storage/emulated/0/Android/data/com.guo.firstapplication/files`
![[Pasted image 20230306031108.png]]
![[Pasted image 20230306032244.png]]

#### 外部存储路径
-------
##### 多种sd卡路径表示的区别总结
```
/sdcard/
/storage/sdcard0/
/storage/emulated/0/
/storage/emulated/legacy/
```
-  `/sdcard/`：只是一个符号链接，链接到`/storage/sdcard0/`
- `/storage/emulated/0/`：这是参照“emulated MMC”，通常指其内部，“0”代表第一个用户，即设备拥有者。 如果您创建其他用户，则此数字将为每个用户增加。
- `/storage/emulated/legacy/`：与上同理，但指向当前工作用户的部分。（对于“0”用户而言，这是`/storage/emulated/0/`的符号链接）
- `/storage/sdcard0/`：注意这里的“0”并非是一个单独文件夹名，而是作为后缀一样，意味着“0”并不代表用户，而是设备（卡）本身，因此它不需要legacy链接。人们可以通过OTG将读卡器与另一个SD卡连接起来，然后路径将成为`/storage/sdcard1/`。

##### sdcard0名称变化
**“sdcard”**这个文件夹是手机外置SD卡的文件夹名称，**“sdcard0”**这个文件夹是手机内存文件夹名称。手机内存本身是无法扩展的，但是外置SD卡的空间用户可以根据手机最大使用限制购买更换。因此**“sdcard0”**是内置储存，**“sdcard1”**是扩展储存，就是一般的sd卡。


#### 额外参考
[一篇文章搞懂android存储目录结构(维护更新) - 知乎](https://zhuanlan.zhihu.com/p/165140637)