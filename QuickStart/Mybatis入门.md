Mybatis入门

- 核心组件

  - SqlSessionFactory:用于创建SqlSession的工厂类
  - SqlSession:MyBatis的核心组件，用于向数据库执行SQL
  - 主配置文件：XML配置文件，可以对MyBatis的底层行为作出详细的配置
  - Mapper接口：就是DAO接口
  - Mapper映射器：用于编写SQL,并将SQL和实体类映射的组件，采用XML,注解均可实现

- 使用Mybatis

  1. 在pom.xml文件中添加Mybatis依赖

  ```xml
  <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
  	<artifactId>mybatis-spring-boot-starter</artifactId>
  	<version>1.3.0</version>
  </dependency>
  ```

  2. 在application.properties中配置Mybatis

  ```properties
  #DataSourceProperties
  spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
  spring.datasource.url=jdbc:mysql://localhost:3306/community?characterEncoding=utf-8&useSSL=false&serverTimezone=Hongkong
  spring.datasource.username=root
  spring.datasource.password=123456
  spring.datasource.type=com.zaxxer.hikari.HikariDataSource
  spring.datasource.hikari.maximum-pool-size=15
  spring.datasource.hikari.minimum-idle=5
  spring.datasource.hikari.idle-timeout=30000
  
  #MybatisProperties
  mybatis.mapper-locations=classpath:mapper/*.xml
  mybatis.type-aliases-package=com.nowcoder.community.entity
  mybatis.configuration.use-generated-keys=true
  mybatis.configuration.map-underscore-to-camel-case=true
  ```

  或者在XML文件中对Mybatis进行核心配置

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE configuration
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
  <configuration>
    <environments default="development">
      <environment id="development">
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
      <mapper resource="org/mybatis/example/BlogMapper.xml"/>
    </mappers>
  </configuration>
  ```

  3. 编写实体类User

  ```java
  package com.nowcoder.community.entity;
  
  import java.util.Date;
  
  public class User {
  
      private int id;
      private String username;
      private String password;
      private String salt;
      private String email;
      private int type;
      private Date createTime;
      ....省略Setter，Getter方法和toString()方法
  }
  ```

  4. 编写UserMapper接口

  ```java
  package com.nowcoder.community.dao;
  
  import com.nowcoder.community.entity.User;
  import org.apache.ibatis.annotations.Mapper;
  
  @Mapper
  public interface UserMapper {
      User selectById(int id);
      User selectByName(String username);
      User selectByEmail(String email);
      int insertUser(User user);
      int updateStatus(int id,int status);
      int updateHeader(int id,String headerUrl);
      int updatePassword(int id,String password);
  }
  ```

  5. 编写基于XML的UserMapper映射语句

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE mapper
          PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="com.nowcoder.community.dao.UserMapper">
  
      <sql id="insertFields">
          username,password,salt,email,type,status,activation_code,header_url,create_time
      </sql>
      <sql id="selectFields">
          id,username,password,salt,email,type,status,activation_code,header_url,create_time
      </sql>
  
      <select id="selectById" resultType="User">
          select <include refid="selectFields"></include>
          from user
          where id = #{id}
      </select>
      <select id="selectByName" resultType="User">
          select <include refid="selectFields"></include>
          from user
          where username= #{username}
      </select>
      <select id="selectByEmail" resultType="User">
          select <include refid="selectFields"></include>
          from user
          where email = #{email}
      </select>
  
      <insert id="insertUser" parameterType="User" keyProperty="id">
          insert into user(<include refid="insertFields"></include>)
          values(#{username},#{password},#{salt},#{email},#{type},#{status},#{activationCode},#{headerUrl},#{createTime})
      </insert>
  
      <update id="updateStatus">
          update user set status= #{status} where id=#{id}
      </update>
      <update id="updateHeader">
          update user set header_url= #{headerUrl} where id=#{id}
      </update>
      <update id="updatePassword">
          update user set password= #{password} where id=#{id}
      </update>
  </mapper>
  ```

  

