---
layout: post
title: 单例模式(Singleton)
key: szk
tags: 设计模式
---
在程序运行时，会生成很多的实例，那么当需要在系统中只生成一个实例时，如表示系统相关设置的类。这时就需要用到单例模式，只需要保证在整个系统中只运行了一次new class()的操作即可。
**注意：在代码中，必须要保证构造方法是私有的，从而保证无法在类的外部去实例化本类，因此实现单例。我们现在探讨的单例模式不考虑序列化、克隆以及反射带来的问题。**

## 饿汉式-线程安全
实例在类加载的时候就进行了初始化，无论使用还是不使用，一直占用内存直至类被卸载。适用于单例占用内存小且初始化时间短的情况下。
```
public class Singleton {
	private final static Singleton INSTANCE = new Singleton();
	private Singleton(){
	}
	
	public static Singleton getSingletonInstance() {
		return INSTANCE;
	}
	
}

```
## 懒汉式
### 实现一：线程非安全
在单线程中是没问题的，但多线程中就会重复的实例化。
```
public class Singleton {

	private static Singleton INSTANCE;

	private Singleton() {
	}

	public static Singleton getSingletonInstance() {
		if (INSTANCE == null) {
			INSTANCE = new Singleton();
		}
		return INSTANCE;
	}

}
```
### 实现二： 线程安全
相较于实现一是线程安全的，但是synchronized同步的是整个方法，这会使得高并发的情况下，效率变低。
```
public class Singleton {

	private static Singleton INSTANCE;

	private Singleton() {
	}

	public static synchronized Singleton getSingletonInstance() {
		if (INSTANCE == null) {
			INSTANCE = new Singleton();
		}
		return INSTANCE;
	}

}
```
## 双重校验锁实现（Double-Check-Lock）-线程安全
相较于实现二，这种方式效率更高。使用与jdk1.5及以上的版本。
那么为何需要加上volatile呢？是由于volatile关键字会**禁止指令重排序以及保证对象内存可见性**。
**INSTANCE = new Singleton();**这样一个实例化对象的操作会经历三个过程：
1. 分配内存空间
2. 初始化对象
3. 将内存空间的地址分配给INSTANCE

但是由于操作系统会对指令进行重排序，所有上述的过程也有可能是1,3,2。
现在假设有两个线程A，B。在没有关键字volatile时，A线程执行到实例化对象的语句，按照1,3,2的顺序，此时完成了3的操作。然后B线程在调用到INSTANCE的地方判断INSTANCE不为空并访问其引用的对象，但此时A线程中2的操作还未完成，B线程就会访问到一个未初始化的对象。
如下图：T4时刻，线程B会出错。

| 时间 | 线程A | 线程B |
| :---: | :---: | :---: |
| T1 |1-分配内存空间|  -  |
| T2 |3-将地址分给INSTANCE| - | 
| T3 | -| 判断INSTANCE不为空| 
| T4 | -| 由于INSTANCE不为空，访问其引用的对象|  
| T5 |2-初始化对象|-|  
| T6 |访问INSTANCE引用的对象|-|   


```
public class Singleton {

	//volatile保证了多线程间变量的可见性
	private static volatile Singleton INSTANCE;

	private Singleton() {
	}

	public static  Singleton getSingletonInstance() {
		//双重校验锁，保证了线程安全性
		if (INSTANCE == null) {
			synchronized (Singleton.class) {
				if(INSTANCE == null) {
					INSTANCE = new Singleton();
				}
			} 
		}
		return INSTANCE;
	}

}
```
## 静态内部类-线程安全
将对象的实例化放到静态内部类中，这样由于静态内部类只会被加载一次，因此这种方法也是线程安全的。
```
public class Singleton {
	private Singleton(){
		
	}

	private static class SingletonHolder {
		public static Singleton INSTANCE = new Singleton();
	}
	
	public static Singleton getSingletonInstance() {
		return SingletonHolder.INSTANCE;
	}
}
```
## 枚举方式
枚举方式能够抵御反射、克隆以及序列化对单例的破坏。其他的实现方式需要添加额外的判断代码。
```
public enum Singleton {

	INSTANCE;
	public Singleton getInstance() {
		return INSTANCE;
	}
}

```
