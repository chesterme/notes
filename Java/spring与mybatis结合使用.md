## 配置*mybatis*
1. 总配置文件
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <!-- 设置 -->
    <settings>
        <!-- 是否选择延迟加载 -->
        <setting name="lazyLoadingEnabled" value="true"/>
        <!-- 控制是否采用层级加载 -->
        <setting name="aggressiveLazyLoading" value="false"/>
        <!-- 开启二级缓存 -->
        <setting name="cacheEnabled" value="true"/>
        <!-- 允许JDBC支持生成的键。需要适当的驱动。如果设置为true，则这个设置强制生辰的键被使用，尽管一些驱动拒绝兼容但仍然有效 -->
        <setting name="useGeneratedKeys" value="true"/>
        <!-- 配置默认的执行器 -->
        <setting name="defaultExecutorType" value="REUSE"/>
        <!-- 设置超时时间 -->
        <setting name="defaultStatementTimeout" value="25000"/>
    </settings>
    <typeAliases>
        <!-- 别名 -->
        <typeAlias alias="role" type="ch10.pojo.Role"/>
    </typeAliases>

    <!-- 映射文件 -->
    <mappers>
        <package name="ch12.mapper"/>
    </mappers>
</configuration>
```
2. 设置访问数据库的接口

```java
package ch12.mapper;

import ch03.pojo.Role;
import ch05.bean.PageParams;
import ch05.bean.RoleParams;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.session.RowBounds;

import java.util.List;
import java.util.Map;

public interface RoleMapper {
    /**
     * 从Role表中根据id选择对应的Role
     * @param id
     * @return
     */
    Role getRole(Long id);

    /**
     * 往Role表中插入一条记录
     * @param role
     * @return
     */
    int insertRole(Role role);

    /**
     * 根据id从Role表中删除一条记录
     * @param id
     * @return
     */
    int deleteRole(Long id);

    /**
     * 根据role对象从Role表中更新一条记录
     * @param role
     * @return
     */
    int updateRole(Role role);

    /**
     * 从Role表中查找所有同名的记录
     * @param roleName
     * @return
     */
    List<Role> findRoles(String roleName);

    /**
     * 可以通过多个参数进行查询
     * @param parameterMap
     * @return
     */
    List<Role> findRolesByMap(Map<String, Object> parameterMap);

    List<Role> findRolesByAnnotation(@Param("roleName") String roleName, @Param("note") String note);

    List<Role> findRolesByBean(RoleParams roleParams);

    List<Role> findByMix(@Param("params") RoleParams roleParams, @Param("page") PageParams pageParams);

    List<Role> findByRowBounds(@Param("note") String note, RowBounds rowBounds);
}
```
3. 设置实现访问数据库接口的mapper
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="ch12.mapper.RoleMapper">

    <resultMap id="roleMapper" type="role">
        <result property="id" column="id"/>
        <result property="roleName" column="role_name" jdbcType="VARCHAR" javaType="string"/>
        <result property="note" column="note" typeHandler="ch04.handler.MyTypeHandler"/>
    </resultMap>

    <sql id="roleCols">
        id, role_name, note
    </sql>

    <sql id="roleCols2">
        ${alias}.id, ${alias}.role_name, ${alias}.note
    </sql>

    <select id="getRole" parameterType="long" resultMap="roleMapper">
        select <include refid="roleCols"/> from t_role where id = #{id}
    </select>

    <insert id="insertRole" parameterType="role" useGeneratedKeys="true" keyProperty="id">
        insert into t_role(role_name, note) values(#{roleName}, #{note})
    </insert>
    <delete id="deleteRole" parameterType="Long">
        delete from t_role where id = #{id}
    </delete>

    <update id="updateRole" parameterType="role">
        update t_role set role_name = #{roleName}, note = #{note} where id = #{id}
    </update>

    <select id="findRoles" parameterType="string" resultMap="roleMapper">
        select
        <include refid="roleCols2">
            <property name="alias" value="r"/>
        </include>
        from t_role r
        where role_name like concat('%', #{roleName, jdbcType=VARCHAR, javaType=string}, '%')
    </select>

    <select id="findRoles2" parameterType="string" resultMap="roleMapper">
        select id, role_name, note from t_role
        where note like concat('%', #{note, typeHandler=ch04.handler.MyTypeHandler}, '%')
    </select>

    <select id="findRolesByMap" parameterType="map" resultType="role">
        select id, role_name as roleName, note from t_role
        where role_name like concat('%', #{roleName}, '%')
        and note like concat('%', #{note}, '%')
    </select>

    <select id="findRolesByAnnotation" resultType="role">
        select id, role_name as roleName, note from t_role
        where role_name like concat('%', #{roleName}, '%')
        and note like concat('%', #{note}, '%')
    </select>

    <select id="findRolesByBean" resultType="role" parameterType="ch05.bean.RoleParams">
        select id, role_name as roleName, note from t_role
        where role_name like concat('%', #{roleName}, '%')
        and note like concat('%', #{note}, '%')
    </select>

    <select id="findByMix" resultType="role">
        select id, role_name as roleName, note from t_role
        where note like concat('%', #{params.note}, '%')
        limit #{page.start}, #{page.limit}
    </select>

    <select id="findByRowBounds" resultType="role">
        select id, role_name as roleName, note from t_role
        where note like concat('%', #{note}, '%')
    </select>
</mapper>
```

## 配置*Spring*
1. 设置数据源
```xml
<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource">
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/ssm?characterEncoding=utf-8&amp;serverTimezone=GMT%2B8"/>
        <property name="username" value="test"/>
        <property name="password" value="123456"/>
        <!-- 连接池最大数据库连接数 -->
        <property name="maxOpenPreparedStatements" value="255"/>
        <!-- 最大等待连接中的数量 -->
        <property name="maxIdle" value="5"/>
        <!-- 最大等待毫秒数 -->
        <property name="maxWaitMillis" value="10000"/>
    </bean>
```

2. 让*Spring Ioc*容器能够找到*Mybatis*配置文件，并创建*SqlSessionFactory*对象
```xml
<!-- 配置SalSessionFactoryBean -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!-- 配置数据源 -->
    <property name="dataSource" ref="dataSource"/>
    <!-- 引入mybatis配置文件 -->
    <property name="configLocation" value="classpath:./ch12/config/mybatis-config.xml"/>
</bean>
```
> ps: 这里设置*mybatis*配置文件路径的时候会经常出现一些错误：
>找不到配置文件所在，原因有以下几个：
>1. *idea*集成环境的问题，在编译结束后，它不会把源代码目录下的资源文件移动到编译后的目录，这个问题可以通过修改*pom.xml*文件来修正
>2. 资源文件已经存在于编译后的目录下，但是如果通过相对路径来进行访问的话，也会出现找不到的情况，例如将上面的路径修改为：*classpath:ch12/config/mybatis-config.xml*

3. 可以构建一个*SqlSessionTemplate*对象，它是线程安全的，可以确保每个线程使用的*SqlSession*唯一且不相互冲突。
```xml
<!-- 配置SalSessionTemplate -->
<bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
    <constructor-arg ref="sqlSessionFactory"/>
</bean>
```

4. 配置*MapperFactoryBean*，可以从*Spring*容器中获取*Mapper*对象
```xml
<!-- 配置MapperFactoryBean -->
<bean id="roleMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
    <!-- RoleMapper接口将被扫描为Mapper -->
    <property name="mapperInterface" value="ch12.mapper.RoleMapper"/>
    <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
    <!-- 如果同时注入SqlSessionTemplate和SqlSessionFactory，则只会启动SqlSessionTemplate -->
<!--        <property name="sqlSessionTemplate" ref="sqlSessionTemplate"/>-->
</bean>
```
使用mapper对象
```java
ApplicationContext act = new ClassPathXmlApplicationContext("ch12/beans/Bean.xml");
RoleMapper roleMapper = act.getBean(RoleMapper.class);

Role role = new Role();
role.setRoleName("role_name_mapperFactoryBean");
role.setNote("note_mapperFactoryBean");
roleMapper.insertRole(role);
```

5. 配置*MapperScannerConfigurer*，通过扫描的形式配置*Mapper*类
```xml
<!-- 配置MapperScannerConfigurer -->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <!-- 指定扫描的基类 -->
    <property name="basePackage" value="ch12.mapper"/>
    <!-- 指定在Spring中定义Sql'SessionFactory的Bean名称-->
    <property name="sqlSessionFactoryBeanName" value="SqlSessionFactory"/>
    <!-- 使用sqlSessionTemplateBeanName将覆盖SqlSessionFactoryBeanName的配置 -->
<!--        <property name="sqlSessionTemplateBeanName" value="SqlSessionFactory"/>-->
    <!-- 指定标注，这样才能扫描成为Mapper -->
    <property name="annotationClass" value="org.springframework.stereotype.Repository"/>
</bean>
```


