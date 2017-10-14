---
title: Java中enum类型的使用
date: 2017-10-14 20:00:26
tags: enum
categories: java
---
### enum简介
java中的enum类型，大家可能都比较了解，可是使用频率并不高，下面直接给出enum类型的写法
```
public enum DataSourceEnum{
	WebCache("WebCache"),
    SearchHub("SearchHub"),
	QueryRewriteServer("QueryRewriteServer")
	;

	private String name;
	DataSourceEnum(String n){
		this.name = n;
	}

	public String getName(){
		return name;
	}

	@Override
	public String toString(){
		return getName();
	}
}
```

枚举类型和常量的定义相似，定义了一个独一无二的类型描述符，但是常量有一致性差、类型不安全等缺点，还会造成魔数问题，如果没有注释活着不是开发者很难理解含义

上面的代码，使用时候传入一个枚举的类型，在调用getName()方法获得枚举变量的值

Enum类型提出给JAVA编程带了了极大的便利，让程序的控制更加的容易，也不容易出现错误。心里没有概念之前很难想到如何使用enum，以后在遇到需要控制程序流程时候，多想想是否可以利用enum来实现。

### 参考资料
[Java语言中Enum类型的使用介绍](https://www.ibm.com/developerworks/cn/java/j-lo-enum/index.html)

[java枚举类型enum的使用](http://blog.csdn.net/wgw335363240/article/details/6359614)




