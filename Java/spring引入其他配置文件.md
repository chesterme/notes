在*Spring*主配置文件中写入：
```xml
 <!-- 指定组件扫描基类 -->
<context:component-scan base-package="ch13.mapper, ch13.config, ch13.service"/>

<!-- 引入其他配置文件 -->
<import resource="classpath*:./ch13/config/transaction-config.xml"/>
```