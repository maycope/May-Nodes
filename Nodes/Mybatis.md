# MyBatis

## 基础部分
### 什么是MyBatis
MyBatis 是一个半ORM（对象关系映射）框架，内部封装了JDBC，在开发的过程中只需要关注SQL本身，而不再花费精力去处理加载驱动，创建连接，创建`statement`等复杂的过程，直接编写原生的SQL语句，有更高的灵活度。
#### ORM是什么
ORM（Object Relational Mapping），对象关系映射，是一种为了解决关系型数据库数据与简单Java对象（POJO）的映射关系的技术。简单的说，ORM是通过使用描述对象和数据库之间映射的元数据，将程序中的对象自动持久化到关系型数据库中。
### 有什么优点
（1）基于SQL语句编程，相当灵活，不会对应用程序或者数据库的现有设计造成任何影响，SQL写在XML里，解除sql与程序代码的耦合，便于统一管理；提供XML标签，支持编写动态SQL语句，并可重用。
（2）与JDBC相比，减少了50%以上的代码量，消除了JDBC大量冗余的代码，不需要手动开关连接；
（3）很好的与各种数据库兼容（因为MyBatis使用JDBC来连接数据库，所以只要JDBC支持的数据库MyBatis都支持）。
（4）能够与Spring很好的集成；
（5）提供映射标签，支持对象与数据库的ORM字段关系映射；提供对象关系映射标签，支持对象关系组件维护。
### 缺点
SQL语句的编写工作量较大，尤其当字段多、关联表多时，对开发人员编写SQL语句的功底有一定要求
SQL语句依赖于数据库，导致数据库移植性差，不能随意更换数据库
### #{}与 ${} 的不同之处
#{}是**预编译**处理，${}是**字符串替换**。
Mybatis在处理#{}时，会将sql中的#{}替换为?号，调用PreparedStatement的set方法来赋值；
Mybatis在处理${}时，就是把${}替换成变量的值。
使用#{}可以有效的防止SQL注入，提高系统安全性。

## XML 文件

### XML映射文件是如何和Dao接口进行对应

对于Dao接口就是我们常说的Mapper接口，对于Dao接口的全限定名，就是映射文件中`<mapper>`标签的`namespace`值。对于接口中的各个方法的名称就是M映射文件中Mapper的 **StateMent**的`id`值。接口中方法的参数就是传递给sql语句的参数值。

对于Mapper接口来说其并没有完整意义上的实现类，当在调用接口方法的时候，接口全限名+方法名拼接字符串作为key值，可唯一定位一个MapperStatement。在Mybatis中，每一个<select>、<insert>、<update>、<delete>标签，都会被解析为一个MapperStatement对象。

举个栗子：有如下userMapper接口

![image-20200818110118977](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200818110118977.png)

这个时候我们在XML中就可以进行这样填写:

![image-20200818110315420](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200818110315420.png)

注意： 在Mapper接口里面的方法是不能够进行重载的，是不能重载的，因为是使用 全限名+方法名 的保存和寻找策略。`Mapper 接口的工作原理是JDK动态代理，Mybatis运行时会使用JDK动态代理为Mapper接口生成代理对象proxy，代理对象会拦截接口方法，转而执行MapperStatement所代表的sql，然后将sql执行结果返回`。

### MyBatis是如何进行分页的

### 对于Sql语句的结果是如何封装为目标对象进行返回

第一种是使用<resultMap>标签，逐一定义数据库列名和对象属性名之间的映射关系。

第二种是使用sql列的别名功能，将列的别名书写为对象属性名。

有了列名与属性名的映射关系后，Mybatis通过反射创建对象，同时使用反射给对象的属性逐一赋值并返回，那些找不到映射关系的属性，是无法完成赋值的。

### Mybaits 的Xml映射文件，不同的XMl文件 id是否可以重复
 不同的Xml映射文件，如果配置了namespace，那么id可以重复；如果没有配置namespace，那么id不能重复；

原因就是namespace+id是作为Map<String, MapperStatement>的key使用的，如果没有namespace，就剩下id，那么，id重复会导致数据互相覆盖。有了namespace，自然id就可以重复，namespace不同，namespace+id自然也就不同。

但是，在以前的Mybatis版本的namespace是可选的，不过新版本的namespace已经是必须的了

### 如何传递多个参数

1. **方法1：顺序传参法**

   ```java
   public User selectUser(String name, int deptId);
   
   <select id="selectUser" resultMap="UserResultMap">
       select * from user
       where user_name = #{0} and dept_id = #{1}
   </select>
   123456
   ```

   \#{}里面的数字代表传入参数的顺序。

   这种方法不建议使用，sql层表达不直观，且一旦顺序调整容易出错。

   **方法2：@Param注解传参法**

   ```java
   public User selectUser(@Param("userName") String name, int @Param("deptId") deptId);
   
   <select id="selectUser" resultMap="UserResultMap">
       select * from user
       where user_name = #{userName} and dept_id = #{deptId}
   </select>
   123456
   ```

   \#{}里面的名称对应的是注解@Param括号里面修饰的名称。

   这种方法在参数不多的情况还是比较直观的，推荐使用。

   **方法3：Map传参法**

   ```java
   public User selectUser(Map<String, Object> params);
   
   <select id="selectUser" parameterType="java.util.Map" resultMap="UserResultMap">
       select * from user
       where user_name = #{userName} and dept_id = #{deptId}
   </select>
   123456
   ```

   \#{}里面的名称对应的是Map里面的key名称。

   这种方法适合传递多个参数，且参数易变能灵活传递的情况。

   **方法4：Java Bean传参法**

   ```java
   public User selectUser(User user);
   
   <select id="selectUser" parameterType="com.jourwon.pojo.User" resultMap="UserResultMap">
       select * from user
       where user_name = #{userName} and dept_id = #{deptId}
   </select>
   123456
   ```

   \#{}里面的名称对应的是User类里面的成员属性。

   这种方法直观，需要建一个实体类，扩展不容易，需要加属性，但代码可读性强，业务逻辑处理方便，推荐使用。

### mapper接口调用时候有什么要求

1、Mapper接口方法名和mapper.xml中定义的每个sql的id相同。

2、Mapper接口方法的输入参数类型和mapper.xml中定义的每个sql 的parameterType的类型相同。

3、Mapper接口方法的输出参数类型和mapper.xml中定义的每个sql的resultType的类型相同。

4、Mapper.xml文件中的namespace即是mapper接口的类路径。

## 分页插件原理

###  如何进行分页以及分页插件的原理

Mybatis使用RowBounds对象进行分页，它是针对ResultSet结果集执行的内存分页，而非物理分页，可以在sql内直接书写带有物理分页的参数来完成物理分页功能，也可以使用分页插件来完成物理分页。

分页插件的基本原理是使用Mybatis提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的sql，然后重写sql，根据dialect方言，添加对应的物理分页语句和物理分页参数。

举例：select * from student，拦截sql后重写为：select t.* from (select * from student) t limit 0, 10

### 分页插件原理

Mybatis仅可以编写针对ParameterHandler、ResultSetHandler、StatementHandler、Executor这4种接口的插件，Mybatis使用JDK的动态代理，为需要拦截的接口生成代理对象以实现接口方法拦截功能，每当执行这4种接口对象的方法时，就会进入拦截方法，具体就是InvocationHandler的invoke()方法，当然，只会拦截那些你指定需要拦截的方法。

实现Mybatis的Interceptor接口并复写intercept()方法，然后在给插件编写注解，指定要拦截哪一个接口的哪些方法即可。



## Mybatis 解析与运行原理（源码解析）

### 执行步骤

1、 创建SqlSessionFactory

2、 通过SqlSessionFactory创建SqlSession

3、 通过sqlsession执行数据库操作

4、 调用session.commit()提交事务

5、 调用session.close()关闭会话

```java
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory =
                new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        try{
            User user = sqlSession.selectOne("cn.maycope.mapper.userMapper.getUser","abc");
        }finally {
            sqlSession.close();
        }
```

### 工作原理

![image-20200818160302237](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200818160302237.png)

1）读取 MyBatis 配置文件：mybatis-config.xml 为 MyBatis 的全局配置文件，配置了 MyBatis 的运行环境等信息，例如数据库连接信息。

2）加载映射文件。映射文件即 SQL 映射文件，该文件中配置了操作数据库的 SQL 语句，需要在 MyBatis 配置文件 mybatis-config.xml 中加载。mybatis-config.xml 文件可以加载多个映射文件，每个文件对应数据库中的一张表。

3）构造会话工厂：通过 MyBatis 的环境等配置信息构建会话工厂 SqlSessionFactory。

4）创建会话对象：由会话工厂创建 SqlSession 对象，该对象中包含了执行 SQL 语句的所有方法。

5）Executor 执行器：MyBatis 底层定义了一个 Executor 接口来操作数据库，它将根据 SqlSession 传递的参数动态地生成需要执行的 SQL 语句，同时负责查询缓存的维护。

6）MappedStatement 对象：在 Executor 接口的执行方法中有一个 MappedStatement 类型的参数，该参数是对映射信息的封装，用于存储要映射的 SQL 语句的 id、参数等信息。

7）输入参数映射：输入参数类型可以是 Map、List 等集合类型，也可以是基本数据类型和 POJO 类型。输入参数映射过程类似于 JDBC 对 preparedStatement 对象设置参数的过程。

8）输出结果映射：输出结果类型可以是 Map、 List 等集合类型，也可以是基本数据类型和 POJO 类型。输出结果映射过程类似于 JDBC 对结果集的解析过程。

### 源码分析

#### 前景提要

上面讲到通过下面的

```java
 SqlSessionFactory sqlSessionFactory =
                new SqlSessionFactoryBuilder().build(inputStream);
 SqlSession sqlSession = sqlSessionFactory.openSession();
```

代码创建`sqlsession`,然后就可以通过**sqlsession**执行数据库具体操作。但是现在有两个问题：

1. 想要执行具体的SQL语句我们需要先和数据库底层建立连接，这样才能在获取到具体的SQL语句的时候能够顺利执行。（**mybatis-config.xml配置文件如下**）读取到我们具体的`driver`，`url`,`username`等进行数据库连接的建立，关于这个过程是如何具体实现的需要我们具体去分析源码弄明白。

   **mybatis-config.xml**

   ![](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200818160614075.png)

2. 在建立完成之后，还需要读取对应的SQL语句,就是我们把`userMapper.xml`中的内容来读取出来，我们具体要进行的是什么SQL语句，是`select`，`update`，亦或是`delete`，又是如何从我们的userMapper.xml中读取到具体的sql语句，也需要进行具体的源码分析。**userMapper.xml**如下。

**userMapper.xml**

![image-20200818161359672](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200818161359672.png)

在完成以上的建立连接和读取到具体的SQL语句之后，才能够开始具体的执行过程。

#### 拿到数据库配置源信息

因为通过上面我们知道具体的连接和读取sql语句操作都是在`SqlSessionFactory`中进行，所以我们打上断点进行单步的调试，

![image-20200818162355032](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200818162355032.png)

我看可以看到有一个`XMLConfigBuilder`其实就是用来解析我们的`xml`文件,这个时候将解析的结果传递给了parser 我们进入到 `parse()`来一探究竟。

![image-20200818162558027](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200818162558027.png)

如下图所示：调用到`parseConfiguration`，传递过了一个参数 `root`,我们这里可以简单看一看，若是前面能够成功的读取到我们的配置文件，这里的`root`值也就应该是我们配置文件的内容，这里可以大体看到其内容就是我们配置文件内容（见标注地方）

注意：对于`parseConfiguration`就是读取我们所有配置信息的地方。

![image-20200818164152676](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200818164152676.png)

下面我将其具体值复制出来，见下。可以发现和我们配置文件中的内容如出一辙，现在的我们获取到配置文件中全部的内容，现在思考的就是如何将其中的配置具体读出来呢？

```text
<configuration>
    <properties resource="config.properties"/>    
    <environments default="development">
        <environment id="development">
        // 看这个environment 标签中的内容就是存放我们主要配置信息的包裹标签。
            <transactionManager type="JDBC"/>            
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>                
                <property name="url" value="${url}"/>                
                <property name="username" value="${username}"/>                
                <property name="password" value="${password}"/>                
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="mybatis/userMapper.xml"/>        
    </mappers>
</configuration>

```

可以看到对于`environment`包裹标签来说，在源码中(见下图)也有获取其值的具体方法，我们进去一探究竟。看看会不会能够获取到我们的 连接的具体信息。

![image-20200818164626937](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200818164626937.png)

打上断点，进入其中：将具体的`context`值复制出来如下：

![image-20200818165019864](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200818165019864.png)

可以发现此时的context已经成功将我们的具体信息都拿到，可以看到此时对于我们的`driver`和`url`都读取到了具体的值。

```text
<environments default="development">
    <environment id="development">
        <transactionManager type="JDBC"/>        
        <dataSource type="POOLED">
            <property name="driver" value="com.mysql.jdbc.Driver"/>            
            <property name="url" value="jdbc:mysql://127.0.0.1:3306/spring_mybatis"/>            
            <property name="username" value="root"/>            
            <property name="password" value="123456"/>            
        </dataSource>
    </environment>
</environments>
```

**思考**：对于具体信息的内容是什么是否被加载到具体的对应值中：在进行配置文件读取的时候，会进行`this.propertiesElement(root.evalNode("properties"));`,读取到的也就是我们在`mybatis-config.xml `文件中进行的具体配置

![image-20200818165802599](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200818165802599.png)

拿到具体的配置文件和其具体的所有值之后就可以开始进行具体的解析操作，将所有的值都解析出来：

如下图所示，打断点进入到 `dataSourceElement`中

![image-20200818170257489](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200818170257489.png)

展示结果：可以发现和我们`mybatis-config.xml `中的type的类型相同。表示对于我们的`dataSourceElement`就是

![image-20200818170527851](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200818170527851.png)

当我们具体都执行完成`DataSourceFactory dsFactory = this.dataSourceElement(child.evalNode("dataSource"));`之后就可以发现我们的`dataSource`就正确获取到我们的所有的配置信息：

![image-20200818171335257](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200818171335257.png)

完成之后就会使用到`setEnvironment`进行赋值处理。

![image-20200819092151969](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200819092151969.png)

至此我们就把配置文件信息都赋值给了一个具体的类：

![image-20200819092340984](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200819092340984.png)

现在我们的数据源的读取就完整结束。用下面时序图来完整进行描述。


![image-20200818143422771](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200818143422771.png)



#### 获取SQL信息

在上面的操作中，具体讲解了是如何通过对配置文件的读取获取到我们的配置源信息，主要就是如下，进行我们各种配置文件的读取，对于SQL语句读取部分，就主要在`this.mapperElement(root.evalNode("mappers"));`中进行具体的分析。

![image-20200818172329393](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200818172329393.png)

打上对应的断点进入其中：可以发现会进行对mappers的不同类型的加载。

![image-20200818172725779](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200818172725779.png)

注意对于`mybatis 加载mappers 有四种方式`。且从上面的代码中可以得出来`package`的优先级最高。

![image-20200818172911501](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200818172911501.png)

这里我们使用到的是`resource`在xml中使用到。

进入到相对应的情况下，点击进入到mapperParser()中。

![image-20200819084308611](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200819084308611.png)

可以看到此时就解析到我们对应的XML文件信息了。

![image-20200819085055650](upload\image-20200819085055650.png)

然后进入到`configurationElement`中，可以看到此时就对我们具体的sql信息进行了解读：

我们将参数context中的值打印出来：

```text
<mapper namespace="cn.maycope.userMapper">
    <resultMap type="cn.maycope.User" id="BaseResultMap">
        <id jdbcType="INTEGER" column="id" property="id"/>        
        <result jdbcType="VARCHAR" column="username" property="username"/>        
        <result jdbcType="VARCHAR" column="password" property="password"/>        
    </resultMap>
    <select resultMap="BaseResultMap" id="getUser">
        select * from users where username= #{username}
    </select>
</mapper>

```

可以看到此时就获取到我们所有的sql语句信息。

下面开始查询使用到的是哪个具体的关键字。

![image-20200819085335499](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200819085335499.png)

再进入到上图中对应的方法中，进入到`parseStatementNode`,我们的核心方法中

![image-20200819085733929](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200819085733929.png)

发现其就是将我们的一些信息和对应的SQL语句都暂时赋值给了一些临时的变量

![image-20200819085947919](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200819085947919.png)

赋值完成之后进入到`addMappedStatement`中

![image-20200819090125997](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200819090125997.png)

发现就是我们的一些基础的赋值的语句，最后将所有的信息都传递给了`MappedStatement`

![image-20200819090552975](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200819090552975.png)

此时的`MappedStatement`的`addMappedStatement()`方法就是最后将我们具体的sql语句变换成为具体类的部分。

![image-20200819090658264](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200819090658264.png)

所以最后的信息会传递给我们的`MappedStatement`。就是最终的设置SQL语句。



#### 完成解析

上面我们完成了对配置文件的读取，获取到所有的配置文件的信息，最终存在`configuration`中，然后开始对sql语句进行解析，最终都存在`MappedStatement`上，这些也都是

```java
SqlSessionFactory sqlSessionFactory =
        new SqlSessionFactoryBuilder().build(inputStream);
```

语句的执行结果，获取到上面的信息，下面开始具体的调用

```java
User user = sqlSession.selectOne("cn.maycope.userMapper.getUser","abc");
```

开始对sql语句进行具体解析，开始具体的数据库操作。

对`SelectOne()`方法打断点进入其中：

![image-20200819093017104](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200819093017104.png)

进入到query方法中。

![image-20200819084426708](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200819084426708.png)

可以查看到此时已经获取到了我们对应的SQL语句,此时我们的`#{}`就被取代成为了`？`就是我们的占位符

![image-20200819084655493](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200819084655493.png)

以上就是我们先获取到连接的信息，然后获取配置文件中的SQL语句，最后再将SQL语句拿出来具体执行的过程，但是下面是如何继续执行，又是如何将具体的结果存储起来呢，详情信息后续会持续更新。

**未完待续**







































































![image-20200818151444175](https://gitee.com/maycopes/MyImages/raw/master//images/image-20200818151444175.png)