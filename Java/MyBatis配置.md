## 配置文件元素
```xml
<!--配置-->
<configuration>
    <!-- 属性 -->
    <properties/>
    <!-- 设置 -->
    <settings/>
    <!-- 类型命名 -->
    <typeAliases>
        <!-- 别名 -->
        <typeAlias alias="role" type="ch03.pojo.Role"/>
    </typeAliases>
    <!-- 类型处理器 -->
    <typeHandlers/>
    <!-- 对象工厂 -->
    <objectFactory/>
    <!-- 插件 -->
    <plugins/>
    <!-- 配置环境 -->
    <environments default="development">
        <!-- 环境变量 -->
        <environment id="development">
            <!-- 事务管理器 -->
            <transactionManager type="JDBC"/>
            <!-- 数据源 -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/ssm?characterEncoding=utf-8&amp;serverTimezone=GMT%2B8"/>
                <property name="username" value="test"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
    <!-- 数据库厂商标识 -->
    <databaseIdProvider/>
    <!-- 映射器 -->
    <mappers>
        <mapper resource="ch03/mapper/RoleMapper.xml"/>
    </mappers>
</configuration>
```

## properties属性
1. 可以在properties属性中设置变量
```xml
<properties>
    <property name="database.driver" value="com.mysql.cj.jdbc.Driver"/>
    <property name="database.url" value="jdbc:mysql://localhost:3306/ssm?characterEncoding=utf-8&amp;serverTimezone=GMT%2B8"/>
    <property name="database.username" value="test"/>
    <property name="database.password" value="123456"/>
</properties>
```
2. 可以引用外部properties文件
```xml
<properties resource="jdbc.properties"></properties>
```
## typeHandler类型转换器
在数据库类型和java类型之间进行相互转换
### 自定义类型转换器
1. 实现*TypeHandler*接口或者继承*BaseTypeHandler*抽象类
```java
public class MyTypeHandler implements TypeHandler<String> {

    Logger logger = Logger.getLogger(MyTypeHandler.class);

    @Override
    public void setParameter(PreparedStatement ps, int i, String parameter, JdbcType jdbcType) throws SQLException {
        logger.info("设置string参数[" + parameter + "]");
        // 在sql预处理语句中将第i个参数上的值设置成parameter的值
        ps.setString(i, parameter);
    }

    @Override
    public String getResult(ResultSet rs, String columnName) throws SQLException {
        // 在ResultSet对象中取出当前行中指定列名的值，并将其转换成java中String类型
        String result = rs.getString(columnName);
        logger.info("读取string参数1 [" + result + "]");
        return result;
    }

    @Override
    public String getResult(ResultSet rs, int columnIndex) throws SQLException {
        // 在ResultSet对象中取出当前行中指定列的值，并将其转换成java中String类型
        String result = rs.getString(columnIndex);
        logger.info("读取string参数2 [" + result + "]");
        return result;
    }

    @Override
    public String getResult(CallableStatement cs, int columnIndex) throws SQLException {
        // 在结果中取出当前行中指定列的值，并将其转换成java中String类型
        String result = cs.getString(columnIndex);
        return null;
    }
}
```
2. 配置类型转换器
```xml
<!-- 类型处理器 -->
<typeHandlers>
    <typeHandler handler="ch04.handler.MyTypeHandler" javaType="string" jdbcType="varchar"/>
</typeHandlers>
```
### 使用自定义类型转换器
+ 指定与自定义typeHandler一致的jdbcType和javaType
```xml
<select id="findRoles" parameterType="string" resultMap="roleMapper">
    select id, role_name, note from t_role
    where role_name like concat('%', #{roleName, jdbcType=VARCHAR, javaType=string}, '%')
</select>
```
+ 直接使用typeHandler指定具体的实现类
```xml
<select id="findRoles2" parameterType="string" resultMap="roleMapper">
    select id, role_name, note from t_role
    where note like concat('%', #{note, typeHandler=ch04.handler.MyTypeHandler}, '%')
</select>
```

### 枚举类型转换器
+ EnumOrdinalTypeHandler: 根据枚举数组下标索引的方式进行匹配，要求数据库返回一个整数作为其下标，它会根据下标找到对应的枚举类型
+ EnumTypeHandler: 把使用的名称转换为对应的枚举

### blob字段转换为byte[]
1. 定义一个pojo
```java
public class TestFile {

    private long id;
    private byte[] content;

    // getter and setter
}
```
2. 定义一个接口
```java
public interface FileMapper {

    TestFile getFile(long id);
    void insertFile(TestFile content);

}
```
3. 定义映射器
```xml
<mapper namespace="ch04.mapper.FileMapper">
    <resultMap id="file" type="ch04.pojo.TestFile">
        <id column="id" property="id"/>
        <result column="content" property="content" typeHandler="org.apache.ibatis.type.BlobTypeHandler"/>
    </resultMap>
    <select id="getFile" parameterType="long" resultMap="file" databaseId="mysql">
        select id, content from t_file where id = #{id}
    </select>
    <insert id="insertFile" parameterType="ch04.pojo.TestFile" databaseId="mysql">
        insert into t_file(content) values (#{content})
    </insert>
</mapper>
```
4. 引入映射器
```xml
<mappers>
    <mapper resource="ch04/mapper/FileMapper.xml"/>
</mappers>
```

## 常见错误
1. jdbc中不存在*varchar*的枚举常量
```error
### Cause: org.apache.ibatis.builder.BuilderException: Error parsing SQL Mapper Configuration. Cause: org.apache.ibatis.builder.BuilderException: Error resolving JdbcType. Cause: java.lang.IllegalArgumentException: No enum constant org.apache.ibatis.type.JdbcType.varchar
```
修正：将所有*varchar*改为*VARCHAR*
2. MapperRegistry不知道UserMapper接口是用来关联映射器文件的
```error
org.apache.ibatis.binding.BindingException: Type interface ch04.mapper.UserMapper is not known to the MapperRegistry.
```
修正：在基础配置文件中加入映射器
```xml
<mappers>
    <mapper resource="ch04/mapper/UserMapper.xml"/>
</mappers>
```
3. 忘记提交事务
```error
org.apache.ibatis.transaction.jdbc.JdbcTransaction: Rolling back JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@f5ac9e4]
```
修正：在执行完sql语句之后，需要提交事务
```java
fileMapper.insertFile(file);
sqlSession.commit();
```
## 映射器
### 引入映射器的方法
1. 文件路径
```xml
<mappers>
    <mapper resource="ch04/mapper/UserMapper.xml"/>
</mappers>
```
2. 包名
```xml
<mappers>
    <package name="ch04.mapper">
</mappers>
```
3. 用类注册
```xml
<mappers>
    <mapper class="ch04.mapper.UserMapper"/>
</mappers>
```
4. 完全限定资源定位符
```xml
<mappers>
    <mapper url="file:///path/to/mapper.xml"/>
</mappers>
```
### 传递参数
1. 使用map接口传递多个参数
```xml
<select id="findRolesByMap" parameterType="map" resultType="role">
    select id, role_name as roleName, note from t_role
    where role_name like concat('%', #{roleName}, '%')
    and note like concat('%', #{note}, '%')
</select>
```
2. 使用注解
```java
List<Role> findRolesByAnnotation(@Param("roleName") String roleName, @Param("note") String note);
```
```xml
 <select id="findRolesByAnnotation" resultType="role">
    select id, role_name as roleName, note from t_role
    where role_name like concat('%', #{roleName}, '%')
    and note like concat('%', #{note}, '%')
</select>
```
3. 使用java bean
```java
public class RoleParams {
    
    private String roleName;
    private String note;

    // getter and setter
}
// 更改接口
List<Role> findRolesByBean(RoleParams roleParams);
```
```xml
<select id="findRolesByBean" resultType="role" parameterType="ch05.bean.RoleParams">
    select id, role_name as roleName, note from t_role
    where role_name like concat('%', #{roleName}, '%')
    and note like concat('%', #{note}, '%')
</select>
```

## 延迟加载
全局设置：
+ lazyLoadingEnable: 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 fetchType 属性来覆盖该项的开关状态
+ aggressiveLazyLoading：当开启时，任何方法的调用都会加载该对象的所有属性。 否则，每个属性会按需加载
```xml
<!-- 设置 -->
<settings>
    <!-- 是否选择延迟加载 -->
    <setting name="lazyLoadingEnabled" value="true"/>
    <!-- 控制是否采用层级加载 -->
    <setting name="aggressiveLazyLoading" value="false"/>
</settings>
```

局部设置：
fetchType: 它出现在级联元素(association,collection)，它存在两个值：
+ eager: 获取当前POJO后立即加载对应的数据
+ lazy：获取当前POJO后延迟加载对应的数据
```xml
<!-- 对工牌进行一对一级联，property对应Employee类中的属性，column对应sql的列，它作为参数传递给select对应的sql语句中 -->
<association property="workCard" column="id" fetchType="lazy" select="ch05.mapper.WorkCardMapper.getById"/>
<!-- 对工作任务进行一对多级联，property对应Employee类中的属性，column对应sql的列，它作为参数传递给select对应的sql语句中 -->
<collection property="employeeTaskList" column="id" fetchType="lazy" select="ch05.mapper.EmployeeTaskMapper.getById"/>
```
## 缓存
+ 一级缓存在SqlSession上缓存，二级缓存在SqlSessionFactory上缓存
+ 默认情况下，mybatis开启一级缓存，此时不需要对POJO对象进行可序列化
### 开启二级缓存
1. 全局设置
```xml
<!-- 设置 -->
<settings>
    <!-- 开启二级缓存 -->
    <setting name="cacheEnabled" value="true"/>
</settings>
```
2.. 在mapper文件中加入：
```xml
<cache/>
```
3. 在POJO中实现*Serializable*接口

### 其中会发生的一些问题
#### 未序列化的对象在已经序列化的对象中存储
```error
### Error committing transaction.  Cause: org.apache.ibatis.cache.CacheException: Error serializing object.  Cause: java.io.NotSerializableException: org.apache.log4j.Logger
```
原因出处：
```java
public class MyObjectFactory extends DefaultObjectFactory{

    private static final long serialVersionUID = 2398571423004161112L;
    // error
    Logger log = Logger.getLogger(MyObjectFactory.class);
    // 修改：改为静态变量，例如：
    // private static final Logger log = Logger.getLogger(MyObjectFactory.class);
    private Object temp = null;

    // ...
}
```
例子：
```java
public class Test1 implements Serializable {

    private static final long serialVersionUID = 7162955568868357363L;
    private Test2 test2;
    private int value;

    public Test1(Test2 test2, int value) {
        this.test2 = test2;
        this.value = value;
    }
}
public class Test2 {

    private String data;

    public Test2(String data) {
        this.data = data;
    }
}
public class WriteObject {

    public static void main(String[] args){

        try{
            ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("object.txt"));
            Test2 test2 = new Test2("test2");
            Test1 test1 = new Test1(test2, 1);
            oos.writeObject(test1);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

}
```
这里调用*ObjectOutputStream.writeObject*方法时，会发生以下错误：
```error
java.io.NotSerializableException
```
改正：
方法1： 将未序列化的对象改成静态对象
方法2： 序列化未序列化的对象

#### 缓存不起作用
```error
Cache Hit Ratio [ch05.mapper.User2Mapper]: 0.0
```
```java
SqlSession sqlSession = SqlSessionFactoryUtils.openSqlSession();
SqlSession sqlSession1 = SqlSessionFactoryUtils.openSqlSession();
User2Mapper mapper = sqlSession.getMapper(User2Mapper.class);
User2Mapper mapper2 = sqlSession1.getMapper(User2Mapper.class);
try{
    mapper.getById(1L);
    //改正： sqlSession.commit();
    mapper2.getById(1L);
    //改正： sqlSession2.commit();
}finally{
    if(sqlSession != null){
        sqlSession.close();
    }
    if(sqlSession1 != null){
        sqlSession1.close();
    }
}
```
原因：
在排除缓存配置问题之后，每次查询不提交事务，这样一来，查询结果不会被缓存到*SqlSessionFactory*中

