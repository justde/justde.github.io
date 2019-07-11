---
layout: post
title: '设计模式学习笔记之工厂模式'
date: 2018-10-18
author: Justd
cover: '/assets/img/2018-10/16/tag.jpg'
tags: Java   
---
本文讲述一个披萨的诞生

我有一家披萨店，顾客来点了想吃的品种，然后我要准备材料、烘烤、剪切、帮顾客打包。这个过程用代码怎么实现呢？
# 传统方式   
首先定义好准备、烘烤、剪切和打包这些动作   
``` java
public abstract class Pizza {
    protected String name;

    public abstract void prepare();
    public void bake()
    {
        System.out.println(name+" baking;");
    }
    public void cut()
    {
        System.out.println(name+" cutting;");
    }
    public void box()
    {
        System.out.println(name+" boxing;");
    }
    public void setname(String name)
    {
        this.name=name;
    }
}
```
然后定义一下都可以做哪些披萨种类   
```Java
public class CheesePizza extend Pizza{
    @Override
    public void prepare(){
        super.setname("CheesePizza");
        System.out.println("CheesePizza");
    }
}
```
```java
public class GreekPizza extends Pizza {

    @Override
    public void prepare() {
        super.setname("GreekPizza");
        System.out.println("GreekPizza");
    }
}
```
准备工作做完，就等着顾客来点餐了
```Java
public class OrderPizza {
    public OrderPizza() {
        String type = null;
        Pizza pizza = null;
        do {
            type = getType();
            if ("greek".equals(type)){
                pizza = new GreekPizza();
            }else if ("cheese".equals(type)){
                pizza = new CheesePizza();
            }else {
                break;
            }
            pizza.prepare();
            pizza.bake();
            pizza.cut();
            pizza.box();
        }while (true);
    }


    private String getType(){
        String type = null;
        BufferedReader br = null;
        try {
            br = new BufferedReader(new InputStreamReader(System.in));
            System.out.println("input pizza type:  ");
            type = br.readLine();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return type;
    }
}
```
整个过程如下：   
顾客根据披萨的种类进行点餐，然后根据类型进行创建披萨对象，进行准备、烘焙、剪切、打包

很明显，这样设计是有问题的。如果店里有了新品，或者下架某种类披萨，就要修改OrderPizza类中根据顾客输入来创建对象这段代码，显然是违背[上文](https://justed.github.io/2018/10/17/DesingPatterns-outline.html)提到的开放封闭原则。如下代码，现在我们已经知道哪些会改变，哪些不会改变，是时候使用封装了。      
```Java
//需要修改
if ("greek".equals(type)){
    pizza = new GreekPizza();
else if ("cheese".equals(type)){
    pizza = new CheesePizza();
}else {
    break;
}
//不需要修改
pizza.prepare();
pizza.bake();
pizza.cut();
pizza.box();
```    
# 简单披萨工厂    
现在最好将创建对象移到orderPizza()之外，但怎么做呢？我们可以把创建披萨的代码移到另一个对象中，由这个新对象专职创建披萨。
我们称这个新对象为“工厂”。    
工厂（factory）处理创建对象的细节。一旦有了SimplePizzaFactory，orderPizza()就变成此对象的客户。当需要披萨时，就叫披萨工厂做一个。那些orderPizza()方法需要知道希腊披萨或者蛤蜊披萨的日子一去不复返了。现在orderPizza()方法只关心从工厂得到了一个披萨，而这个披萨实现了Pizza接口，所以它可以调用prepare()、bake()、cut()、box()来分别进行准备、烘烤、切片、装盒。    
>SimplePizzaFactory是我们的新类，它负责为客户创建披萨   

```Java
public class SimpleFactoryPizza {
    public Pizza createPizza(String type){
        Pizza pizza = null;
        if ("greek".equals(type)) {
            pizza=new GreekPizza();
        }else if ("cheese".equals(type)) {
            pizza=new CheesePizza();
        }else if ("beef".equals(type)) {
            pizza=new BeefPizza();
        }
        return pizza;
    }
}
```
>新的order只需构造时传入一个工厂，然后带入订单类型来使用工厂创建披萨，代替之前具体的实例化    

```Java
public class OrderPizza {
    SimpleFactoryPizza factory;

    public OrderPizza(SimpleFactoryPizza factory) {
        this.factory = factory;
    }

    Pizza orderPizza(String type) {
        Pizza pizza = null;
        pizza = factory.createPizza(type);
        if (pizza != null) {
            pizza.prepare();
            pizza.bake();
            pizza.cut();
            pizza.box();
        }
        return pizza;
    }
}

```
虽然看似代码同传统方式一样，但是我们已经将变化的代码抽取出来了，在OrderPizza中我们无需再次修改，此时我们已经将变化的和不变化的隔离开来了。   

# 工厂模式     
如果披萨店生意越来越好，考虑开几家加盟店。身为加盟公司的经营者，你希望确保加盟店的运营质量，还希望各地的披萨有自己不同的区域特点。在推广SimpleFactoryPizza时，发现加盟店采用的统一的工厂创建的披萨，但是其他部分却开始采用自创的流程，比如烘烤方式，包装方式等等。那么如果建立一个框架，约束关键步骤的同时又能保持一定的弹性呢？
### 修改给披萨店使用的框架   
有一个办法可以让披萨制作活动局限于OrderPizza类，同时让不同的店还拥有自己的特色。    
所要做的事情就是把createPizza()方法放回到OrderPizza中，不过得将它设置成抽象方法，然后为每个店铺创建一个OrderPizza的子类。
来看下修改后的OrderPizza：
```Java
public abstract class OrderPizza {
    public Pizza orderPizza(String type){
        Pizza pizza;
        //将创建方法从工厂对象中移回OrderPizza中
        pizza = createPizza(type);
        //没变过
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
        return pizza;
    }
    //创建工厂方法改为抽象的，方便子类修改
    abstract Pizza createPizza(String type);
}
```
现在已经有一个OrderPizza作为父类，让不同区域的店铺来继承OrderPizza，每个子类各自决定制作什么风味的披萨。   
### 允许子类（不同店铺）做决定
OrderPizza已经有一个不错的订单系统，由orderPizza()负责处理订单，而你希望所有加盟店对于订单的处理都能一致。
各个区域披萨店之间的差异在于他们制作披萨的风味（纽约披萨的薄脆、芝加哥披萨的饼厚等），我们现在要让现在createPizza()能够应对这些变化来负责创建正确种类的披萨。做法是让OrderPizza的各个子类负责定义自己的createPizza()方法。所以我们会得到一些OrderPizza具体的子类，每个子类都有自己的披萨变体，而仍然适合OrderPizza框架，并使用调试好的orderPizza()方法。    
```Java
// 如果加盟店为顾客提供纽约风味的披萨，就使用NyStyleOrderPizza，
// 因为此类的createPizza()方法会建立纽约风味的披萨
public class NyStyleOrderPizza extends OrderPizza{
 
    //子类自己定义创建披萨方法
    @Override
    Pizza createPizza(String type) {
        Pizza pizza = null;
        if (type.equals("cheese")) {
            pizza = new NyStyleCheesePizza();
        } else if (type.equals("veggie")) {
            pizza = new NyStyleVeggiePizza();
        }
        return pizza;
    }
}
 
// 类似的，利用芝加哥子类，我们得到了带芝加哥原料的createPizza()实现
public class ChicagoStyleOrderPizza extends OrderPizza{
 
    @Override
    Pizza createPizza(String type) {
        Pizza pizza = null;
 
        if (type.equals("cheese")) {
            pizza = new ChicagoCheesePizza();
        } else if (type.equals("veggie")) {
            pizza = new ChicagoVeggiePizza();
        }
        return pizza;
    }
}
```   
现在问题来了，OrderPizza的子类终究只是子类，如何能够做决定？在NyStyleOrderPizza类中，并没有看到任何做决定逻辑的代码。
关于这个方面，要从OrderPizza类的orderPizza()方法观点来看，此方法在抽象的OrderPizza内定义，但是只在子类中实现具体类型。
orderPizza()方法对对象做了许多事情（例如：准备、烘烤、切片、装盒），但由于Pizza对象是抽象的，orderPizza()并不知道哪些实际的具体类参与进来了。换句话说，这就是解耦（decouple）！
当orderPizza()调用createPizza()时，某个披萨店子类将负责创建披萨。做哪一种披萨呢？当然是由具体的披萨店决定。
那么，子类是实时做出这样的决定吗？不是，但从orderPizza()的角度看，如果选择在NyStyleOrderPizza订购披萨，就是由这个子类（NyStyleOrderPizza）决定。严格来说，并非由这个子类实际做“决定”，而是由“顾客”决定哪一家风味的披萨店才决定了披萨的风味。   
我们来看下如何调用：   
```Java
public class test {
    public static void main(String[] args) {
        OrderPizza orderPizza = new NyStyleOrderPizza();
        Pizza pi = orderPizza.orderPizza("cheese");
        System.out.println("-------");
        Pizza pizza = orderPizza.orderPizza("veggie");
    }
}
```



执行结果如下：   
```java
Preparing NyStyleCheesePizza baking;
Preparing NyStyleCheesePizza cutting;
Preparing NyStyleCheesePizza boxing;
-------
Preparing NyStyleVeggiePizza baking;
Preparing NyStyleVeggiePizza cutting;
Preparing NyStyleVeggiePizza boxing;
```

---
>本文代码例子来源：《head-first设计模式》  

设计模式学习系列：   
[基本概念](https://justed.github.io/2018/10/17/DesingPatterns-outline.html)    
[工厂模式](https://justed.github.io/2018/10/18/DesingPatterns-Factory.html)   
[装饰者模式](https://justed.github.io/2018/10/16/DesingPatterns-decorator.html)