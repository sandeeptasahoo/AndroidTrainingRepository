#### course link : https://samsungu.udemy.com/course/kotlindesignpatterns/learn/lecture/27528290?start=1#overview
#### course book : gang of four
#### kotlin ide : Intellij IDEA

## Introduction
there are 3 types of patterns
### 1. Creational pattern : 
this handles how to instantiate a class, and how to handle variables while instantiating
eg: 
1. Singleton
2. Factory
3. Builder
4. lazy initialization
5. prototype

### 2. Structural pattern :
this makes the structures are testable and readable
1. bridge
2. Adapter
3. facade
4. dcorator
5. composite
6. proxy

### 3. Behavioural pattern : 
this makes the structure readable and the interdependency between methods and classes are handled 
1. observer
2. chain of responsibility
3. command
4. strategy
5. state
6. visitor
7. mediator
8. memento

There is a 4th type of pattern named **Concurrency design pattern** 

## Singleton
creating a single instance of a class to access resources is very useful 
Java way to create a singleton 
https://www.geeksforgeeks.org/singleton-class-java/

kotlin: create an object class 

## Factory pattern
### factory method
this gives a pattern that says the main class, which is accessing a class inherited by an interface, won't know how many similar classes are there having the same interface.
The main class will access the factory to access the class the class will provide the targeted class according to requirement. 
some parameters will be passed to the factory to identify which class the main class requires.
 
java: https://www.geeksforgeeks.org/factory-method-design-pattern-in-java/

### factory abstract
An abstract factory is a capping-over factory method where an abstract factory will provide a factory which will provide the actual class 
the caller of abstract factory won't know what class is coming having the same interface 

java: https://www.geeksforgeeks.org/abstract-factory-pattern/

## Builder pattern 
when a class needs to be created with some optional variable like polymorphism in the method. 
The builder pattern is very handy in Java where you can construct the class with any combination of parameter. 
Kotlin handles this by naming parameters where we can assign specific parameters.

java: https://www.geeksforgeeks.org/builder-design-pattern/ 

## Lazy Initialisation
when large memory objects are created which can take more ram space while initialising, can use this pattern. Here in this pattern, the object will be initialized by the processor when the object is being called for, even when we have declared the object initially while creating the class itself.

Kotlin handles this with lazy keyword:
through lazy only the val type of variable is created 
to create var we can use lateinit where we can declare the object in the later part of initiation to the variable 

## Prototype pattern
here main class is accessing a class through an interface and the class is given by giving a clone of the target class.
java link: https://www.geeksforgeeks.org/prototype-design-pattern/

this pattern feels similar to a factory pattern

## Bridge pattern 
when class objects need to be created and they have some properties similar to each other then making an interface of them and separating them will increase readability. But when their property increases way too much and multiple interfaces are needed to inherit it's better to create an object with some inheritance and use that object in another inheritance making the targeted property complete.

eg: I wanted to create a TV remote, AC remote, and car remote
here the properties are device and control so I will make the interface
but if I create the direct object by inheriting two interfaces it will be clumsy 

so I see control property works as control over a device so I can pass a parameter to the control interface to define it. that will make a bridge between two interfaces and will create targeted objects 

## Adapter pattern 
when the conversion of one data class to another data class is needed, the adapter is used to convert and typecast. 

## facade pattern
It's a method to segregate complexity from the caller. That means the main class won't need to think about handling exceptions or any edge case the target class handles all and provides the info in an easy interface with less complexity.

eg: API access of cloud and application layer
wrapper class in Android

## Decorator pattern 
It is basically linear inheritance. This solves adding more functionality to older functions by overriding or adding extra behavior to it.

## Composite pattern 
this pattern says to segregate classes into small interfaces that are mutually inclusive then create a **tree** pattern to by  one interface inheriting others and finally create a target class 

## Proxy pattern 
when accessing certain data takes time or certain extra functionality or layering is required to access the actual data proxy class is implemented between the target and accessor. This extra proxy class handles all the extra complexity. 
it is similar to the facade pattern but handles more complexity.
It is also similar to Decorator but has more extra functionality.

## Observer pattern
this is the most used pattern. This is basically an event generator and event subscriber mechanism when the subscriber is notified that the event has been generated.

link : https://www.geeksforgeeks.org/observer-pattern-set-1-introduction/

## Chain of responsibility
here a request is made and that request can go through multiple layers of handlers. Initially, a request is being processed by a handler if it is unable to process, it moves the request to another handler or returns the result.

this layer can take the shape of a chain or tree.

## Command design pattern
it talks about an adapter class which can take a request and fed the request to proper handler to process 

## strategy design pattern
when a class can be defined differently at run type. This can be done when the class has a dynamic parameter through which the behavior of the class can be altered.

## State design pattern
when a state change mechanism is required where one state can change to some certain state and this state needs to be encapsulated.

## Visitor design pattern
when an interface is a device that knows how to consume the given parameter by encapsulation to the caller.
https://www.geeksforgeeks.org/visitor-design-pattern/

## Mediator design pattern
when many classes are calling each other and its creating complex behavior, a mediator will be introduced which will connect the call within the classes but all classes will just call the mediator 

## Memento design pattern
This pattern is needed when we require to restore a data class from its deleted condition 
It has 3 parts : 
memento: stores the state
Originator: creates the state
CareTaker: Decides to save or restore the state

