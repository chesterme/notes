1. 建立一个maven web项目
![1569222399542](img\1569222342576.png)

2. 设置ground-id和artifact-id
+ ground-id: 项目组的唯一标识，可以填写主页的逆序,b比如：com.learn.me
+ artifact-id: 项目的唯一标识，比如：myapp-hello

4. 设置项目的编译器版本
![001](img\001.png)

5. 设置项目的目录结构
```
-src // 源代码和测试代码的根目录
    -main // 主要目录
        -java // 源代码目录
        -resources // 资源目录
        -webapp // 包含Java Web应用程序的目录
        -config // 配置文件目录
        -scripts // 脚本目录
    -test // 测试目录
        -java
        -resources
```

6. 设置部署
![003](img/003.png)
+ 设置Java Web应用部署描述文件的路径
+ 设置Java Web应用资源文件目录

ps: 什么是Maven Artifacts？
> An artifact is a file, usually a JAR, that gets deployed to a Maven repository.

> A Maven build produces one or more artifacts, such as a compiled JAR and a "sources" JAR.

> Each artifact has a group ID (usually a reversed domain name, like com.example.foo), an artifact ID (just a name), and a version string. The three together uniquely identify the artifact.

> A project's dependencies are specified as artifacts.

ps: 什么是Maven facets？

7. 设置项目如何打包
![004](img/004.png)

8. 设置tomcat运行环境
![005](img/005.png)
![006](img/006.png)

9. idea没有办法找到映射器
原因：idea不会编译src中java目录下的xml文件，故在mybatis配置文件中不能找到映射器文件

解决办法1：
> 将映射器文件放置在资源文件目录下

解决办法2：
> 在maven的pom文件中添加：

```xml
 <resources>
    <resource>
        <directory>src/main/java</directory>
        <includes>
            <include>**/*.xml</include>
        </includes>
    </resource>
</resources>
```