---
author: "Phong Nguyen"
title: "Design Patterns"
date: "2025-02-11"
description: "Design Patterns."
tags: ["java"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 8    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---
Explain how to use the design patterns. (Creational Patterns)
[Refer1](https://github.com/kamranahmedse/design-patterns-for-humans?tab=readme-ov-file#-simple-factory)
[Refer2](https://java-design-patterns.com/)
[Refer3](https://www.tutorialspoint.com/design_pattern/factory_pattern.htm)
[Refer4](https://www.oodesign.com/factory-pattern)<br>

## 1. Creational Design Patterns. 
- Creational design patterns provide various **object creation mechanisms**, which increase flexibility and reuse of existing code.
### 1.1 Simple Factory
- Factory is an object for creating other objects 
- **Use cases:**
  - when the class does not know beforehand the exact types and dependencies of the objects it needs to create.
  - When a method returns one of several possible classes that share a common super class and wants to encapsulate the logic of which object to create.
  - when designing frameworks or libraries to give the best flexibility and isolation from concrete class types

- **Examples:** You want to manufacture the products many times.
  -  Design UML:
![image](/images/factory_pattern.png)
 
  - Implementation codes:

```java
package design.patterns.factory;

public interface IProduct {
	public void getType();
}

package design.patterns.factory;

public class ProductA implements IProduct {
	private static String TYPE_NAME = "ProductA";
	@Override
	public void getType() {
		System.err.println(TYPE_NAME);
	}

}

package design.patterns.factory;

public class ProductB implements IProduct {
	private static String TYPE_NAME = "ProductB";
	@Override
	public void getType() {
		System.err.println(TYPE_NAME);
	}
}

package design.patterns.factory;

public class ProductC implements IProduct {
	private static String TYPE_NAME = "ProductC";
	@Override
	public void getType() {
		System.err.println(TYPE_NAME);
	}

}

package design.patterns.factory;

public enum ProductType {
	TYPEA,
	TYPEB,
	TYPEC;
}

package design.patterns.factory;

public class ProductFactory {
	public static IProduct createProduct(ProductType type) {
		if (type == null) {
			return null;
		}

		switch (type) {
		case TYPEA: {
			return new ProductA();
		}
		case TYPEB: {
			return new ProductB();
		}
		case TYPEC: {
			return new ProductC();
		}
		}
		return null;
	}
}

package design.patterns.factory;

public class MainTestFactory {

	public static void main(String[] args) {
		ProductFactory productFactory = new ProductFactory();
		
		IProduct product1 = productFactory.createProduct(ProductType.TYPEA);
		product1.getType();
		
		IProduct product2 = productFactory.createProduct(ProductType.TYPEB);
		product2.getType();
		
		IProduct product3 = productFactory.createProduct(ProductType.TYPEC);
		product3.getType();
				
	}

}


```
<br>

### 1.2. Factory Method
- **Factory Method** is a creational design pattern that provides an interface for creating objects in a superclass, but allows subclasses to alter the type of objects that will be created.
- **Use cases:**
  - when class cannot anticipate the class of objects it must create.
  - When class **wants its subclasses** to **specify the objects it creates**.
  - when designing frameworks or libraries to give the best flexibility and isolation from concrete class types

- **Examples:** You want to manufacture the products. You must be able to create both A and B (C,D) products and add/switch between them without modify the existing source codes.
  -  Design UML:
![image](/images/factory_method_pattern.png)
 
  - Implementation codes:

```java
package design.patterns.factory.method;

public interface IProduct {
	public String getType();
}

package design.patterns.factory.method;

public class ProductA implements IProduct {
	private static String TYPE_NAME = "ProductA";
	@Override
	public String getType() {
		return TYPE_NAME;
	}

}
package design.patterns.factory.method;

public class ProductB implements IProduct {
	private static String TYPE_NAME = "ProductB";
	@Override
	public String getType() {
		return TYPE_NAME;
	}
}

package design.patterns.factory.method;

public class ProductC implements IProduct {
	private static String TYPE_NAME = "ProductC";
	@Override
	public String getType() {
		return TYPE_NAME;
	}

}

package design.patterns.factory.method;

public abstract class AbstractFactory {
	public abstract IProduct createProduct();
	
	public void pickProduct() {
		IProduct product = createProduct();
		System.out.println("Do something with" + product.getType());
	}
}

package design.patterns.factory.method;

public class FactoryA extends AbstractFactory {

	@Override
	public IProduct createProduct() {
		// TODO Auto-generated method stub
		return new ProductA();
	}

}

package design.patterns.factory.method;

public class FactoryB extends AbstractFactory {

	@Override
	public IProduct createProduct() {
		// TODO Auto-generated method stub
		return new ProductB();
	}
}

package design.patterns.factory.method;

public class FactoryC extends AbstractFactory {

	@Override
	public IProduct createProduct() {
		// TODO Auto-generated method stub
		return new ProductC();
	}
}

package design.patterns.factory.method;

public class MainTestFactory {

	public static void main(String[] args) {
		AbstractFactory factoryA = new FactoryA();
		IProduct productA = factoryA.createProduct();
		factoryA.pickProduct();

		AbstractFactory factoryA1 = new FactoryB();
		IProduct productA1 = factoryA1.createProduct();
		factoryA1.pickProduct();
	}

}

```

<br>

### 1.3. Abstract Factory
- **Abstract Factory** is a creational design pattern that lets you produce families of related objects without specifying their concrete classes.
- **Use cases:**
  - when need a way to **create individual furniture objects** so that they **match other objects of the same family**. 
  - When don’t want to change existing code when adding new products or families of products to the program. 

- **Examples:** We intend to manufacture a product line that included a family of related products: A,B,and C - along with several variants such as AWin,ALinux, BWin,BLinux, CWin, and CLinux. 
  -  Design UML:
![image](/images/factory_abstract_pattern.png)
 
  - Implementation codes:

```java
package design.patterns.factoryabstract;

public interface IProductA {
	public String getType();
}

package design.patterns.factoryabstract;

public interface IProductB {
	public String getType();
}

package design.patterns.factoryabstract;

public class ProductALinux implements IProductA {

	private static final String TYPE = "ProductALinux";

	@Override
	public String getType() {
		return TYPE;
	}

}

package design.patterns.factoryabstract;

public class ProductAWin implements IProductA {

	private static final String TYPE = "ProductAWin";

	@Override
	public String getType() {
		return TYPE;
	}
}

package design.patterns.factoryabstract;

public class ProductBLinux implements IProductB {

	private static final String TYPE = "ProductBLinux";

	@Override
	public String getType() {
		return TYPE;
	}
}

package design.patterns.factoryabstract;

public class ProductBWin implements IProductB {

	private static final String TYPE = "ProductBWin";

	@Override
	public String getType() {
		return TYPE;
	}
}

package design.patterns.factoryabstract;

public interface IFactoryProduct {
	public IProductA createProductA();

	public IProductB createProductB();
}

package design.patterns.factoryabstract;

public class FactoryProductLinux implements IFactoryProduct {

	@Override
	public IProductA createProductA() {
		return new ProductALinux();
	}

	@Override
	public IProductB createProductB() {
		return new ProductBLinux();
	}

}

package design.patterns.factoryabstract;

public class FactoryProductWin implements IFactoryProduct {

	@Override
	public IProductA createProductA() {
		return new ProductAWin();
	}

	@Override
	public IProductB createProductB() {
		return new ProductBWin();
	}

}

package design.patterns.factoryabstract;

public class UserApplication {
	private IFactoryProduct factory;
	private IProductA productA;
	private IProductB productB;

	public UserApplication(IFactoryProduct factory) {
		this.factory = factory;
	}

	public void createProducts() {
		productA = factory.createProductA();
		productB = factory.createProductB();
	}

	public void someOperations() {
		System.err.println(
				"This is the User Application created with " + productA.getType() + " and " + productB.getType());
	}
}

package design.patterns.factoryabstract;

public class Main {

	public static void main(String[] args) {
		IFactoryProduct factory = new FactoryProductWin();
//		IFactoryProduct factory = new FactoryProductLinux();

		UserApplication userApp = new UserApplication(factory);
		userApp.createProducts();
		userApp.someOperations();

	}
}

```
<br>

### 1.4. Builder
- **Builder** is a creational design pattern that lets you construct complex objects step by step. The pattern allows you to produce different types and representations of an object using the same construction code.
- **Use cases:**
  -  build objects step by step, using only those steps that you really need. 

- **Examples:** We need to create some products: A,B and C with multiple optional attributes.
  -  Design UML:
![image](/images/.png)
 
  - Implementation codes:
`https://www.baeldung.com/java-builder-pattern`

## 2. Structural Design Patterns. 
- Structural design patterns explain how to **assemble objects and classes into larger structures**, while keeping these structures flexible and efficient. (ava.util.Arrays#asList())
### 2.1. Adapter
- The adapter pattern is adapting between classes and objects.
- **Use cases:**
  - Convert the interface of a class into another interface clients expect
  - Make incompatible objects work together.

- **UML:**
![image](/images/adapter_pattern.png)

