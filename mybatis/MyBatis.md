# MyBatis

## 环境配置（environments）

MyBatis可以配置成适配多种环境。
但是每个SqlSessionFactory实例只能选择一种环境。
MyBatis默认的TransactionManger是JDBC, 连接池POOLED



## MyBatis Configuration

### Property 的优先顺序

```java
1.  The highest priority properties are those passed in as a method parameter
     Properties prop = new Properties();
     prop.setProperty("datasource.username","root3");    // 1 最高
     sqlSessionFactory =
                    new SqlSessionFactoryBuilder().build(inputStream,prop);

2.  Properties loaded from the classpath resource or url attributes
    <properties resource="config/datasource.property">   // 在datasource.property里面的优先级高于mybatis-config.xml
    datasource.username root2                            // 2 中间
    
3.  Properties specified in the body of the properties element are read first. Lowest priority.
    在mybatis-config.xml中定义的：
    
    <properties resource="config/datasource.property">
        <!--property defined here will be replaced by the resource file-->
        <property name="datasource.username" value="root1"/>   // 3 最低
    </properties>
```

### Property定义默认值

Since the MyBatis 3.4.2, your can specify a default value into placeholder.
This feature is disabled by default. If you specify a default value into placeholder, you should be enable this feature by adding a special property.

```xml
<dataSource type="POOLED">
  <!-- ... -->
  <property name="username" value="${username:ut_user}"/> <!-- If 'username' property not present, username become 'ut_user' -->
</dataSource>

<!--enable default value-->
<properties resource="org/mybatis/example/config.properties">
  <!-- ... -->
  <property name="org.apache.ibatis.parsing.PropertyParser.enable-default-value" value="true"/> <!-- Enable this feature -->
</properties>
```

> **NOTE** This will conflict with the `":"` character in property keys (e.g. `db:username`) or the ternary operator of OGNL expressions (e.g. `${tableName != null ? tableName : 'global_constants'}`) on a SQL definition. If you use either and want default property values, you must change the default value separator by adding this special property:

```xml
<properties resource="org/mybatis/example/config.properties">
  <!-- ... -->
  <property name="org.apache.ibatis.parsing.PropertyParser.default-value-separator" value="?:"/> <!-- Change default value of separator -->
</properties>

<dataSource type="POOLED">
  <!-- ... -->
  <property name="username" value="${db:username?:ut_user}"/>
</dataSource>
```

### settings

| Setting                          | **Description**                                              | **Valid Values**       | **Default**            |
| -------------------------------- | ------------------------------------------------------------ | ---------------------- | ---------------------- |
| cacheEnabled                     | Globally enables or disables any caches configured in any mapper under this configuration. | true \|false           | true                   |
| lazyLoadingEnabled               | Globally enables or disables lazy loading. When enabled, all relations will be lazily loaded. This value can be superseded for a specific relation by using the `fetchType` attribute on it. | true \|false           | false                  |
| aggressiveLazyLoading            | When enabled, any method call will load all the lazy properties of the object. Otherwise, each property is loaded on demand (see also `lazyLoadTriggerMethods`). | true \|false           | false (true in ≤3.4.1) |
| autoMappingBehavior              | Specifies if and how MyBatis should automatically map columns to fields/properties. NONE disables auto-mapping. PARTIAL will only auto-map results with no nested result mappings defined inside. FULL will auto-map result mappings of any complexity (containing nested or otherwise). | NONE, PARTIAL, FULL    | PARTIAL                |
| autoMappingUnknownColumnBehavior | Specify the behavior when detects an unknown column (or unknown property type) of automatic mapping target. | NONE, WARNING, FAILING | NONE                   |

example:

```java
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="multipleResultSetsEnabled" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods"
    value="equals,clone,hashCode,toString"/>
</settings>
```

### typeAliases

A type alias is simply a shorter name for a Java type. It's only relevant to the XML configuration and simply exists to reduce redundant typing of fully qualified classnames. 

```xml
<!--With this configuration, Blog can now be used anywhere that domain.blog.Blog could be.-->
<typeAliases>
  <typeAlias alias="Author" type="domain.blog.Author"/>
  <typeAlias alias="Blog" type="domain.blog.Blog"/>
  <typeAlias alias="Comment" type="domain.blog.Comment"/>
  <typeAlias alias="Post" type="domain.blog.Post"/>
  <typeAlias alias="Section" type="domain.blog.Section"/>
  <typeAlias alias="Tag" type="domain.blog.Tag"/>
</typeAliases>


```

Each bean found in `domain.blog` , if no annotation is found, will be registered as an alias using uncapitalized non-qualified class name of the bean. That is `domain.blog.Author` will be registered as `author`. If the `@Alias` annotation is found its value will be used as an alias. 

```xml
<typeAliases>
  <package name="domain.blog"/>    <!--会自动scan包下的所有类，如果没有自定义alias,会生成一个首字母小写的alias-->
</typeAliases>
```



```java
//define 
@Alias("author")         // 自定义一个 alias 'author'
public class Author {
    ...
}
```

 built-in type aliases for common Java types.

| Alias      | Mapped Type |
| :--------- | :---------- |
| _byte      | byte        |
| _long      | long        |
| _short     | short       |
| _int       | int         |
| _integer   | int         |
| _double    | double      |
| _float     | float       |
| _boolean   | boolean     |
| string     | String      |
| byte       | Byte        |
| long       | Long        |
| short      | Short       |
| int        | Integer     |
| integer    | Integer     |
| double     | Double      |
| float      | Float       |
| boolean    | Boolean     |
| date       | Date        |
| decimal    | BigDecimal  |
| bigdecimal | BigDecimal  |
| object     | Object      |
| map        | Map         |
| hashmap    | HashMap     |
| list       | List        |
| arraylist  | ArrayList   |
| collection | Collection  |
| iterator   | Iterator    |

### TypeHandlers

You can override the type handlers or create your own to deal with unsupported or non-standard types. To do so, implement the interface `org.apache.ibatis.type.TypeHandler` or extend the convenience class `org.apache.ibatis.type.BaseTypeHandler` and optionally map it to a JDBC type.



## 生命周期和作用域

生命周期和作用域是至关重要的，错误的使用会导致非常严重的**并发问题**

**SqlSessionFactoryBuilder:**

- 一旦创建就不需要了
- 局部变量

**SqlSessionFactory:**

- 一旦创建在运行期间就一直存在。没有任何理由丢弃或重新创建。
- 最佳作用域是应用作用域
- 最简答的就是使用**单例模式**或静态单例模式

**SqlSession:**

- 连接到连接池的一个请求
- SqlSession实例是线程不安全的，不能被共享，最佳作用域是请求或方法作用域
- 用完之后需要关闭，否则资源被占用



## ResultMapper

解决属性名和字段名不一致的问题

> The `resultMap` element is the most important and powerful element in MyBatis. It's what allows you to do away with 90% of the code that JDBC requires to retrieve data from `ResultSet`s, and in some cases allows you to do things that JDBC does not even support. In fact, to write the equivalent code for something like a join mapping for a complex statement could probably span thousands of lines of code. The design of the `ResultMap`s is such that simple statements don't require explicit result mappings at all, and more complex statements require no more than is absolutely necessary to describe the relationships.

## 日志

日志工厂

| logImpl | Specifies which logging implementation MyBatis should use. If this setting is not present logging implementation will be autodiscovered. | SLF4J \| LOG4J \| LOG4J2 \| JDK_LOGGING \| COMMONS_LOGGING \| STDOUT_LOGGING \| NO_LOGGING | Not set |
| ------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------- |
|         |                                                              |                                                              |         |

STDOUT_LOGGING ：标准日志输出

```xml
<settings>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>
```

```java
Reader entry: <?xml version="1.0" encoding="UTF-8" ?>
Checking to see if class com.peter.dao.T_UserMapper matches criteria [is assignable to Object]
Opening JDBC Connection
Created connection 2121498593.
Setting autocommit to false on JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@7e737fe1]
==>  Preparing: select * from t_user where user_id = ?
==> Parameters: 13(Integer)
<==    Columns: user_id, user_name, credits, password, last_visit, last_ip
<==        Row: 13, tom, 0, 123456, 2020-05-29 11:46:56, 192.168.0.114
<==      Total: 1
userT_USER{user_id=13, user_name='tom', credits=0, password='MyPassword{encodePsw='999123456', decodePsw='123456'}', last_visit=Fri May 29 19:46:56 SGT 2020, last_ip='192.168.0.114'}
Resetting autocommit to true on JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@7e737fe1]
Closing JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@7e737fe1]
Returned connection 2121498593 to pool.

```



## 分页

### limit分页(mysql独有的)

```sql
语法: SELECT * FROM USER LIMIT <START_IDX><PAGE_SIZE>
select * from user limit 3;  #[0,3]
```

使用mybatis实现分页，核心SQL

### **RowBounds分页**

```java
RowBounds rowBoundes = new RowBounds(1,2);  # start_idx, page_size
List<User> userList = sqlSession.selectList("com.peter.dao.UserMapper.getUserByRowBonds",null,rowBounds);
```

### 分页插件

PageHelper

> https://github.com/pagehelper/Mybatis-PageHelper

## 使用注解开发

### 面向接口编程

- 解耦 Decoupling

### 使用注解开发

```java
//注解在接口上实现
package org.mybatis.example;
public interface BlogMapper {
  @Select("SELECT * FROM blog WHERE id = #{id}")
  Blog selectBlog(int id);
}

//需要在配置文件中绑定接口
<mapper class="com.peter.dao.T_UserMapper"/>
```

### 关于@Param()注解

- 基本类型或者String类型需要加上
- 引用类型不需要加
- 如果只有一个基本类型的话，可以忽略。但是建议加上。
- 我们在SQL中引用的就是我们这里@Param中设定的属性名

### #{}和${}的区别

#将传入的数据都当成一个字符串，会自动对传入数据加一组双引号

#能一定程度防止SQL注入

#会预编译用PreparedStatement parameter(?)， sql可复用高

$是直接拼接SQL,多用于传入表或字段名

> ```html
> String Substitution
> 
> By default, using the #{} syntax will cause MyBatis to generate PreparedStatement properties and set the values safely against the PreparedStatement parameters (e.g. ?). While this is safer, faster and almost always preferred, sometimes you just want to directly inject a string unmodified into the SQL Statement. For example, for ORDER BY, you might use something like this:
> 
> ORDER BY ${columnName}
> Here MyBatis won't modify or escape the string.
> 
> NOTE It's not safe to accept input from a user and supply it to a statement unmodified in this way. This leads to potential SQL Injection attacks and therefore you should either disallow user input in these fields, or always perform your own escapes and checks.
> ```

## 多对一的处理

### Nested Select for Association

| Attribute   | Description                                                  |
| :---------- | :----------------------------------------------------------- |
| `column`    | The column name from the database, or the aliased column label that holds the value that will be passed to the nested statement as an input parameter. This is the same string that would normally be passed to `resultSet.getString(columnName)`. Note: To deal with composite keys, you can specify multiple column names to pass to the nested select statement by using the syntax `column="{prop1=col1,prop2=col2}"`. This will cause `prop1` and `prop2` to be set against the parameter object for the target nested select statement. |
| `select`    | The ID of another mapped statement that will load the complex type required by this property mapping. The values retrieved from columns specified in the column attribute will be passed to the target select statement as parameters. A detailed example follows this table. Note: To deal with composite keys, you can specify multiple column names to pass to the nested select statement by using the syntax `column="{prop1=col1,prop2=col2}"`. This will cause `prop1` and `prop2` to be set against the parameter object for the target nested select statement. |
| `fetchType` | Optional. Valid values are `lazy` and `eager`. If present, it supersedes the global configuration parameter `lazyLoadingEnabled` for this mapping. |

```sql
<resultMap id="blogResult" type="Blog">
  <association property="author" column="author_id" javaType="Author" select="selectAuthor"/>
</resultMap>

<select id="selectBlog" resultMap="blogResult">
  SELECT * FROM BLOG WHERE ID = #{id}
</select>

<select id="selectAuthor" resultType="Author">
  SELECT * FROM AUTHOR WHERE ID = #{id}
</select>
```

**Note: While this approach is simple, it will not perform well for large data sets or lists. This problem is known as the "N+1 Selects Problem".**

### Nested Results for Association

| Attribute       | Description                                                  |
| :-------------- | :----------------------------------------------------------- |
| `resultMap`     | This is the ID of a ResultMap that can map the nested results of this association into an appropriate object graph. This is an alternative to using a call to another select statement. It allows you to join multiple tables together into a single ResultSet. Such a ResultSet will contain duplicated, repeating groups of data that needs to be decomposed and mapped properly to a nested object graph. To facilitate this, MyBatis lets you "chain" result maps together, to deal with the nested results. An example will be far easier to follow, and one follows this table. |
| `columnPrefix`  | When joining multiple tables, you would have to use column alias to avoid duplicated column names in the ResultSet. Specifying columnPrefix allows you to map such columns to an external resultMap. Please see the example explained later in this section. |
| `notNullColumn` | By default a child object is created only if at least one of the columns mapped to the child's properties is non null. With this attribute you can change this behaviour by specifiying which columns must have a value so MyBatis will create a child object only if any of those columns is not null. Multiple column names can be specified using a comma as a separator. Default value: unset. |
| `autoMapping`   | If present, MyBatis will enable or disable automapping when mapping the result to this property. This attribute overrides the global autoMappingBehavior. Note that it has no effect on an external resultMap, so it is pointless to use it with `select` or `resultMap` attribute. Default value: unset. |

```sql
<select id="selectBlog" resultMap="blogResult">
  select
    B.id            as blog_id,
    B.title         as blog_title,
    B.author_id     as blog_author_id,
    A.id            as author_id,
    A.username      as author_username,
    A.password      as author_password,
    A.email         as author_email,
    A.bio           as author_bio
  from Blog B left outer join Author A on B.author_id = A.id
  where B.id = #{id}
</select>

<resultMap id="blogResult" type="Blog">
  <id property="id" column="blog_id" />
  <result property="title" column="blog_title"/>
  <association property="author" resultMap="authorResult" />
</resultMap>

<resultMap id="authorResult" type="Author">
  <id property="id" column="author_id"/>
  <result property="username" column="author_username"/>
  <result property="password" column="author_password"/>
  <result property="email" column="author_email"/>
  <result property="bio" column="author_bio"/>
</resultMap>
```

## 一对多处理

### Nested Select for Collection

```xml
<!--javaType="ArrayList" ofType="Post"-->
<resultMap id="blogResult" type="Blog">
  <collection property="posts" javaType="ArrayList" column="id" ofType="Post" select="selectPostsForBlog"/>
</resultMap>

<select id="selectBlog" resultMap="blogResult">
  SELECT * FROM BLOG WHERE ID = #{id}
</select>

<select id="selectPostsForBlog" resultType="Post">
  SELECT * FROM POST WHERE BLOG_ID = #{id}
</select>
```

### Nested Results for Collection

```sql
<select id="selectBlog" resultMap="blogResult">
  select
  B.id as blog_id,
  B.title as blog_title,
  B.author_id as blog_author_id,
  P.id as post_id,
  P.subject as post_subject,
  P.body as post_body,
  from Blog B
  left outer join Post P on B.id = P.blog_id
  where B.id = #{id}
</select>
```

```xml
<!--ofType="Post"-->
<resultMap id="blogResult" type="Blog">
  <id property="id" column="blog_id" />
  <result property="title" column="blog_title"/>
  <collection property="posts" ofType="Post">
    <id property="id" column="post_id"/>
    <result property="subject" column="post_subject"/>
    <result property="body" column="post_body"/>
  </collection>
</resultMap>
```

## 动态SQL

One of the most powerful features of MyBatis has always been its Dynamic SQL capabilities. 

### if

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <if test="title != null">
    AND title like #{title}
  </if>
  <if test="author != null and author.name != null">
    AND author_name like #{author.name}
  </if>
</select>
```

### choose, when, otherwise

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
```

### trim, where, set

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG
  <where>
    <if test="state != null">
         state = #{state}
    </if>
    <if test="title != null">
        AND title like #{title}
    </if>
    <if test="author != null and author.name != null">
        AND author_name like #{author.name}
    </if>
  </where>
</select>
```

The *where* element knows to only insert "WHERE" if there is any content returned by the containing tags. Furthermore, if that content begins with "AND" or "OR", it knows to strip it off.

If the *where* element does not behave exactly as you like, you can customize it by defining your own trim element. For example, the trim equivalent to the *where* element is:

```xml
<trim prefix="WHERE" prefixOverrides="AND |OR ">
  ...
</trim>
```

There is a similar solution for dynamic update statements called *set*. The *set* element can be used to dynamically include columns to update, and leave out others. For example:

```xml
<update id="updateAuthorIfNecessary">
  update Author
    <set>
      <if test="username != null">username=#{username},</if>
      <if test="password != null">password=#{password},</if>
      <if test="email != null">email=#{email},</if>
      <if test="bio != null">bio=#{bio}</if>
    </set>
  where id=#{id}
</update>
```

The *set* element will dynamically prepend the SET keyword, and also eliminate any extraneous commas that might trail the value assignments after the conditions are applied.

```xml
<trim prefix="SET" suffixOverrides=",">
  ...
</trim>
```

### foreach

Another common necessity for dynamic SQL is the need to iterate over a collection, often to build an IN condition. For example:

```xml
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT *
  FROM POST P
  WHERE ID in
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
</select>
```

**NOTE** You can pass any Iterable object (for example List, Set, etc.), as well as any Map or Array object to foreach as collection parameter. When using an Iterable or Array, index will be the number of current iteration and value item will be the element retrieved in this iteration. When using a Map (or Collection of Map.Entry objects), index will be the key object and item will be the value object.

### bind

The `bind` element lets you create a variable out of an OGNL expression and bind it to the context. For example:

```xml
<select id="selectBlogsLike" resultType="Blog">
  <bind name="pattern" value="'%' + _parameter.getTitle() + '%'" />
  SELECT * FROM BLOG
  WHERE title LIKE #{pattern}
</select>
```

## MyBatis Cache

By default, just **local session(first cache)** caching is enabled that is used solely to cache data for the duration of a session. To enable a **global second level(second cache)** of caching you simply need to add one line to your SQL Mapping file:

```xml
<cache/>
```

**注意:**要开启二级缓存，需要设定cachEnabled. 一旦开启，会先用global cache,然后 local session cache.

| Setting      | Description                                                  | Valid Values  | Default |
| :----------- | :----------------------------------------------------------- | :------------ | :------ |
| cacheEnabled | Globally enables or disables any caches configured in any mapper under this configuration. | true \| false | true    |



Literally that's it. The effect of this one simple statement is as follows:

- All results from select statements in the mapped statement file will be cached.
- All insert, update and delete statements in the mapped statement file will **flush the cache**.   **# 手动调用commit也会刷新缓存**
- The cache will use a **Least Recently Used (LRU)** algorithm for eviction.
- The cache will not flush on any sort of time based schedule (i.e. **no Flush Interval**).
- The cache will store **1024 references** to lists or objects (whatever the query method returns).
- The cache will be treated as a **read/write cache**, meaning objects retrieved are not shared and can be safely modified by the caller, without interfering with other potential modifications by other callers or threads.                   # Read/write cache的pojo对象，需要实现序列化 serializable. Read only则不需要

**NOTE** The cache will only apply to statements declared in the mapping file where the cache tag is located. If you are using the Java API in conjunction with the XML mapping files, then statements declared in the companion interface will not be cached by default. You will need to refer to the cache region using the @CacheNamespaceRef annotation.

The available eviction policies available are:

- `LRU` – Least Recently Used: Removes objects that haven't been used for the longst period of time.
- `FIFO` – First In First Out: Removes objects in the order that they entered the cache.
- `SOFT` – Soft Reference: Removes objects based on the garbage collector state and the rules of Soft References.
- `WEAK` – Weak Reference: More aggressively removes objects based on the garbage collector state and rules of Weak References.