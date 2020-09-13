---
title: Mybatis学习总结
date: 2020-09-04 15:45:40
tags: [Mybatis]
Categories: [Mybatis]
created: 2020-05-25
abbrlink: '202006030942'
cover: https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202006030942/cover_1.png
top_img: https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202006030942/cover_2.png
description: Mybatis的相关知识学习总结以及对整体的体系架构和工作流程的梳理，通过学习总结该部分内容深入理解该框架。
---
# Mybatis的介绍
---
## Mybatis的特性
  + 使用连接池对连接进行管理
  + SQL和代码分离,集中管理
  + 参数可以映射并且可以写动态SQL
  + 结果集映射
  + 缓存管理
  + 重复SQL可以提取重复使用
  + 提供插件机制
  
## Mybatis的核心对象和其生命周期

| 对象                     | 生命周期                    |
| ------------------------ | --------------------------- |
| SqlSeesionFactoryBuiler  | 方法局部 (method)           |
| SqlSessionFactory (单例) | 应用级别 (application)      |
| SqlSession               | 请求和操作 (request/method) |
| Mapper                   | 方法 (method)               |

## Mybatis的配置文件加载

```java
String resource = "/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```
Mybatis会将配置文件加载到Configurition这个配置对象中(在SqlSessionFactoryBuilder中会对xml配置文件进行解析,并且通过XMLConfigBuilder对象将标签配置解析到Configuration对象中).
```java
private XMLConfigBuilder(XPathParser parser, String environment, Properties props) {
    super(new Configuration());
    this.localReflectorFactory = new DefaultReflectorFactory();
    ErrorContext.instance().resource("SQL Mapper Configuration");
    this.configuration.setVariables(props);
    this.parsed = false;
    this.environment = environment;
    this.parser = parser;
}
```

## Mybatis的一些配置

> TypeHandlers

Mybatis之所以可以将java类型与数据库类型进行转换是因为mybatis提供了大量的类型处理器(typeHandler),mybatis使用TypeHandlerRegistry对象去处理数据类型的对应,代码如下 :

```java
public final class TypeHandlerRegistry {
    private final Map<JdbcType, TypeHandler<?>> JDBC_TYPE_HANDLER_MAP = new EnumMap(JdbcType.class);
    private final Map<Type, Map<JdbcType, TypeHandler<?>>> TYPE_HANDLER_MAP = new ConcurrentHashMap();
    private final TypeHandler<Object> UNKNOWN_TYPE_HANDLER = new UnknownTypeHandler(this);
    private final Map<Class<?>, TypeHandler<?>> ALL_TYPE_HANDLERS_MAP = new HashMap();
    private static final Map<JdbcType, TypeHandler<?>> NULL_TYPE_HANDLER_MAP = new HashMap();

    public TypeHandlerRegistry() {
        this.register((Class)Boolean.class, (TypeHandler)(new BooleanTypeHandler()));
        this.register((Class)Boolean.TYPE, (TypeHandler)(new BooleanTypeHandler()));
        this.register((JdbcType)JdbcType.BOOLEAN, (TypeHandler)(new BooleanTypeHandler()));
        this.register((JdbcType)JdbcType.BIT, (TypeHandler)(new BooleanTypeHandler()));
        this.register((Class)Byte.class, (TypeHandler)(new ByteTypeHandler()));
        this.register((Class)Byte.TYPE, (TypeHandler)(new ByteTypeHandler()));
        this.register((JdbcType)JdbcType.TINYINT, (TypeHandler)(new ByteTypeHandler()));
        this.register((Class)Short.class, (TypeHandler)(new ShortTypeHandler()));
        this.register((Class)Short.TYPE, (TypeHandler)(new ShortTypeHandler()));
        this.register((JdbcType)JdbcType.SMALLINT, (TypeHandler)(new ShortTypeHandler()));
        this.register((Class)Integer.class, (TypeHandler)(new IntegerTypeHandler()));
        this.register((Class)Integer.TYPE, (TypeHandler)(new IntegerTypeHandler()));
        this.register((JdbcType)JdbcType.INTEGER, (TypeHandler)(new IntegerTypeHandler()));
        this.register((Class)Long.class, (TypeHandler)(new LongTypeHandler()));
        this.register((Class)Long.TYPE, (TypeHandler)(new LongTypeHandler()));
        this.register((Class)Float.class, (TypeHandler)(new FloatTypeHandler()));
        this.register((Class)Float.TYPE, (TypeHandler)(new FloatTypeHandler()));
        this.register((JdbcType)JdbcType.FLOAT, (TypeHandler)(new FloatTypeHandler()));
        this.register((Class)Double.class, (TypeHandler)(new DoubleTypeHandler()));
        this.register((Class)Double.TYPE, (TypeHandler)(new DoubleTypeHandler()));
        this.register((JdbcType)JdbcType.DOUBLE, (TypeHandler)(new DoubleTypeHandler()));
        this.register((Class)Reader.class, (TypeHandler)(new ClobReaderTypeHandler()));
        this.register((Class)String.class, (TypeHandler)(new StringTypeHandler()));
        this.register((Class)String.class, JdbcType.CHAR, (TypeHandler)(new StringTypeHandler()));
        this.register((Class)String.class, JdbcType.CLOB, (TypeHandler)(new ClobTypeHandler()));
        this.register((Class)String.class, JdbcType.VARCHAR, (TypeHandler)(new StringTypeHandler()));
        this.register((Class)String.class, JdbcType.LONGVARCHAR, (TypeHandler)(new ClobTypeHandler()));
        this.register((Class)String.class, JdbcType.NVARCHAR, (TypeHandler)(new NStringTypeHandler()));
        this.register((Class)String.class, JdbcType.NCHAR, (TypeHandler)(new NStringTypeHandler()));
        this.register((Class)String.class, JdbcType.NCLOB, (TypeHandler)(new NClobTypeHandler()));
        this.register((JdbcType)JdbcType.CHAR, (TypeHandler)(new StringTypeHandler()));
        this.register((JdbcType)JdbcType.VARCHAR, (TypeHandler)(new StringTypeHandler()));
        this.register((JdbcType)JdbcType.CLOB, (TypeHandler)(new ClobTypeHandler()));
        this.register((JdbcType)JdbcType.LONGVARCHAR, (TypeHandler)(new ClobTypeHandler()));
        this.register((JdbcType)JdbcType.NVARCHAR, (TypeHandler)(new NStringTypeHandler()));
        this.register((JdbcType)JdbcType.NCHAR, (TypeHandler)(new NStringTypeHandler()));
        this.register((JdbcType)JdbcType.NCLOB, (TypeHandler)(new NClobTypeHandler()));
        this.register((Class)Object.class, JdbcType.ARRAY, (TypeHandler)(new ArrayTypeHandler()));
        this.register((JdbcType)JdbcType.ARRAY, (TypeHandler)(new ArrayTypeHandler()));
        this.register((Class)BigInteger.class, (TypeHandler)(new BigIntegerTypeHandler()));
        this.register((JdbcType)JdbcType.BIGINT, (TypeHandler)(new LongTypeHandler()));
        this.register((Class)BigDecimal.class, (TypeHandler)(new BigDecimalTypeHandler()));
        this.register((JdbcType)JdbcType.REAL, (TypeHandler)(new BigDecimalTypeHandler()));
        this.register((JdbcType)JdbcType.DECIMAL, (TypeHandler)(new BigDecimalTypeHandler()));
        this.register((JdbcType)JdbcType.NUMERIC, (TypeHandler)(new BigDecimalTypeHandler()));
        this.register((Class)InputStream.class, (TypeHandler)(new BlobInputStreamTypeHandler()));
        this.register((Class)Byte[].class, (TypeHandler)(new ByteObjectArrayTypeHandler()));
        this.register((Class)Byte[].class, JdbcType.BLOB, (TypeHandler)(new BlobByteObjectArrayTypeHandler()));
        this.register((Class)Byte[].class, JdbcType.LONGVARBINARY, (TypeHandler)(new BlobByteObjectArrayTypeHandler()));
        this.register((Class)byte[].class, (TypeHandler)(new ByteArrayTypeHandler()));
        this.register((Class)byte[].class, JdbcType.BLOB, (TypeHandler)(new BlobTypeHandler()));
        this.register((Class)byte[].class, JdbcType.LONGVARBINARY, (TypeHandler)(new BlobTypeHandler()));
        this.register((JdbcType)JdbcType.LONGVARBINARY, (TypeHandler)(new BlobTypeHandler()));
        this.register((JdbcType)JdbcType.BLOB, (TypeHandler)(new BlobTypeHandler()));
        this.register(Object.class, this.UNKNOWN_TYPE_HANDLER);
        this.register(Object.class, JdbcType.OTHER, this.UNKNOWN_TYPE_HANDLER);
        this.register(JdbcType.OTHER, this.UNKNOWN_TYPE_HANDLER);
        this.register((Class)Date.class, (TypeHandler)(new DateTypeHandler()));
        this.register((Class)Date.class, JdbcType.DATE, (TypeHandler)(new DateOnlyTypeHandler()));
        this.register((Class)Date.class, JdbcType.TIME, (TypeHandler)(new TimeOnlyTypeHandler()));
        this.register((JdbcType)JdbcType.TIMESTAMP, (TypeHandler)(new DateTypeHandler()));
        this.register((JdbcType)JdbcType.DATE, (TypeHandler)(new DateOnlyTypeHandler()));
        this.register((JdbcType)JdbcType.TIME, (TypeHandler)(new TimeOnlyTypeHandler()));
        this.register((Class)java.sql.Date.class, (TypeHandler)(new SqlDateTypeHandler()));
        this.register((Class)Time.class, (TypeHandler)(new SqlTimeTypeHandler()));
        this.register((Class)Timestamp.class, (TypeHandler)(new SqlTimestampTypeHandler()));
...
}
```

如果业务需要对某个字段类型进行特殊处理,我们可以自定义TypeHandler进行使用,如下 :

```java
// 自定义类型转换器进行业务处理
public class ExampleTypeHandler extends BaseTypeHandler<String> {

  @Override
  public void setNonNullParameter(PreparedStatement ps, int i, String parameter, JdbcType jdbcType) throws SQLException {
    ps.setString(i, parameter);
  }

  @Override
  public String getNullableResult(ResultSet rs, String columnName) throws SQLException {
    return rs.getString(columnName);
  }

  @Override
  public String getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
    return rs.getString(columnIndex);
  }

  @Override
  public String getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
    return cs.getString(columnIndex);
  }
}
```

```xml
<!-- 注册该自定义的Handler 配置mybatis-config.xml -->
<typeHandlers>
  <typeHandler handler="com.mybatis.test.ExampleTypeHandler"/>
</typeHandlers>
```
```xml
<!-- 使用方式一 jdbc类型转java类型 -->
<resultMap id="BaseResultMap" type="cn.az.model.ActivityApply" >
    <id column="ACTIVITY_APPLY_ID" property="activityApplyId" jdbcType="VARCHAR" />
    <result typeHandler="com.mybatis.test.ExampleTypeHandler" column="EMP_K_ACCOUNT" property="empKAccount" jdbcType="VARCHAR" />
</resultMap>
```
```xml
<!-- 使用方式二 java类型转jdbc类型-->
<select id="getBrandList" resultType="cn.az.vo.DictionariesVo">
		select emp.BRAND_CODE as code,brand.BRAND_NAME as name
		from T_MARKET_EMPLOYEE_BRAND emp
		left join T_BRAND_INFO brand
		on brand.BRAND_CODE = emp.BRAND_CODE
		and brand.TA_CODE = #{taCode, typeHandler=com.mybatis.test.ExampleTypeHandler}
		where MARKET_EMPLOYEE_ID = #{marketEmployeeId}
</select>
```

更多配置以及标签的使用可参考文档[Mybatis中文学习官网](https://mybatis.org/mybatis-3/zh/)

# Mybatis的缓存
---
## 一级缓存
一级缓存是存放在BaseExecutor执行器中的,是会话级别的缓存,无法跨会话访问,当用户访问数据库时,会先创建一个执行器,执行器访问缓存,如果已有数据则直接返回如果没有再调用数据库并且将查询结果返回用户后将数据存入缓存中。
一级缓存默认就是开启的,namespace级别(同namespace中共享)
`mybatis的增删改操作默认是会清空缓存的,查询操作默认是不会清空缓存的,也就是在mapper.xml文件中的<insert>/<update>等标签上有flushCache属性,会有默认值,为true执行该语句后会清空缓存,为false执行该语句后不会清空缓存`
[![img](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202006030942/2020060309421.png)](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202006030942/2020060309421.png)
`一级缓存如果跨会话使用的话会有脏数据,如一个会话先做一个查询然后对数据进行更新,更新后使用另一个会话去查询,此时查询的是另一个缓存中的数据而不是更新后的数据.`
如果解决脏数据问题,那么就要使用二级缓存了.

## 二级缓存
二级缓存使用的是装饰着模式,当我们开启二级缓存后mybatis会对BaseExecutor进行一个包装,该包装对象为CachingExecutor.二级缓存的管理是使用TransactionalCacheManager进行管理的.
如何开启二级缓存 :
1. 在mybatis-config.xml配置文件中开启二级缓存
```xml
<setting name="cacheEnable" value="true"/>
```
2. 需要在写SQL的mapper.xml文件中加上标签
```xml
<cache/>
<!--上下两者相等,下面的属性配置为默认配置-->
<!-- <cache type="org.apache.ibatis.cache.impl.PerpetualCache"
	   size="1024"
	   eviction="LRU"
	   flushInterval="120000"
	   readOnly="false"/> -->
```
如果开启二级缓存后,想要对某一个`<select>`标签进行二级缓存的使用关闭,可以使用属性`useCache=false`
[![img](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202006030942/2020060309422.png)](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202006030942/2020060309422.png)

# Mybatis的批量操作方式
---

> 通过JAVA代码

通过java代码的方式进行for循环然后在循环内部进行新增和编辑操作(不建议使用)该方式虽然写起来简单,但是会非常消耗性能,会多次建立连接和释放连接

> mybatis支持批量操作的语句

   SQL批量插入的语句 :
   `insert into User (id,name ....) values (1, aaa ....) , (2, bbb ....) , (3, ccc ...) ....`
   mybatis 使用动态标签`<foreach>` 拼接成该方式去批量插入,这样会减少数据库的连接与释放的次数以减少性能的消耗,但是数据库在接受sql时会有大小的限制,默认是`4m`大小,如果SQL语句拼接的过长的话会报错.
   
> mybatis支持自定义批量操作执行器

   1) 自定义一个批量操作的执行器 Batch Executor
   2) mybatis 支持三中执行器,执行器是封装在DefaultSqlSession 中的,真正执行SQL的就是该执行器,而SessionFactory所创建的Session只是提供了一些API供我们调用,执行器的三中模式 :
   a. SIMPLE 最普通的执行器,使用的是Statement处理语句
   b. REUSE 会重用预处理语句,使用PreparedStatement处理语句,意思是我们所执行过的语句会把它缓存起来,下次再执行的时候会从缓存里面去拿到该语句然后进行执行
   c. BATCH 批处理执行器,JDBC链接操作数据库时提供了PreparedStatement.addBatch()方法去添加多个SQL语句,然后成批次的放入PreparedStatement的批处理执行器(executeBatch)中执行,无论是mybatis还是spirngjdbc都封装了该批处理方式

# Mybatis的体系架构与执行流程
---
## 体系架构

1. 提供给应用直接使用 : 接口层
2. 处理数据库操作 : 核心层
3. 支持工作(代码的抽取提供复用) : 基础层

[![img](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202006030942/2020060309423.jpg)](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202006030942/2020060309423.jpg)

## 工作流程

1. 解析配置文件初始化Configuration对象
2. 使用Build创建工厂类
3. 使用工厂类创建会话
4. 会话操作数据库

> 流程图如下
[![img](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202006030942/2020060309424.png)](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202006030942/2020060309424.png)

> 具体流程细节时序图

1. 配置文件解析流程与SqlSessionFactory的创建过程
[![img](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202006030942/2020060309425.png)](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202006030942/2020060309425.png)

2. 会话工厂创建会话的过程
[![img](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202006030942/2020060309426.png)](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202006030942/2020060309426.png)

3. 会话工厂通过getMapper(xxx.class)获取代理对象的过程
[![img](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202006030942/2020060309427.png)](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202006030942/2020060309427.png)

4. 代理对象执行SQL的过程
[![img](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202006030942/2020060309428.png)](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202006030942/2020060309428.png)

以上大多是以流程图以及时序图展示的过程，如果想具体了解可以查看下方代码，该代码通过上方的过程摘取出Mybatis重要的构建过程，分为V1,V2两个版本,V1为简易版，V2为详细版
{% ghcard zhangbiao-code/mebatis, theme=solarized-light %}
