---
layout: post
title: Spring项目中@RequestBody如何接受Abstract或Interface的类型的参数
date: 2023/02/24
categories: blog
tags: [springboot]
description: 本文演示环境springboot 2.7.6

---

## 1、场景

在实际生产过程中，我们可能需要在对象中建立一个Abstract或Interface的类型属性用来代表可变不确定的属性。

比如我们有一个方法是生产车辆
    
    @PostMapping
    public String createCar(@RequestBody Car car) {
        ...
    }

    public class Car {
        private String name;
        private Tyre tyre; // 汽车配件 - 轮胎
        ...
    }
    
    public interface Tyre extends Serializable {
    }

    public class MICHELINTyre implements Tyre {
        //米其林轮胎
        ...
    }

    public class BRIDGESTONETyre implements Tyre {
        //普利司通轮胎
        ...
    }

因为对于不同的汽车，轮胎使用的配件类型其实也是有部分区别的，所以这里我们会使用到Abstract或Interface来定义这个Tyre.

当我们的程序运行起来的时候，系统这时就会出现异常：

    Caused by: com.fasterxml.jackson.databind.JsonMappingException: Can not construct instance of com.xxx.Tyre, 
    problem: abstract types either need to be mapped to concrete types, have custom deserializer, or be instantiated with additional type information at [Source: java.io.PushbackInputStream@4e40388; line: 1, column: 2]

这是因为当实际请求json过来的时候，Jackson不知道该采用Tyre的哪个子类来对{"name": "宝马", "tyre": {...}}进行反序列化。

## 2、解决方法

首先修改刚才定义的interface

    public interface Tyre extends Serializable {
        Type getType();
        
        enum Type{
            MICHELIN,
            BRIDGESTON
        }
    }

接着我们修改实现类

    public class MICHELINTyre implements Tyre {
        public Type getType(){
            return Tyre.Type.MICHELIN;
        }
        //米其林轮胎
        ...
    }

    public class BRIDGESTONETyre implements Tyre {
        public Type getType(){
            return Tyre.Type.BRIDGESTONE;
        }
        //普利司通轮胎
        ...
    }

最后我们需要在定义interface上添加@JsonSubTypes以及@JsonTypeInfo注解
    
    import com.fasterxml.jackson.annotation.JsonSubTypes;
    import com.fasterxml.jackson.annotation.JsonTypeInfo;

    @JsonTypeInfo(use = JsonTypeInfo.Id.NAME, include = JsonTypeInfo.As.PROPERTY, property = "type")
    @JsonSubTypes({
        @JsonSubTypes.Type(value = MICHELINTyre.class, name = "MICHELIN"),
        @JsonSubTypes.Type(value = BRIDGESTONETyre.class, name = "BRIDGESTONE")
    })
    public interface Tyre extends Serializable {
        Type getType();
        
        enum Type{
            MICHELIN,
            BRIDGESTON
        }
    }

至此，我们再运行程序的时候，程序运行OK。

    


