### 在realme手机上调试Android程序
今天使用“真我”手机（一个国产品牌realme）进行调试Android项目，run后出现了如下错误。  
而在华为手机上进行调试，没有出现问题。

问题整体描述：Installation did not succeed.  
The application could not be installed: INSTALL_FAILED_TEST_ONLY  
Installation failed due to: ‘null’

### 解决

在项目根目录gradle.properties中，增加配置：

`android.injected.testOnly=false`