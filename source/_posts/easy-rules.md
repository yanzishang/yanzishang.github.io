---
title: java规则引擎之easy rules源码阅读
date: 2017-09-18 20:40:01
tags: java-library
---


### 简介

easy rules是一个简单而强大的java规则引擎，它有以下特性：

- 轻量级框架，学习成本低
- 基于POJO
- 为定义业务引擎提供有用的抽象和简便的应用
- 从原始的规则组合成复杂的规则

它主要包括几个主要的类或接口：Rule,RulesEngine,RuleListener,Facts  
还有几个主要的注解：@Action,@Condition,@Fact,@Priority,@Rule

### 实现

使用easy rules有两种方式：

1. 实现Rule接口，并实现其evaluate和execute方法。
2. 使用@Rule注解修饰POJO

### 源码


#### Rule.java
```java
public interface Rule extends Comparable<Rule> {

    /**
     * 默认的规则名称
     */
    String DEFAULT_NAME = "rule";

    /**
     * 默认的规则描述
     */
    String DEFAULT_DESCRIPTION = "description";

    /**
     * 默认的规则优先级
     */
    int DEFAULT_PRIORITY = Integer.MAX_VALUE - 1;

    getter and setter...

    /**
     * 规则引擎判断条件
     * 如果提供的facts被应用到规则上返回true，否则返回false
     */
    boolean evaluate(Facts facts);

    /**
     * 规则引擎判断条件返回true后，执行此方法
     */
    void execute(Facts facts) throws Exception;

}
```
由上面代码可以知道：一个规则由名称、描述、优先级三个属性和判断、执行两个方法组成，实现Rule接口，和使用@Rule,@Condition,@Action,@Priority,@Fact注解的效果是一样的。


#### RulesEngine.java
```java
public interface RulesEngine {

    /**
     * 返回规则引擎的参数
     */
    RulesEngineParameters getParameters();

    /**
     * 返回已注册的规则监听器的列表
     */
    List<RuleListener> getRuleListeners();

    /**
     * 在给定的因素上开启所有已注册的规则
     */
    void fire(Rules rules, Facts facts);

    /**
     * 检查规则和因素是否符合
     */
    Map<Rule, Boolean> check(Rules rules, Facts facts);
}
```
RulesEngine负责检查和开启规则，同时可以得到规则引擎的参数和规则监听器列表


#### RuleListener.java
```java
public interface RuleListener {

    /**
     * 规则条件判断之前的触发器
     */
    boolean beforeEvaluate(Rule rule, Facts facts);

    /**
     * 规则条件判断之后的触发器
     */
    void afterEvaluate(Rule rule, Facts facts, boolean evaluationResult);

    /**
     * 规则执行之前的触发器
     */
    void beforeExecute(Rule rule, Facts facts);

    /**
     * 规则执行成功之后的触发器
     */
    void onSuccess(Rule rule, Facts facts);

    /**
     * 规则执行失败之后的触发器
     */
    void onFailure(Rule rule, Facts facts, Exception exception);

}
```
RuleListener在规则执行的4个阶段加上了触发器，可以灵活地控制规则执行结果


#### Facts.java
```java
public class Facts implements Iterable<Map.Entry<String, Object>> {

    private Map<String, Object> facts = new HashMap<>();

    /**
     * 在工作空间放置一个因素
     */
    public Object put(String name, Object fact) {
        Objects.requireNonNull(name);
        return facts.put(name, fact);
    }

    /**
     * 删除因素
     */
    public Object remove(String name) {
        Objects.requireNonNull(name);
        return facts.remove(name);
    }

    /**
     * 通过name得到因素
     */
    public Object get(String name) {
        Objects.requireNonNull(name);
        return facts.get(name);
    }

    /**
     * 以map形式返回因素
     */
    public Map<String, Object> asMap() {
        return facts;
    }
    ...
}
```
Facts就是一个hashmap，通过注解@Fact(String value)，其中的value是map的key，可以拿到Facts中的value

更多请去easy-rules的github：https://github.com/j-easy/easy-rules

