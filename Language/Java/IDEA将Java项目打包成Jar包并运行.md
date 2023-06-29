### 普通 Java 项目打包成 Jar 包并运行
---
1. 点击【File】→【Project Structure...】。
![[idea生成Jar第一步.png]]

2. 点击【Artifacts】→【+】→【JAR】→【From modules with dependencies...】。
![[生成包含依赖的Jar包.png]]

Empty生成的jar包中不包含依赖（这种jar包只能用来工程引入，不能单独执行，因为缺少jar包）， 需要引入jar包的工程自己导入相关的依赖。

3. 点击【Main Class】后的【文件夹图标】→【OK】。如果你的项目是多模块项目，那么可以通过设置【Module】来选择要打包的模块。

![[IDEA将Java项目打包第三步.png]]

4. 选中【extract to the target JAR】，点击【OK】。
![[IDEA将Java项目打包第4步.png]]

5. 最后 build 项目即可生成Jar包。

6. 运行Jar包， `java -jar test.jar`

#### 可能出现的问题
运行上面的Jar包可能出现下面的问题：

```bash
Exception in thread "main" java.lang.SecurityException: Invalid signature file digest for Manifest main attributes
```

这是因为Jar依赖的Jar中有一些签名文件，和这个Jar包的签名文件冲突了。最简单的解决办法就是，用下面的命令删除掉jar包中的相关的文件。
```bash
zip -d yourjar.jar 'META-INF/.SF' 'META-INF/.RSA' 'META-INF/*SF'
```

如果你使用的是maven管理的idea工程，可以在pom.xml中正确的位置添加`<exclude>`块：
```xml
<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-shade-plugin</artifactId>
<version>2.4.2</version>
<executions>
    <execution>
        <phase>package</phase>
        <goals>
            <goal>shade</goal>
        </goals>
        <configuration>

            <filters>
                <filter>
                    <artifact>spark-streaming-twitter_2.10:spark-streaming-twitter_2.10</artifact>
                    <includes>
                        <include>**</include>
                    </includes>
                </filter>
                <filter>
                    <artifact>twitter4j-stream:twitter4j-stream</artifact>
                    <includes>
                        <include>**</include>
                    </includes>
                </filter>
                <filter>
                    <artifact>*:*</artifact>
                    <excludes>
                        <exclude>META-INF/*.SF</exclude>
                        <exclude>META-INF/*.DSA</exclude>
                        <exclude>META-INF/*.RSA</exclude>
                    </excludes>
                </filter>
            </filters>
        </configuration>
    </execution>
</executions>
</plugin>

```