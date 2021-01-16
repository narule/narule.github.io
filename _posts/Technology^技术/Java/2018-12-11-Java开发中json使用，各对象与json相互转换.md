---
title: Java开发中json使用，各对象与json相互转换
author: Narule
date: 2018-12-11 20:10:00 +0800
categories: [Blogging, Technology^技术, Java]
tags: [writing, Java, Json]

---



Json：一种网络通信使用的数据格式，因为便于解析，比较流行，对象可以转为json，同样json也可以转对象。

下面介绍下Json工具的简单使用（fastjson && jackson）。''

## FastJson

　　阿里的json数据解析工具包，国内比较流行，用的较多。

##### 　　对象转json字符串

　　　　JSON.toJSONString(user);

##### 　　对象转json对象

　　　　(JSONObject)JSON.toJSON(user);

##### 　　json字符串转对象

　　　　JSON.parseObject(jsonString, User.class);　　

##### 　　json对象转对象 

　　　　User javaObject = JSON.toJavaObject(jsonObject, User.class);

##### 　　list 转 json

　　　　String jsonString = JSON.toJSONString(users);

#####  　json 转 list

　　　　List<User> parseArray = JSON.parseArray(jsonString, User.class);

##### 　　完整示例代码(需要引入fastjson的jar包)：

```java
package test;//JsonTest.java 文件所在位置:test包

import java.util.ArrayList;
import java.util.List;


import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;


public class JsonTest {//User 类
    static class User{
        private Integer age;
        private String name;
        public Integer getAge() {
            return age;
        }
        public void setAge(Integer age) {
            this.age = age;
        }
        public String getName() {
            return name;
        }
        public void setName(String name) {
            this.name = name;
        }
        @Override
        public String toString() {
            return "User [age=" + age + ", name=" + name + "]";
        }
        
    }
    
    public static void main(String[] args) {
        // new 一个User对象
        User user = new User();
        user.setAge(1);
        user.setName("xiaojun");
        
        //对象转json字符串
        String jsonString = JSON.toJSONString(user);
        System.out.println("对象转json字符串:\t" + jsonString);
        //对象转json对象
        user.setAge(2);
        JSONObject jsonObject = (JSONObject)JSON.toJSON(user);
        System.out.println("对象转json:\t" + jsonObject);
        
        //json字符串转对象
        User parseObject = JSON.parseObject(jsonString, User.class);
        System.out.println("json字符串转对象:\t" + parseObject);
        
        //json对象转对象
        User javaObject = JSON.toJavaObject(jsonObject, User.class);
        System.out.println("json对象转对象:\t" + javaObject);
        
        User user2 = new User();
        user2.setAge(9);
        user2.setName("xiaochen");
        ArrayList<User> users = new ArrayList<>();
        users.add(user2);
        users.add(user);
        System.out.println("list 转json============\t");
        System.out.println("准备转json的list:\t" + users);
        
        //list 转json
        String jsonString2 = JSON.toJSONString(users);
        System.out.println("list转json:\t" + jsonString2);
        
        //json 转list
        List<User> parseArray = JSON.parseArray(jsonString2, User.class);
        System.out.println("json转list:\t" + parseArray);
    }
    
}
```

　　

 

执行main方法Console控制台打印结果如下：

![img](https://img2018.cnblogs.com/blog/1436620/201812/1436620-20181218152550378-1612908110.png)

 

## Jackson 

　与FastJson一样,Jackson 应该比fastjson 出现更早,api用法有些不一样,但是使用效果差不多

　使用jackson编写的json工具类，JsonUtil：

```java
import java.util.List;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.JavaType;
import com.fasterxml.jackson.databind.ObjectMapper;

/**
 *
 */
public class JsonUtil {

    // 定义jackson对象
    private static final ObjectMapper MAPPER = new ObjectMapper();

    /**
     * 将对象转换成json字符串。
     * <p>Title: pojoToJson</p>
     * <p>Description: </p>
     * @param data
     * @return
     */
    public static String objectToJson(Object data) {
        try {
            String string = MAPPER.writeValueAsString(data);
            return string;
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 将json结果集转化为对象
     *
     * @param jsonData json数据
     * @param clazz 对象中的object类型
     * @return
     */
    public static <T> T jsonToPojo(String jsonData, Class<T> beanType) {
        try {
            T t = MAPPER.readValue(jsonData, beanType);
            return t;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 将json数据转换成pojo对象list
     * <p>Title: jsonToList</p>
     * <p>Description: </p>
     * @param jsonData
     * @param beanType
     * @return
     */
    public static <T>List<T> jsonToList(String jsonData, Class<T> beanType) {
        JavaType javaType = MAPPER.getTypeFactory().constructParametricType(List.class, beanType);
        try {
            List<T> list = MAPPER.readValue(jsonData, javaType);
            return list;
        } catch (Exception e) {
            e.printStackTrace();
        }

        return null;
    }

}
```

 

 

本文不定期更新