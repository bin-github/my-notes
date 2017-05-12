### mybatis结果集映射总结

---

#### 1.普通对象结果集映射
>1.对象的属性都是一些java的八种常用类型或者String类型，
直接进行映射即可，如下：

```
<resultMap type="com.test.entity.Person" id="person">
		<id property="id" column="id" />
		<result property="name" column="user_name"/>
		<result property="sex" column="user_sex"/>
                         	┇
</resultMap>
```
>2.

#### 2.对象属性含有另一种对象的结果集映射
>1.对象属性中，部分属性是另一种对象，并且是has - one关系（一对一关系），如：一个博客有一个作者。
>这类情况使用 **association** 映射节点。
>>1.association 和普通的resultMap配置属性基本相同

>>2.具体如下：

```
<select id="selectBlog" parameterType="int" resultMap="blogResult">
    select  
    B.id as blog_id,  
    B.title as blog_title,  
    **B.author_id as blog_author_id,**  
    **A.id as author_id,**  
    A.username as author_username,  
    A.password as author_password,  
    A.email as author_email,  
    A.bio as author_bio  
    from Blog B left outer join Author A on B.author_id = A.id  
    where B.id = #{id}  
</select>
*如上的这类一对一嵌套查询可以使用如下的这类映射：*
<resultMap id="blogResult" type="Blog">  
    <id property=”blog_id” column="id" />  
    <result property="title" column="blog_title"/>  
    <association property="author" column="blog_author_id"     javaType="Author"  
    resultMap=”authorResult”/>  
</resultMap>  
       
<resultMap id="authorResult" type="Author">  
    <id property="id" column="author_id"/>  
    <result property="username" column="author_username"/>  
    <result property="password" column="author_password"/>  
</resultMap>
*或者直接将这两个查映射直接嵌套写：*
<resultMap id="blogResult" type="Blog">  
    <id property=”blog_id” column="id" />  
    <result property="title" column="blog_title"/>  
    <association property="author" column="blog_author_id"     javaType="Author">  
        <id property="id" column="author_id"/>  
        <result property="username" column="author_username"/>  
        <result property="password" column="author_password"/>  
    </association>  
</resultMap>
    ** id元素在嵌套结果映射中扮演了非常重要的角色，你应该总是指定一个或多个属性来唯一标识这个结果集。
事实上，如果您没有那样做，MyBatis也会工作，但是会导致严重性能开销。选择尽量少的属性来唯一标识结果，
而使用主键是最明显的选择（即使是复合主键）。 **
     在<resultMap>节点内，如果子节点为主键，则应该使用<id>，而不是<result>
```

>2.对象属性中，部分属性是另一种对象，并且是has - many关系（一对多关系），如：一个博客有多篇文章。
>这类情况使用 **collection** 映射节点，具体如下：
>>1.collection的属性 ofType ：是集合中元素的类型，一般是普通java对象，其余与association 基本相同

>>2.具体实现：

```
<select id="selectBlog" parameterType="int" resultMap="blogResult">
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
*将两个结果集嵌套，当然也可以分开，提高重用性*
<resultMap id="blogResult" type="Blog">  
    <id property=”id” column="blog_id" />  
    <result property="title" column="blog_title"/>  
    <collection property="posts" ofType="Post">  
        <id property="id" column="post_id"/>  
        <result property="subject" column="post_subject"/>  
        <result property="body" column="post_body"/>  
    </collection>  
</resultMap>
```
>> 3、 **在具体的实现mapper中，几个属性是有顺序的：id，result，association，collection的顺序一定要注意，如上顺序不能乱，不然会报结果集不匹配的错误。**

>> 4、resultMap中的id属性：id属性用于标识类别的，如下，奖励有多个名次奖励，每个名次有多种奖品
```
<resultMap id="cptDetail" type="com.…….CptDetail">
        <result column="cpt_begin_time" property="cptBeginTime" typeHandler="org.apache.ibatis.type.LocalTimeTypeHandler"/>
        <collection property="rewardInfos" ofType="…….CptRewardInfo" resultMap="rewardInfo"/>
    </resultMap>
    
     <resultMap id="rewardInfo" type="…….CptRewardInfo">
        <!-- 使用id属性的值去分类-->
        <id column="reward_rank" property="rewardRank"/>
        <collection property="cptRewards" ofType="…….CptReward" resultMap="cptReward"/>
    </resultMap>
```

#### 3.Discriminator(识别器)结果集筛选：
>1.功能类似于java的switch，

>2.case中的resultMap需要extends父resultMap，不然结果集值包含case中resultMap中的属性。

>3.具体实现：

```
<resultMap id="vehicleResult" type="Vehicle">  
    <id property=”id” column="id" />  
    <result property="vin" column="vin"/>  
    <result property="color" column="color"/>  
    <discriminator javaType="int" column="vehicle_type">  
        <case value="1" resultMap="carResult"/>  
        <case value="2" resultMap="truckResult"/>  
    </discriminator>  
</resultMap> 

<resultMap id="carResult" type="Car" **extends=”vehicleResult”**>
    <result property=”doorCount” column="door_count" />
</resultMap>
```
### 4、Java8中localTime和localDate在mybatis中的映射：
> 1、Java8中的localTime和localDate在DB中的映射：

type | header 1 | header 2
--- | ---|---
Java8 | DateTime | localDate
db | time | localTime

> 2、程序中使用localTime和localDate时，mybatis结果集映射需要额外的jar包转换，

```
<dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-typehandlers-jsr310</artifactId>
</dependency>
```
> 该jar下其实是一组typeHandler（当然，也可以自己实现这些typeHandler），主要针对Java8中新时间api的转换，如果使用的mybatis版本是 **3.4** 之后的版本时，只需要在maven依赖中添加该jar即可，如果是之前的版本，还需要在实际的熟悉字段后面添加对应的typeHandler，如下：

```
<resultMap id="cptSimInfo" type="…… .CptSimInfo">
        
        <result column="cpt_show_end_day" property="cptShowEndDay" typeHandler="org.apache.ibatis.type.LocalDateTypeHandler"/>
        <result column="cpt_show_begin_time" property="cptShowBeginTime" typeHandler="org.apache.ibatis.type.LocalTimeTypeHandler"/>
        
    </resultMap>
```

参考：http://blog.csdn.net/jr_soft/article/details/35274691
http://www.cnblogs.com/yansum/p/5819973.html
http://www.mybatis.org/mybatis-3/sqlmap-xml.html
