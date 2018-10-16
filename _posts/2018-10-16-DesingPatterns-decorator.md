---
layout: post
title: '设计模式之装饰着模式'
date: 2018-10-16
author: Justd
cover: '/assets/img/2018-10/10/post-bg-idea.jpg'
tags: Java   
---

装饰者模式：动态的给一个对象添加一些额外的职责。就增加功能来说，它相比生成子类更为灵活   

下面通过一个例子来具体了解    

早餐店点餐系统：   
早点种类：手抓饼、鸡蛋灌饼、煎饼果子
可加配料：生菜、鸡蛋、火腿肠    
顾客首先选择饼的种类，然后在选择可加的配料，我们需要描述出顾客选择的早点种类和添加配料名称，及其花费的总价   

## 比较简陋的方案：  
![](/assets/img/2018-10/16/one.png)
首先定义了一个抽象的Food父类，里面定义出一个抽象的cost()方法用于返回消费的总价，用setDescription来描述各个消费商品的名称，使用getDescrption将客户的消费订单打印，后面又将大概的客户消费可能出现情况一一列举出来，比如手抓饼加蛋、煎饼果子加肠等。但是由于这个消费组合情况太多，没有用代码描述，各个消费情况集成Food类，实现cost()抽象方法，使用父类super.getPrice()返回总价，在构造中使用super.setDescriptin()和super.setPrice()分别去设置自己的产品名称及其价格。图中我们可以看到加入有许多种情况，我们需要定义太多的类。真正实现中太过麻烦，每个顾客的组合都要重新定义。

## 稍微好点的方案：   
![](/assets/img/2018-10/16/two.png)   
从上图可以看到基本和第一种方案基本类似，只是将配料的种类在父类中体现，以boolean类型修饰是否添加某种配料。如果顾客需要在饼中加入鸡蛋，只需要调用setEgg()方法，egg为true,使用hasEgg()方法判断是否有某种调料，但是就像图中所示，如果增加配料种类，则需要修改父类方法，并且无法添加双份配料。

## 本章重点--使用装饰者模式解决上述问题   
``` java
public abstract class Food {
    public String description = "";
    private int price = 0;

    public abstract int cost();

    public String getDescription() {
        return description + "---" + this.getPrice();
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public int getPrice() {
        return price;
    }

    public void setPrice(int price) {
        this.price = price;
    }
}
```
``` java
 public class Bing extends Food {

    @Override
    public int cost() {
        return super.getPrice();
    }
}
```
``` java
public class JianBingGuoZi extends Bing {
    public JianBingGuoZi(){
        super.setDescription("煎饼果子");
        super.setPrice(6);
    }
}
```
``` java 
public class JiDanGuanBing extends Bing{
    public JiDanGuanBing(){
        super.setDescription("鸡蛋灌饼");
        super.setPrice(7);
    }
}
```
``` java
public class ShouZhuoBing extends Bing {
    public ShouZhuoBing(){
        super.setDescription("手抓饼");
        super.setPrice(5);
    }
}
```
从上面可以看出首先定义一个抽象的food父类，用于设置各种饼的名称及价格，然后定义出一个中间类Bing类继承food类，重写了cost()方法，用于返回单品（不添加任何配料）的价格，各个饼种需要继承Bing类在构造函数中设置的单品的名称及价格，接下来看下装饰者的应用：   
``` Java
public class Decorator extends Food {

    Food food;

    public Decorator(Food food) {
        this.food = food;
    }

    @Override
    public int cost() {
        return super.getPrice()+this.food.cost();
    }

    @Override
    public String getDescription() {
        return super.getDescription()+" \n "+this.food.getDescription();
    }
}
``` 
可以看到装饰者代码继承自food类，构造函数中将其带入作为参数，重写cost与getDescription方法，使用迭代的形式将每次添加的配料累加，形成一个循环，将其返回：
``` Java
public class Egg extends Decorator {
    public Egg(Food food) {
        super(food);
        super.setDescription("鸡蛋");
        super.setPrice(1);
    }
}
```
```Java
public class Lettuce extends Decorator {
    public Lettuce(Food food) {
        super(food);
        super.setDescription("生菜");
        super.setPrice(1);
    }
}
```
```Java
public class Sausage extends Decorator {
    public Sausage(Food food) {
        super(food);
        super.setDescription("香肠");
        super.setPrice(2);
    }
}
```
各个配料类需要继承自Decorator装饰这类，然后在构造函数中带入food类作为参数，传入父类，并需要设置其价格及其描述

现在来看下测试效果
```Java
public class BreakfastTest {
    public static void main(String[] args) {
        JianBingGuoZi jianBingGuoZi = new JianBingGuoZi();
        System.out.println("order1 desc: "+jianBingGuoZi.getDescription());
        System.out.println("order1 price "+jianBingGuoZi.cost());

        System.out.println("``````````````````````````````");
        Food food = new ShouZhuoBing();
        food = new Egg(food);
        food = new Sausage(food);
        System.out.println("order2 desc: "+food.getDescription());
        System.out.println("order2 price "+food.cost());
    }
}
```
![](/assets/img/2018-10/16/test.png)

<h3> 从上面的代码中我们可以看出使用装饰者后，我们无论需要什么配料都可以累加，假如有一天有了大饼鸡蛋或者配料添加了肉松，只需要按照上面类似加入新的类就可以，不用修改父类或者大量改动已有的代码。




