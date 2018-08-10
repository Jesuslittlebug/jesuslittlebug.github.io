---
layout: post
title: 模版方法（Template Method）
key: szk
tags: 设计模式
---

## 定义
什么是模版方法呢？模版方法就是指在父类中定义处理流程的框架，在子类中实现具体的处理的模式。

## 示例
假设员工总工资由基础工资与绩效工资组成，不同职位有着不同的计算方式。
像如下代码一样，在父类Employee中定义了流程及抽象方法，在两个子类中给予具体的实现。

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