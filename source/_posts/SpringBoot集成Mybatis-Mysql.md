title: SpringBoot集成Mybatis+Mysql
author: Wtli
tags:
  - SpringBoot
  - Mybatis
  - Mysql
  - ''
categories:
  - 后端
date: 2020-08-26 13:58:00
---
使用SpringBoot+Mysql+Mybatis+Mybatis Generator。

<!-- more-->

### 添加pom依赖
```
<!-- MyBatis 生成器 -->
<dependency>
     <groupId>org.mybatis.generator</groupId>
     <artifactId>mybatis-generator-core</artifactId>
     <version>1.3.7</version>
 </dependency>
 <!--Mysql数据库驱动-->
 <dependency>
     <groupId>mysql</groupId>
     <artifactId>mysql-connector-java</artifactId>
     <version>8.0.15</version>
</dependency>
<!--集成druid连接池-->
<dependency>
     <groupId>com.alibaba</groupId>
     <artifactId>druid-spring-boot-starter</artifactId>
     <version>1.1.10</version>
</dependency>
```
添加三个依赖，分别是：
> MyBatis 生成器：能够根据数据库自动生成实体类，方便开发   
> Mysql数据库驱动：Mysql在Java中的驱动器  
> 集成druid连接池：连接池的功能是管理数据库连接，Druid能够提供强大的监控和扩展功能。

### 添加application.yml配置

1. 添加datasource配置
```
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/****?serverTimezone=Asia/Shanghai
    username: root
    password: ****
    driver-class-name: com.mysql.cj.jdbc.Driver
```
温馨提示： 
1. 如果在数据库中用到Date格式的字段记得添加上，？serverTimezone=Asia/Shanghai，否则时间会不准确～～
2. driver-class-name之前是com.mysql.jdbc.Driver，已经弃用了，现在改成使用com.mysql.cj.jdbc.Driver。

2. ~~添加mybatis配置，配置扫描mapper包的位置，这个配置亲试不是必须的。~~
```
mybatis:
  mapper-locations:
    - classpath:mapper/*.xml
```
3. 在自己包中新建mbg文件夹，文件夹中新建mapper和model文件夹，用来存放生成的代码。（这个可以在后面的Generator配置文件中设置）


### 集成Mybatis Generator
Mybatis Generator是代码自动生成工具，作为一个后端不写SQL，感觉还是怪怪的。
添加配置：
1. MybatisConfig  
```
@Configuration
@MapperScan({"com.aircas.monitor.mbg.mapper", "com.aircas.monitor.dao"})
public class MyBatisConfig {
}
```

2. Generator配置类文件  
```
public class Generator {
    public static void main(String[] args) throws Exception {
        //MBG 执行过程中的警告信息
        List<String> warnings = new ArrayList<String>();
        //当生成的代码重复时，覆盖原代码
        boolean overwrite = true;
        //读取我们的 MBG 配置文件
        InputStream is = Generator.class.getResourceAsStream("/mybatisGenerate/generatorConfig.xml");
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(is);
        is.close();
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        //创建 MBG
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
        //执行生成代码
        myBatisGenerator.generate(null);
        //输出警告信息
        for (String warning : warnings) {
            System.out.println("warning:  " + warning);
        }
    }

}
```

3. generatorConfig.xml
Mybatis Generator最主要的配置文件,里面包括的生成model、mapper等代码的配置 ，还包括生成路径的配置。～～ 
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <properties resource="./mybatisGenerate/generator.properties"/>
    <context id="MySqlContext" targetRuntime="MyBatis3" defaultModelType="flat">
        <!-- beginningDelimiter和endingDelimiter：指明数据库的用于标记数据库对象名的符号，比如ORACLE就是双引号，MYSQL默认是`反引号； -->
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimiter" value="`"/>
        <property name="javaFileEncoding" value="UTF-8"/>
        <!-- 为模型生成序列化方法-->
        <plugin type="org.mybatis.generator.plugins.SerializablePlugin"/>
        <!-- 为生成的Java模型创建一个toString方法 -->
        <plugin type="org.mybatis.generator.plugins.ToStringPlugin"/>
        <!--生成mapper.xml时覆盖原文件-->
        <plugin type="org.mybatis.generator.plugins.UnmergeableXmlMappersPlugin" />
        <!--可以自定义生成model的代码注释-->
        <commentGenerator type="com.aircas.mbg.CommentGenerator">
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true"/>
            <property name="suppressDate" value="true"/>
            <property name="addRemarkComments" value="true"/>
        </commentGenerator>
        <!--配置数据库连接-->
        <jdbcConnection driverClass="${jdbc.driverClass}"
                        connectionURL="${jdbc.connectionURL}"
                        userId="${jdbc.userId}"
                        password="${jdbc.password}">
            <!--解决mysql驱动升级到8.0后不生成指定数据库代码的问题-->
            <property name="nullCatalogMeansCurrent" value="true"/>
        </jdbcConnection>
        <!--指定生成model的路径-->
        <!--windows和mac的路径分隔符不同-->
        <javaModelGenerator targetPackage="com.aircas.monitor.mbg.model" targetProject="src/main/java">
            <property name="constructorBased" value="true"/>
        </javaModelGenerator>
        <!--指定生成mapper.xml的路径-->
        <sqlMapGenerator targetPackage="mapper"
                         targetProject="src/main/resources"/>
        <!--指定生成mapper接口的的路径-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.aircas.monitor.mbg.mapper"
                             targetProject="src/main/java"/>
        <!--生成全部表对应代码，就把tableName设为%-->
        <table schema="root" tableName="%">
            <generatedKey column="id" sqlStatement="MySql" identity="true"/>
        </table>
    </context>
</generatorConfiguration>
```
> **温馨提示：**
> - 如果在Spring中想要生成的类，属性为驼峰的表示形式（例如：NetData），那么在数据库中的表名，字段名，请用下划线的形式定义（例如：net_data）。
> - Generator会生成model类，所以使用数据库的操作时，最好一直使用生成的model，防止来回转换。
> - constructorBased作用是能够在生成的model中，自动生成构造函数。
> - table标签作用是设置自动生成代码的表，如果仅仅想自动生成几个表，而不是所有的表，请自定定义tableName。如果想生成所有的表，则可以把tableName定义为%。


4. generator.properties配置属性
这个只能配置在properties中，提供给generatoeConfig.xml文件读取，如果配置在application.yml中，xml文件不能读取。  
```
jdbc.driverClass=com.mysql.cj.jdbc.Driver
jdbc.connectionURL=jdbc:mysql://localhost:3306/***
jdbc.userId=****
jdbc.password=****
```

5. CommentGenerator自定义注释生成器   
```
public class CommentGenerator extends DefaultCommentGenerator {
    private boolean addRemarkComments = false;
    /**
     * 设置用户配置的参数
     */
    @Override
    public void addConfigurationProperties(Properties properties) {
        super.addConfigurationProperties(properties);
        this.addRemarkComments = StringUtility.isTrue(properties.getProperty("addRemarkComments"));
    }
    /**
     * 给字段添加注释
     */
    @Override
    public void addFieldComment(Field field, IntrospectedTable introspectedTable,
                                IntrospectedColumn introspectedColumn) {
        String remarks = introspectedColumn.getRemarks();
        //根据参数和备注信息判断是否添加备注信息
        if (addRemarkComments && StringUtility.stringHasValue(remarks)) {
            addFieldJavaDoc(field, remarks);
        }
    }
    /**
     * 给model的字段添加注释
     */
    private void addFieldJavaDoc(Field field, String remarks) {
        //文档注释开始
        field.addJavaDocLine("/**");
        //获取数据库字段的备注信息
        String[] remarkLines = remarks.split(System.getProperty("line.separator"));
        for (String remarkLine : remarkLines) {
            field.addJavaDocLine(" * " + remarkLine);
        }
        addJavadocTag(field, false);
        field.addJavaDocLine(" */");
    }
}
```

### 运行Mybatis Generator

Generator会根据数据库  
❊ 生成对应的model和modelExample。例如：数据库名称为net_data,会在前面新建的文件夹mbg/model中生成NetData和NetDataExample类文件。生成的类文件中包含
<code> 数据库表中每个字段所对应的数据类型，相当于比着数据库表，创建了一个对应的对象，用来操作数据库。</code>  
❊ modelExample，提供一些基础的方法，能够直接调用。  
❊ mbg/xxMapper，接口文件，实现一些简单的功能。  
❊ mapper.xml，接口实现文件，里面是对数据库操作的SQL，用来实现xxMapper接口。 

### modelExample使用

自动生成的Mapper接口中的方法有很多是基于example作为参数的，example是提供条件添加的类，将所需要的条件全部添加到example中。

```
NetDataExample netDataExample = new NetDataExample();
netDataExample.createCriteria().andDstIpEqualTo("117.18.237.29");
List list = netDataMapper.selectByExample(netDataExample);
```


### mapper接口使用

生成的接口文件中的方法基本是一样的，都是

```
public interface NetDataMapper {
    long countByExample(NetDataExample example);

    int deleteByExample(NetDataExample example);

    int deleteByPrimaryKey(Integer id);

    int insert(NetData record);

    int insertSelective(NetData record);

    List<NetData> selectByExample(NetDataExample example);

    NetData selectByPrimaryKey(Integer id);

    int updateByExampleSelective(@Param("record") NetData record, @Param("example") NetDataExample example);

    int updateByExample(@Param("record") NetData record, @Param("example") NetDataExample example);

    int updateByPrimaryKeySelective(NetData record);

    int updateByPrimaryKey(NetData record);
}
```
每个方法进行学习：

```
$ countByExample : 使用example提供的条件，来统计
```
```
$ deleteByExample : 使用example提供的条件，删除
```
```
$ deleteByPrimaryKey : 通过主键删除
```
```
$ insert : 插入全部的model，使用生成的model
insert into net_data (net_type, src_ip, dst_ip, src_port, dst_port, packets, data_bytes, date) values (?, ?, ?, ?, ?, ?, ?, ?) 
```
```
$ insertSelective : 只插入有数据的model参数
Preparing: insert into net_data ( data_bytes, date ) values ( ?, ? ) 
```
```
$ selectByExample : 使用example提供的条件，查询
```
```
$ selectByPrimaryKey : 使用主键查询
```
```
$ updateByExampleSelective : 更新，sql只有数据的model
```
```
$ updateByExample : 使用example提供的条件，更新
```
```
$ updateByPrimaryKeySelective : 
```
```
$ updateByPrimaryKey : 
```



### 附加

#### Mybatis的CRUD操作

parameterType:参数的类型
resultType:查询结果的类型

```
<!-- 添加 -->
 
<insert id="insertUser" parameterType="User">
 
   insert into user values(null,#{name},#{age},#{birthday})
 
</insert>
 
 
parameterType    参数类型
 
#{name}  #{age}  #{birthday}
 
 name,age,birthday 是 User的属性
 
 
 
<!-- 更新 -->
 
<update id="updateUser" parameterType="User">
 
    update user set name=#{name},age=#{age},birthday=#{birthday} where id=#{id}
 
</update>
 
 
<!-- 删除 -->
 
<delete id="deleteUser" parameterType="User">
 
    delete from user where id=#{id}
 
</delete>
 
 
 
  <!-- 根据主键查询 -->
 
<select id="selectUser" parameterType="int" resultType="User">
 
select * from user where id = #{id}
 
</select>

```
#### Mybatis if

```
<select id="countByExample" parameterType="com.aircas.monitor.mbg.model.NetDataExample" resultType="java.lang.Long">
    select count(*) from net_data
    <if test="_parameter != null">
      <include refid="Example_Where_Clause" />
    </if>
</select>
```

可以看到有个<if test="_parameter != null" >，如果只有一个参数，那么_parameter 就代表该参数，如果有多个参数，那么_parameter 可以get(0)得到第一个参数。include标签引用，可以复用SQL片段。

```
  <sql id="Example_Where_Clause">
    <where>
      <foreach collection="oredCriteria" item="criteria" separator="or">
        <if test="criteria.valid">
          <trim prefix="(" prefixOverrides="and" suffix=")">
            <foreach collection="criteria.criteria" item="criterion">
              <choose>
                <when test="criterion.noValue">
                  and ${criterion.condition}
                </when>
                <when test="criterion.singleValue">
                  and ${criterion.condition} #{criterion.value}
                </when>
                <when test="criterion.betweenValue">
                  and ${criterion.condition} #{criterion.value} and #{criterion.secondValue}
                </when>
                <when test="criterion.listValue">
                  and ${criterion.condition}
                  <foreach close=")" collection="criterion.value" item="listItem" open="(" separator=",">
                    #{listItem}
                  </foreach>
                </when>
              </choose>
            </foreach>
          </trim>
        </if>
      </foreach>
    </where>
  </sql>
```

&lt;trim>:
mybatis的trim标签一般用于去除sql语句中多余的and关键字，逗号，或者给sql语句前拼接 “where“、“set“以及“values(“ 等前缀，或者添加“)“等后缀，可用于选择性插入、更新、删除或者条件查询等操作。

**Criteria是Mybatis Generator添加条件的地方，继续学习netDataExample就可以知道**
#### 监听SQL

1. 在日志中打印SQL
```
mybatis:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```
在application.yml中增加配置，就能够监听打印sql。

2. 使用druid进行监听
应用运行起来之后，能通过地址/Druid访问,能够进入到连接池控制界面，等改天有时间再学习Druid～～
```
$ http://localhost:8080/druid/index.html
```

![upload successful](/images/pasted-42.png)


#### 自定义sql

使用mybatis generator时，如果不选择覆盖之前的mapper.xml，就会造成代码重复。选择设置覆盖时，又不能在.xml中添加自定义的sql，最好的解决方法就是，重新自定义一个
```
public NetDataMapperCoustom extend NetDataMapper
```
然后自定义一个NetDataMapperCoustom.xml，来单独实现sql，这个就不会受到mybatis generator的影响。

#### @Params报错

**报错：程序包org.apache.ibatis.annotations不存在**
添加依赖：
```
<!-- https://mvnrepository.com/artifact/org.apache.ibatis/ibatis-core -->
        <dependency>
            <groupId>org.apache.ibatis</groupId>
            <artifactId>ibatis-core</artifactId>
            <version>3.0</version>
        </dependency>
```