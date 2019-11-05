## MyBatis核心组件
+ SqlSessionFactoryBuilder: 它会根据配置或代码来生成SqlSessionFactory，采用Builder模式
+ SqlSessionFactory(工厂接口): 生成SqlSession，使用工厂模式
+ SqlSession(会话): 包含了面向数据库执行 SQL 命令所需的所有方法。你可以通过 SqlSession 实例来直接执行已映射的 SQL 语句
+ SqlMapper(映射器): 它由一个Java接口和XML文件(或注解)构成，需要给出对应的SQL和映射规则。它负责发送SQL去执行，并返回结果

## 使用XML构建SqlSessionFactory
+ 基础配置文件，配置一些最基本的上下文参数和运行环境
+ 映射文件，配置映射关系、sql、参数等信息

## 基础配置文件
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeAliases>
        <!-- 别名，这里可以使用role来替代ch03.pojo.Role完全限定 -->
        <typeAlias alias="role" type="ch03.pojo.Role"/>
    </typeAliases>
    <!-- 数据库环境 -->
    <environments default="development">
        <environment id="development">
            <!-- 配置事务管理器，这里采用MyBatis的JDBC管理方式 -->
            <transactionManager type="JDBC"/>
            <!-- 配置数据库，其中POOLED表示采用MyBatis内部提供的连接池 -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/ssm/"/>
                <property name="username" value="test"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
    <!-- 映射文件 -->
    <mappers>
        <!-- 表示需要引入哪些映射器 -->
    </mappers>
</configuration>
```

### 构建SqlSessionFactory
```Java
SqlSessionFactory sqlSessionFactory = null;
String resource = "mybatis-config.xml";
InputStream in;
try{
    in = Resources.getResourceAsStream(resource);
    sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
}catch(IOException e){
    e.printStackTrace();
}
```
ps：在构建的过程中可能会遇到一些问题：
```java
java.io.IOException: Could not find resource configs/mybatis-config.xml
```
原因：
1) configs/mybatis-config.xml文件不在编译后的输出文件中
2) configs已经设置为资源文件了，此时不在需要上级文件夹，idea会将configs目录下的所有文件同一放在编译后的输出目录的子节点下，不是子孙节点下。

## SqlSession接口
它的实现类有：
+ DefaultSqlSession： 应用于单线程
+ SqlSessionManager: 应用于多线程环境

它的作用有：
+ 获取Mapper接口
+ 发送SQL给数据库
+ 控制数据库事务

### 如何使用
1. 如何创建SqlSession接口的实现类
```java
SqlSession sqlSession = sqlSessionFactory.openSession();
```
2. 如何控制数据库事务
+ commit(): 提交事务
+ rollback()：回滚事务
```java
SqlSession sqlSession = null;
try{
    sqlSession = sqlSessionFactory.openSession();
    // some code...
    sqlSession.commit(); // 提交事务
}catch(Exception e){
    sqlSession.rollback(); // 发生异常，回滚事务
}finally{
    // 确保资源被顺利回收
    if(sqlSession != null){
        sqlSession.close();
    }
}
```

## 映射器
+ 它由一个接口和对应的XML文件(或注解)组成
+ 主要用于将SQL查询结果映射为一个POJO，或者将POJO的数据插入到数据库中，并定义一些关于缓存等的重要内容

### 如何使用xml文件构建一个映射器
1. 定义一个POJO，用于接收查询结果或者保存插入内容
```java
public class Role {

    private Long id;
    private String roleName;
    private String note;

    // getter and setter
}
```
2. 定义映射器接口
```java
public interface RoleMapper {
    /**
     * 从Role表中根据id选择对应的Role
     * @param id
     * @return
     */
    Role getRole(Long id);
}
```
3. 定义对应的XML文件，即接口中方法对应的sql语句
+ mapper元素中的namespace对应的是一个接口的全限定名，使得mybatis能够找到该接口
+ select元素中id标识这个sql语句，parameterType表示传递给sql语句中的参数类型，resultType表示sql语句结果封装的类型
+ mybtais在默认情况下提供自动映射，只要sql返回的列名能和POJO对应起来即可，这里该查询返回的列为: <id, roleName, note>，正好与Role对应起来
```xml
<!-- RoleMapper.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="ch03.mapper.RoleMapper">
    <select id="getRole" parameterType="long" resultType="role">
        select id, role_name as roleName, note from t_role where id = #{id}
    </select>
</mapper>
```
4. 设置mybatis基本的配置文件
```xml
    <!-- 映射文件 -->
    <mappers>
        <!-- 表示需要引入哪些映射器 -->
        <mapper resource="ch03.mapper.RoleMapper.xml">
    </mappers>
```

### 如何使用注解构建一个映射器
1. 通过注解实现映射器
```java
public interface RoleMapper2 {
    @Select("select id, role_name as roleName, note from t_role where id = #{id}")
    Role getRle(Long id);
}
```
2. 修改mybatis基本配置文件

方法1：
```xml
    <!-- 映射文件 -->
    <mappers>
        <!-- 表示需要引入哪些映射器 -->
        <mapper resource="ch03.mapper.RoleMapper2">
    </mappers>
```
方法2:
```java
// 数据库连接池信息
PooledDataSource dataSource = new PooledDataSource();
dataSource.setDriver("com.mysql.jdbc.Driver");
dataSource.setUsername("test");
dataSource.setPassword("123456");
dataSource.setUrl("jdbc:mysql://localhost:3306/ssm");
dataSource.setDefaultAutoCommit(false);
// 采用MyBatis的JDBC事务方式
TransactionFactory transactionFactory = new JdbcTransactionFactory();
Environment environment = new Environment("development", transactionFactory, dataSource);
// 创建Configuration对象
Configuration configuration = new Configuration(environment);
// 注册一个MyBatis上下文别名
configuration.getTypeAliasRegistry().registerAlias("roles", ch03.pojo.Role.class);
// 加入一个映射器
configuration.addMapper(ch03.mapper.RoleMapper2.class);
// 使用SqlSessionFactoryBuilder构建SqlSessionFactory
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
```

### sqlSession发送sql
1. 直接使用sqlSession发送
+ selectOne: 使用查询并只返回一个对象
+ 参数1：使用全限定名来定位一条sql语句
```java
Role role = (Role)sqlSession.selectOne("ch03.mapper.RoleMapper.getRole", 1L);
```

2. 使用Mapper接口发送
```java
RoleMapper mapper = sqlSession.getMapper(RoleMapper.class);
Role role = mapper.getRole(1L);
```

两者比较：
+ 使用Mapper接口发送，完全面向对象，更能体现业务的逻辑
+ 使用Mapper接口发送，ide会提示错误和校验，而使用sqlSession直接发送，只有在运行时才会知道是否发生错误

## 生命周期
### SqlSessionFactoryBuilder
+ 用于创建SqlSessionFactory，创建完成后，它就失去作用了

### SqlSessionFactory
+ 类似一个数据库连接池，用于创建SqlSession接口对象
+ 它的生命周期存在于mybatis的整个应用中
+ 在一般应用中，往往希望SqlSessionFactory作为一个单例被创建，在应用中共享使用

### SqlSession
+ 类似一个数据库连接，可以在一个事务中执行多条sql语句，然后通过它的commit、rollback等方法，进行提交或回滚事务。
+ 它存在于一个业务请求中，当处理完这个请求，应该关闭这个连接

### Mapper
+ 由SqlSession创建，它的最大生命周期至多和SqlSession保持一致
+ 它表示一个请求中的业务处理，所以它应该在一个请求中，一旦一个请求处理完毕，就应该废弃它


## 常见的错误
1. idea没有办法找到映射器

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
2. jdbc驱动问题
```error
Loading class `com.mysql.jdbc.Driver'. This is deprecated. The new driver class is `com.mysql.cj.jdbc.Driver'. The driver is automatically registered via the SPI and manual loading of the driver class is generally unnecessary.
```
修改：在mybatis基础配置文件中重置jdbc驱动
```xml
 <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
```

3. 服务器时钟问题
```error
### Error querying database.  Cause: java.sql.SQLException: The server time zone value 'ÖÐ¹ú±ê×¼Ê±¼ä' is unrecognized or represents more than one time zone. You must configure either the server or JDBC driver (via the serverTimezone configuration property) to use a more specifc time zone value if you want to utilize time zone support.
```
修改：
方法1： 在mybatis基础配置文件中重置数据库连接url，设置时钟
```xml
<property name="url" value="jdbc:mysql://localhost:3306/ssm?characterEncoding=utf-8&amp;serverTimezone=GMT%2B8"/>
```
