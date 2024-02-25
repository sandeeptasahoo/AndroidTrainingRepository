#### course link : https://samsungu.udemy.com/course/kotlindesignpatterns/learn/lecture/27528290?start=1#overview
#### course book : gang of four
#### kotlin ide : Intellij IDEA

## Introduction
there are 3 types of patterns
### 1. Creational pattern : 
this handles how to instantiate a class, and how to handle variables while instantiating

### 2. Structural pattern :
this makes the structures are testable and readable

### 3. Behavioural pattern : 
this makes the structure readable and the interdependency between methods and classes are handled 

There is a 4th type of pattern named **Concurrency design pattern** 

## Singleton
creating a single instance of a class to access resources is very useful 
java way to create a singleton 
https://www.geeksforgeeks.org/singleton-class-java/

kotlin : create a object class 

## Factory pattern
### factory method
this gives a pattern that says the main class, who is accessing a class inherited by an interface, wont know how many similar class are there having same interface.
The main class will access factory to access the class the class will provide the targeted class according to requirement. 
some parameter will be passed to factory to identify which class the main class requires.
 
java : https://www.geeksforgeeks.org/factory-method-design-pattern-in-java/

### factory abstract
An abstract factory is a capping-over factory method where an abstract factory will provide a factory which will provide the actual class 
the caller of abstract factory won't know that what class is coming having same interface 

java : https://www.geeksforgeeks.org/abstract-factory-pattern/

## Builder pattern 
when a class is needed to be created with some optional variable like polymorphism in method. 
Builder pattern is very handy in java where you can construct the class with any combination of parameter. 
Kotlin handles this by named parameter where we can assign specific parameter.

java : https://www.geeksforgeeks.org/builder-design-pattern/

## Lazy Initialisation
when large memory objects are created which can take more ram space while initialising, can use this pattern. Here in this pattern, the object will be initialized by the processor when the object is being called for, even when we have declared the object initially while creating the class itself.

Kotlin handles this with lazy keyword:
through lazy only val type of variable is created 
to create var we can use lateinit where we can declare the object in the later part of initiation to variable 

## Prototype pattern
here main class is accessing an class through an interface and the class is given by giving a clone of the target class.
java link : https://www.geeksforgeeks.org/prototype-design-pattern/

this pattern feels similar to factory pattern

## Adapter pattern 
