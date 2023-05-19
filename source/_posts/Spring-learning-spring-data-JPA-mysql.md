---
title: Spring学习：数据访问层采用JPA集成MySQL
date: 2020-12-16 13:58:00
tags: 
  - Java
  - Spring
  - 学习笔记
categories:
  - Java
  - Spring
---

# 数据访问层采用JPA集成MySQL
## 简介

JPA采用仓库模式实现数据持久化处理，它屏蔽底层处理细节，简化开发应用程序时对数据源处理的过程。数据访问层采用JPA，主要以Java实体类为核心，实体类封装数据字典映射数据表，结合仓库类实现数据的CRUD. 需要知道的是JPA是Java Persistence API的缩写。

MySQL是常见的开源免费关系型数据库，应用广泛支持良好，一般作为项目开发的首选。此次学习我们采用IDEA搭建Spring Data JPA + MySQL项目。

## 项目搭建

1. 创建新项目

{% asset_img image-20201216143045844.png %}

2. 初始化项目，设置Group、Artifact、Package，项目采用Maven构建，Java版本采用v8

{% asset_img image-20201216143226651.png %}

3. 项目依赖选择：Spring Boot DevTools、Lombok、Spring Web、Thymeleaf、Spring Data JPA、MySQL Driver.

{% asset_img image-20201216143548925.png %}

4. 完成项目创建

{% asset_img image-20201216143827567.png %}

## 项目配置

1. 修改application.properties为application.yml

{% asset_img image-20201216144257396.png %}

2. 设置配置文件

   ```yaml
   server:
     port: 10016
   spring:
     datasource:
       driver-class-name: com.mysql.cj.jdbc.Driver
       url: jdbc:mysql://localhost:3306/test_db?useTimezone=true&serverTimezone=GMT%2b8&characterEncoding=utf8
       username: testUser
       password: testPwdxxxxxx
     jpa:
       database: mysql
       show-sql: true
       generate-ddl: true
   
   ```


3. pom依赖配置

   ```xml
   <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-data-jpa</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-thymeleaf</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
   
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-devtools</artifactId>
               <scope>runtime</scope>
               <optional>true</optional>
           </dependency>
           <dependency>
               <groupId>mysql</groupId>
               <artifactId>mysql-connector-java</artifactId>
               <scope>runtime</scope>
           </dependency>
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
               <optional>true</optional>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-test</artifactId>
               <scope>test</scope>
           </dependency>
       </dependencies>
   
       <build>
           <plugins>
               <plugin>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-maven-plugin</artifactId>
                   <configuration>
                       <excludes>
                           <exclude>
                               <groupId>org.projectlombok</groupId>
                               <artifactId>lombok</artifactId>
                           </exclude>
                       </excludes>
                   </configuration>
               </plugin>
           </plugins>
       </build>
   ```

## 项目编码

### 项目分层

在项目正式编码前，我们简单的定义下各个package，用于项目分层化分解。

config包：存放项目启动相关配置类；

controller包：存放spring mvc Controller类；

domain.entity包：存放数据表相关实体类，实体类用于映射数据表；

domain.enums包：存放枚举类型；

domain.model包：存放自定义模型，用于封装数据表以外的数据结构；

repository包：存放仓库类，提供数据的CRUD功能；

{% asset_img image-20201216164013081.png %}

### 定义实体类

使用JPA作为数据访问层的第一步是封装定义数据实体类。在UserEntity实体类中，封装映射t_user表相关字段。

```java
package org.banana.authserver.domain.entity;

import javax.persistence.*;
import java.util.Date;

/**
 * @author YuanRui
 * @since 2020-12-16 15:38:51
 */
@Table(name = "t_user")
@Entity
public class UserEntity {
    private Integer id;
    private String name;
    private String password;
    private Boolean disabled;
    private Boolean expired;

    /**
     * 获取 编号
     */
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "id", nullable = false)
    public Integer getId() {
        return this.id;
    }

    /**
     * 设置 编号
     */
    public void setId(Integer id) {
        this.id = id;
    }

    /**
     * 获取 用户名
     */
    @Basic
    @Column(name = "name", nullable = true, length=255)
    public String getName() {
        return this.name;
    }

    /**
     * 设置 用户名
     */
    public void setName(String name) {
        this.name = name;
    }

    /**
     * 获取 密码
     */
    @Basic
    @Column(name = "password", nullable = true, length=255)
    public String getPassword() {
        return this.password;
    }

    /**
     * 设置 密码
     */
    public void setPassword(String password) {
        this.password = password;
    }

    /**
     * 获取 是否禁用
     */
    @Basic
    @Column(name = "disabled", nullable = false)
    public Boolean getDisabled() {
        return this.disabled;
    }

    /**
     * 设置 是否禁用
     */
    public void setDisabled(Boolean disabled) {
        this.disabled = disabled;
    }

    /**
     * 获取 是否过期
     */
    @Basic
    @Column(name = "expired", nullable = false)
    public Boolean getExpired() {
        return this.expired;
    }

    /**
     * 设置 是否过期
     */
    public void setExpired(Boolean expired) {
        this.expired = expired;
    }

}
```

### 创建仓库接口

创建UserRepository接口，使用@Repository注解，继承CrudRepository、JpaRepository、JpaSpecificationExecutor，实现对数据表t_user的CRUD.

```java
package org.banana.authserver.repository;

import org.banana.authserver.domain.entity.UserEntity;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Repository;

/**
 * @author YuanRui
 * @since 2020-12-16 15:33:08
 */
@Repository
public interface UserRepository extends CrudRepository<UserEntity, Integer>, JpaRepository<UserEntity, Integer>, JpaSpecificationExecutor<UserEntity> {

}
```

### 创建控制器类

创建HomeController，使用@Controller注解，创建一个action方法: index. index方法内部调用仓库接口，获取所有用户列表，并将模型数据传递到页面。

```
package org.banana.authserver.controller;

import org.banana.authserver.domain.entity.UserEntity;
import org.banana.authserver.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

import java.util.ArrayList;
import java.util.List;

/**
 * @author yuanrui@live.cn
 * @since 2020/12/16 15:47
 */
@Controller
public class HomeController {

    @Autowired
    UserRepository userRepository;

    @GetMapping( value = { "/", "/index"})
    public String index(Model model) {
        List<UserEntity> all = userRepository.findAll();

        List list = new ArrayList<String>();
        for (UserEntity ent : all){
            list.add(ent.getName());
        }
        String users = String.join(",", list);

        model.addAttribute("users", users);

        return "index";
    }
}
```

在资源文件夹resources的templates中创建index.html文件，编辑代码：

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>系统首页</title>
</head>
<body>
系统用户：[[${users}]]
</body>
</html>
```

Shift+F9调试项目，在浏览器中打开：http://localhost:10016/

{% asset_img image-20201216163636095.png %}

### MySQL脚本

```sql
-- ----------------------------
-- Table structure for t_user
-- ----------------------------
DROP TABLE IF EXISTS `t_user`;
CREATE TABLE `t_user`  (
  `id` int NOT NULL AUTO_INCREMENT COMMENT '编号',
  `name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '用户名',
  `password` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '密码',
  `disabled` bit(1) NOT NULL DEFAULT b'0' COMMENT '是否禁用',
  `expired` bit(1) NOT NULL DEFAULT b'0' COMMENT '是否过期',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 3 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of t_user
-- ----------------------------
INSERT INTO `t_user` VALUES (1, 'test', '123456', b'0', b'0');
INSERT INTO `t_user` VALUES (2, 'abc', '123456', b'0', b'0');
```

