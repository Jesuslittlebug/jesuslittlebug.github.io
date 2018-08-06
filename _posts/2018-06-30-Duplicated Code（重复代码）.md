---
layout: post
title: 重复代码
key: szk
tags: 重构代码
mathjax: true
mathjax_autoNumber: true
---
## Duplicated Code（重复代码）
重复代码会有以下三种情况。

- 同一个类的两个函数含有相同的表达式。
- 两个互为兄弟的子类内含有相同表达式。
- 两个毫不相干的类出现重复代码。
按照重复代码出现的位置不同，有不同的处理方式。

### 1.同一类的两个函数中
使用Extract Method提炼重复代码，然后让这两个函数调用提炼出来的独立函数。
### 2.互为兄弟的子类中
- 函数相同，对两个类都使用Extract Method，再对提炼的代码使用Pull Up Method，将其推入超类中。

- 若代码类似但不相同，使用Extract Method 将相似与差异部分分割，构成单独的函数，然后运用Form Template Method 获得一个Template Method（模版方法）设计模式。若有些函数用不同的算法做相同的事情，可选择其中清晰的一个，使用Substitute Algorithm替换其他函数的算法。

### 3.两个毫不相干的类
只对其中一个类使用Extract Method，将重复代码提取到一个独立类中，然后在另一个类中使用新类。
<!--more-->
## Extract Method（提炼函数）
在遇到重复代码、过长函数或需注释才能让人理解的代码时，可以提炼独立函数。

### 示例
分为三种情况：

- 一种是无局部变量
- 一种是有局部变量无赋值操作
- 一种是有局部变量有赋值操作

提炼前的代码：

```java
  void printOwing(double previousAmount){
        Enumeration enumeration = orders.elements();
        
        double outstanding = previousAmount * 1.2;

        //无局部变量的情况
        System.out.println("***************");
        System.out.println("******Test*****");
        System.out.println("***************");

		 //有局部变量且有赋值操作
        while (enumeration.hasMoreElements()){
            Order each = (Order) enumeration.nextElement();
            outstanding += each.getAmount();
        }

        //有局部变量
        System.out.println("name" + _name);
        System.out.println("amount" + outstanding);
    }
```
提炼后的代码：

```
  void printOwing(double previousAmount){
        
        printBanner();

        double outstanding = previousAmount * 1.2;

        outstanding = getOutstanding(outstanding);
        printDetails(outstanding);
        
    }

    //对局部变量赋值，需返回局部变量
    private double getOutstanding(double initialValue) {
        //不建议给参数赋值，建议用临时变量给参数赋值
        double result = outstanding;
        Enumeration enumeration = orders.elements();
        while (enumeration.hasMoreElements()){
            Order each = (Order) enumeration.nextElement();
            result += each.getAmount();
        }
        return result;
    }

    //有局部变量，但无赋值
    private void printDetails(double outstanding) {
        
        System.out.println("name" + _name);
        System.out.println("amount" + outstanding);
    }

    //无局部变量的情况
    private void printBanner() {
        System.out.println("***************");
        System.out.println("******Test*****");
        System.out.println("***************");
    }
```

## Pull Up Method（函数上移）
这种方式**适用于兄弟子类有相同函数的情况**，即继承与同一父类的子类中的函数体相同。

比如，有开发人员Developer和产品经理ProductManager两个类，继承于父类Employee。Developer与ProductManager类中均有getName方法，则可以考虑将这个方法上移到父类Employee中。

在函数上移过程中，最麻烦的情况在于，被提升的函数中引用了只出现于子类而不出现于超类的特性，若被引用的是函数，可以将该函数一起上移或者在超类中建立一个抽象函数，但是这样的话，**超类也要声明为抽象类。**


## Form Template Method（塑造模版函数）
在理解模版函数之前，需要理解设计模式中的模版模式。**模版模式就是在父类中定义处理流程的框架，在子类中实现具体处理的模式**。

示例-模版模式：

假设员工总工资由基础工资与绩效工资组成，不同职位有着不同的计算方式。

```java
//父类
abstract class Employee{
	//基础工资
	abstract double getBaseSalary();
	//绩效工资
	abstract double getPerformanceSalary();
	//总工资，这里设置为final类型，不可覆写
	public final double getTotalSalary(){
		return getBaseSalary() + getPerformanceSalary();
}
		
```
```java
//子类-开发人员
class Developer extends Employee{
	public double getBaseSalary(){
		//具体计算方法
		return baseSalary;
	}
	public double getPerformanceSalary(){
		//具体计算方法
		return performanceSalary;
	}
}
//子类-产品人员
class ProductManager extends Employee{
	public double getBaseSalary(){
		//具体计算方法
		return baseSalary;
	}
	public double getPerformanceSalary(){
		//具体计算方法
		return performanceSalary;
	}
}
```

像以上这种方式就是模版模式。塑造模版函数就是为了解决不同子类有着类似的函数，但是实现方式不同，从而解决兄弟子类有着重复代码的情况。

## Substitute Algorithm（替换算法）
所谓替换算法就是从实现方式但是结果一样的算法中选取一种最简单的方式或最清晰的算法。

在使用这种方式之前，应先确定自己已经尽可能分解了原先函数。替换一个巨大而复杂的函数非常困难，只有先将其分割成简单的小型函数，然后才能有把握的进行替换工作。

针对每个测试用例，要以新旧算法分别进行执行，观察两者结果是否相同。

