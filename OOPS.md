# Comparison of Different Languages

## C

- High performance and more control
- Statically typed (Means the type checking of the variables/ data happens at compile time)

## Java

- Object Oriented, used for high security, maintainability, code reusability,
- Statically typed

## Python

- Motive is to make the development faster by reducing the programming syntax complexity
- Dynamically typed

> **Imperative** : step by step instruction you write to run a program


---

# Stack and Heap Memory

## Stack Memory

Stack memory is used primarily for function calls and their local execution state.

- Stores function call frames, local primitive values in some languages, and return information.
- Follows LIFO behavior (Last In, First Out).
- Allocation and deallocation are generally very fast.
- Each thread typically has its own call stack.

![Stack vs Heap Memory in C# and Unity: A Complete Guide | by JUMAMUROD ARALOV | Medium](https://images.openai.com/static-rsc-4/9sog8-B19RpOLcu1a48qRxeMLZus_xN9H-SDbzJRzeHBzH0pyFC0u4aLQcM_YiAYFnyQd1Hq6-6-jyMgZaA_kHdHHhxkAL3QX8Wv7bt40eKb_P_Q-7567BlEJywScUBdzpLW_4iq_4s13UWrPyoT_P3OQpd6w0lsboluZA39R6s?purpose=inline)

## Heap Memory

Heap memory is generally used for dynamically allocated objects and data structures.

- Stores objects whose lifetimes can extend beyond a single function call.
- Objects can be accessed through references or pointers.
- Allocation and cleanup are more complex.
- Multiple threads can access shared heap objects, subject to synchronization and language rules.


---

# Encapsulation

## How Does Encapsulation Work?

- It wraps the data and methods into one single unit called class
- Hides the implementation details , shows only the necessary information
- Prevent un authorized access , like objects of another class trying to modify the resource in current class
- uses private key word, which can only be changed using getter and setter

Constructors are used to initialize objects when they got created


---

# Java Array

- It cannot store different type of data directly, all the element inside the array must be of same type
- but allows to store objects of different types
- Arrays in java are fixed length, once you declare , you cannot change its size
  so you cannot add or delete element,
- can access or modify the element using index

```java
Object[] arr = {10, "Hello", 3.14, true};

/*To acces the actual data in this object, you have to do */*

x + 5; // ❌ /* this does not work*/

Integer n = (Integer) x; System.out.println(n + 5); // 15
```


---

# Inheritance

## 1. Single Inheritance

**One parent → one child**

```java
class Animal {
    void eat() {
        System.out.println("Eating");
    }
}

class Dog extends Animal {
    void bark() {
        System.out.println("Barking");
    }
}
```

```java
Dog d = new Dog();

d.eat();   // inherited from Animal
d.bark();  // Dog's own method
```

### Structure

```
Animal
   ↓
  Dog
```

### Definition

> **Single inheritance occurs when one child class inherits from one parent class.**

## 2. Multilevel Inheritance

**Parent → Child → Grandchild**

```java
class Animal {
    void eat() {
        System.out.println("Eating");
    }
}

class Dog extends Animal {
    void bark() {
        System.out.println("Barking");
    }
}

class Puppy extends Dog {
    void play() {
        System.out.println("Playing");
    }
}
```

Now:

```java
Puppy p = new Puppy();

p.eat();   // Animal
p.bark();  // Dog
p.play();  // Puppy
```

### Structure

```
Animal
   ↓
  Dog
   ↓
 Puppy
```

### Definition

> **Multilevel inheritance occurs when a class inherits from another derived class, creating a chain of inheritance.**

## 3. Hierarchical Inheritance

**One parent → multiple children**

```java
class Animal {
    void eat() {
        System.out.println("Eating");
    }
}

class Dog extends Animal {
    void bark() {
        System.out.println("Barking");
    }
}

class Cat extends Animal {
    void meow() {
        System.out.println("Meowing");
    }
}
```

### Structure

```
        Animal
        /    \
       ↓      ↓
     Dog      Cat
```

Both `Dog` and `Cat` inherit `eat()` from `Animal`.

```java
Dog d = new Dog();
d.eat();

Cat c = new Cat();
c.eat();
```

### Definition

> **Hierarchical inheritance occurs when multiple child classes inherit from the same parent class.**

## 4. Multiple Inheritance

It Creates ambiguty

**Multiple parents → one child**

Conceptually:

```
Animal       Bird
   \          /
    \        /
      FlyingAnimal
```

For example:

```java
class Animal {
    void eat() {
        System.out.println("Eating");
    }
}

class Bird {
    void fly() {
        System.out.println("Flying");
    }
}
```

You might want:

```java
class FlyingAnimal extends Animal, Bird {
}
```

But Java **does not allow this with classes**.

```java
// ❌ Not allowed in Java
class FlyingAnimal extends Animal, Bird {
}
```

### Why?

Because it can create ambiguity — the famous **diamond problem**.

Java solves this by allowing multiple inheritance through **interfaces**.

```java
interface Animal {
    void eat();
}

interface Bird {
    void fly();
}

class FlyingAnimal implements Animal, Bird {

    public void eat() {
        System.out.println("Eating");
    }

    public void fly() {
        System.out.println("Flying");
    }
}
```

Now:

```java
FlyingAnimal f = new FlyingAnimal();

f.eat();
f.fly();
```

### Definition

> **Multiple inheritance means a class inherits from more than one parent. Java does not support multiple inheritance through classes, but it can be achieved through multiple interfaces.**

## 5. Hybrid Inheritance

Hybrid inheritance is a **combination of two or more types of inheritance**.

For example, combine **hierarchical + multiple inheritance**.

```
          Animal
         /      \
        Dog      Cat
          \      /
           \    /
          PetAnimal
```

However, Java cannot create this exact structure using classes alone because of multiple inheritance.

Using interfaces:

```java
class Animal {
    void eat() {
        System.out.println("Eating");
    }
}

interface Pet {
    void play();
}

class Dog extends Animal implements Pet {

    public void play() {
        System.out.println("Dog playing");
    }
}

class Cat extends Animal implements Pet {

    public void play() {
        System.out.println("Cat playing");
    }
}
```

Here we combine inheritance through a class hierarchy and interface implementation.

### Definition

> **Hybrid inheritance is a combination of two or more types of inheritance. Java can achieve hybrid inheritance using classes and interfaces, but not through multiple class inheritance.**

## Diamond Problem

### Why is it called the Diamond Problem?

Consider:

```
       A
      / \
     B   C
      \ /
       D
```

`A` contains:

```java
void show()
```

Both `B` and `C` inherit it, and `D` inherits from both.

So `D` potentially gets **two paths to the same method**:

```
D → B → A
D → C → A
```

That's why it's called the **diamond problem**.

- `From java 8,, interfaces can conntain abstract as well as default methods but with body`
- `JDK 27` is the latest

## Difference Between Abstract Class and Interfaces

| ***Abstract class***                                                                                                                                                                   | ***Interfaces***                                                                                    |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| uses abstract keyword                                                                                                                                                                  | uses implement keyword                                                                              |
| it can contain constructors                                                                                                                                                            | it canot contain constructor                                                                        |
| it can contain any type of acces modifiers<br>and method declarations except `default `<br>because default is interface method introduced in java 8<br>to provide common functionality | <br>methods can only be `public, default and private`                                               |
| canot used to achieve multiple inheritence                                                                                                                                             | it is used to achieve multiple inheritence                                                          |
| can use normal instance varibles. and they can be modified                                                                                                                             | variables are by default `public static final`. so the variables canot be<br>modified once declared |

## Disadvantages of Inheritance

**Q. What are the disadvantages of inheritance?**

A. Tight coupling between parent and child; a change in the parent can break all children (the **fragile base class problem**); deep hierarchies become hard to understand. This is why designers prefer **composition over inheritance** (see Rank 7).


---

# Polymorphism

> **Definition:** Polymorphism ("many forms") is the ability of a single interface, method name or reference to behave differently depending on the object or the arguments it works with. In Java it comes in two kinds: **compile-time** (method overloading) and **runtime** (method overriding).

## Simple Explanation

One person can be a son at home, an employee at office and a customer at a shop – same person, different behaviour depending on context. Similarly, a `draw()` call draws a circle, square or triangle depending on the actual object.

## Compile-time Polymorphism – Method Overloading

Same method name, **different parameter list** (number, type or order), in the same class. The compiler decides which method to call. Return type alone is NOT enough to overload.

```java
class Calculator {
    int add(int a, int b)            { return a + b; }
    int add(int a, int b, int c)     { return a + b + c; }
    double add(double a, double b)   { return a + b; }
}
```

## Runtime Polymorphism – Method Overriding

A subclass provides its own implementation of a method already defined in its parent, with the **same signature**. The JVM decides at runtime which version to call, based on the **actual object**, not the reference type. This is called **dynamic method dispatch**.

```java
class Notification {
    void send(String msg) { System.out.println("Generic: " + msg); }
}
class EmailNotification extends Notification {
    @Override void send(String msg) { System.out.println("Email: " + msg); }
}
class SmsNotification extends Notification {
    @Override void send(String msg) { System.out.println("SMS: " + msg); }
}

Notification n = new SmsNotification();   // upcasting
n.send("Order shipped");                   // prints "SMS: Order shipped"

List<Notification> all = List.of(new EmailNotification(), new SmsNotification());
for (Notification x : all) x.send("Hi");   // each behaves differently
```

## Overloading vs Overriding

|Point|Overloading|Overriding|
|---|---|---|
|Where|Same class|Parent and child class|
|Parameters|Must differ|Must be the same|
|Return type|Can be anything|Same or a subtype (covariant)|
|Binding|Compile time (static)|Runtime (dynamic)|
|Access modifier|Any|Cannot be more restrictive|
|static/private/final methods|Can be overloaded|Cannot be overridden|

## Upcasting & Downcasting

**Upcasting** : creating a child class object using parent class reference

```java
Animal a = new Dog()
```

**Downcasting** : creating a parent class object with child class object

```java
Animal a = new Animal()

Dog d = a
//if is not always safe
```


---

# SOLID Principles

> **Definition:** SOLID is a set of five design principles (introduced by Robert C. Martin) that make object-oriented code easier to understand, extend and maintain.

## S – Single Responsibility Principle (SRP)

**A class should have only one reason to change.** Each class does one job.

```java
// Bad: one class does everything
class Invoice { void calculate(){} void saveToDb(){} void sendEmail(){} }

// Good: split by responsibility
class Invoice           { double calculateTotal() { return 0; } }
class InvoiceRepository { void save(Invoice i) {} }
class InvoiceMailer     { void send(Invoice i) {} }
```

## O – Open/Closed Principle (OCP)

**Open for extension, closed for modification.** Add new behaviour by adding new classes, not by editing existing, tested code.

## L – Liskov Substitution Principle (LSP)

**Objects of a subclass must be usable wherever the parent is expected, without breaking the program.** A child must honour the parent's contract.

**Violating LSP**

```java
class Rectangle {
    protected int width;
    protected int height;

    public void setWidth(int width) {
        this.width = width;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    public int area() {
        return width * height;
    }
}

class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
        this.width = width;
        this.height = width;
    }

    @Override
    public void setHeight(int height) {
        this.width = height;
        this.height = height;
    }
}
```

## I – Interface Segregation Principle (ISP)

**Clients should not be forced to depend on methods they do not use.** Prefer many small interfaces over one large one.

## D – Dependency Inversion Principle (DIP)

**High-level modules should not depend on low level modules, both should depend on abstractions (interfaces).** Dependencies are usually supplied from outside (**Dependency Injection**).

```java
// Here order sevice(High level module) is directly dependent on low level module -> email service, what if in future you want to change the notification from email to text msg?
class EmailService {
    void sendEmail(String message) {
        System.out.println("Email sent: " + message);
    }
}

class OrderService {
    private EmailService email = new EmailService();

    void placeOrder() {
        System.out.println("Order placed");
        email.sendEmail("Your order is confirmed");
    }
}

// Crating interface, and making both the modules depends on it, making this dependency inversion.
interface NotificationService {
    void send(String message);
}

class EmailService implements NotificationService {
    public void send(String message) {
        System.out.println("Sending email: " + message);
    }
}

class OrderService {
    private NotificationService service;

    OrderService(NotificationService service) {
        this.service = service;
    }

    void placeOrder() {
        service.send("Order placed!");
    }
}
```


---

# Relationship Between Objects

## Association

A weak relations ship type, relationship established between two independent object, where one can exists independently without other

```java
//student -> professor
//sudents and professors can exist independently without each other
class Student {
    String name;

    Student(String name) {
        this.name = name;
    }
}

class Teacher {
    String name;

    Teacher(String name) {
        this.name = name;
    }

    void teach(Student student) {
        System.out.println(name + " teaches " + student.name);
    }
}

public class Main {
    public static void main(String[] args) {
        Teacher t = new Teacher("Ravi");
        Student s = new Student("Arun");

        t.teach(s);
    }
}
```

## Aggregation (Has-A) (Part survives if whole is destroyed)

In a relationship where one part of the relationship survives without other.

example : student -> university number. student can exist without university number but number cannot exist.

```java
//student-> student_university_number

class Student {
    private String name;

    Student(String name) {
        this.name = name;
    }

    String getName() {
        return name;
    }

    void details() {
        System.out.println(name);
    }
}

class UniversityNumber {
    private int number;
    private Student student;

    UniversityNumber(int number, Student student) {
        this.number = number;
        this.student = student;
    }

    void assignNumber() {
        System.out.println(
            "Student Name: " + student.getName()
            + ", Number: " + number
        );
    }
}

public class Main {
    public static void main(String[] args) {
        Student s = new Student("Arun");

        UniversityNumber u = new UniversityNumber(1234, s);

        u.assignNumber();
    }
}
```

## Composition (Owns and contains) (Part dies with whole)

Strongest form of the relationship, where one cannot survive without other.

***home - > rooms***


---

# Constructors and Its Types

## Default Constructor

A constructor automatically provided by the Java compiler when a class declares no constructors. It initializes instance variables with their default values unless other initialization changes them.

```java
class Student {
    String name;
    int age;
}
//name = Null and age = 0 by default
```

## Non Argument Constructor

A constructor explicitly defined by the programmer that takes no parameters. It is used to initialize an object with predefined or default values.

```java
class Student {
    String name;
    int age;

    Student() {
        name = "Unknown";
        age = 18;
    }
}
//name = unknown, age = 18
```

## Parameterized Constructor

A constructor that accepts one or more parameters to initialize an object's fields with values supplied when the object is created.

```java
class Student {
    String name;
    int age;

    Student(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

## Copy Constructor

A user-defined constructor that accepts another object of the same class and initializes the new object using the existing object's values. Java does not automatically provide a copy constructor.

```java
Student(Student other) {
    this.name = other.name;
    this.age =  other.age;
}
```

## Constructor Overloading

Defining multiple constructors within the same class with different parameter lists, allowing objects to be initialized in different ways.

```java
class Student {
    private String name;
    private int age;

    Student() { this("Unknown", 18); }          // constructor chaining with this()

    Student(String name, int age) {
        this.name = name;                        // this = current object
        this.age = age;
    }

    Student(Student other) {                     // copy constructor
        this(other.name, other.age);
    }
}
```

## Constructor Chaining

The process of calling one constructor from another constructor. `this()` calls another constructor in the same class, while `super()` calls a constructor of the parent class.

```java
class Person {
    String name;

    Person(String name) {
        this.name = name;
        System.out.println("Person constructor");
    }
}

class Student extends Person {
    int age;

    Student(String name, int age) {
        super(name);  // Calls Person(String name)
        this.age = age;
        System.out.println("Student constructor");
    }
}
```

`this()` : Used to call another constructor of same class

```java
Student(String name,Integer age){
this(name) ;// calls another constructor which takes exactly one argument, name in the same class
this.age = age;
}
```

`this.` : used to refer to the variables of current class

`super()` : used to access the properties and behaviors of the parent class.


---

# Access Modifiers, static and final

## Access Modifiers

> **Definition:** Access modifiers control **who can see and use** a class, field, method or constructor. They are the language tool that implements encapsulation.

|Modifier|Same class|Same package|Subclass (other package)|Everywhere|
|---|---|---|---|---|
|private|Yes|No|No|No|
|default (none)|Yes|Yes|No|No|
|protected|Yes|Yes|Yes|No|
|public|Yes|Yes|Yes|Yes|


---

# Design Patterns

Design patterns are proven, reusable solutions to common software design problems. They are grouped into **Creational** (how objects are created), **Structural** (how objects are composed) and **Behavioural** (how objects communicate).

![[Pasted image 20261018210215.png]]

### Creational : 
provides mechanism for creation of the object
### Behavioral 
Defines how the object should communicates
### Structural  
How the objects needs to be structured
### 1. Singleton Pattern

Definition: Ensures that a class has only one instance and provides a way to access it.

Real-world example: One shared configuration manager for an application.
