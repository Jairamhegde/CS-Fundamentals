
# Comparison of different languages

#### C :
- High performance and more control
- Statically typed (Means the type checking of the variables/ data happens at compile time)
### Java :
- Object Oriented, used for high security, maintainability, code reusability,
- Statically typed
### Python
- Motive is to make the development faster by reducing the programming syntax complexity
- Dynamically typed

> **Imperative** : step by step instruction you write to run a program 

### How encapsulation does work??
- It wraps the data and methods into one single unit called class 
- Hides the implementation details , shows only the necessary information
- Prevent un authorized access , like objects of another class trying to modify the resource in current class
- uses private key word, which can only be changed using getter and setter
Constructors are used to initialize objects when they got created

## Java Array
---
- It cannot store different type of data directly, all the element inside the array must be of same type
- but allows to store objects of different types 
- Arrays in java are fixed length, once you declare , you cannot change its size
  so you cannot add or delete element,
- can access or modify the element using index

``` java
Object[] arr = {10, "Hello", 3.14, true};

/*To acces the actual data in this object, you have to do */*

x + 5; // ❌ /* this does not work*/

Integer n = (Integer) x; System.out.println(n + 5); // 15

```

# Inheritance

## 1. Single Inheritance

**One parent → one child**

```
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

```
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

# 2. Multilevel Inheritance

**Parent → Child → Grandchild**

```
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

```
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

# 3. Hierarchical Inheritance

**One parent → multiple children**

```
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

```
Dog d = new Dog();
d.eat();

Cat c = new Cat();
c.eat();
```

### Definition

> **Hierarchical inheritance occurs when multiple child classes inherit from the same parent class.**


# 4. Multiple Inheritance
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

```
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

```
class FlyingAnimal extends Animal, Bird {
}
```

But Java **does not allow this with classes**.

```
// ❌ Not allowed in Java
class FlyingAnimal extends Animal, Bird {
}
```

### Why?

Because it can create ambiguity — the famous **diamond problem**.

Java solves this by allowing multiple inheritance through **interfaces**.

```
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

```
FlyingAnimal f = new FlyingAnimal();

f.eat();
f.fly();
```

### Definition

> **Multiple inheritance means a class inherits from more than one parent. Java does not support multiple inheritance through classes, but it can be achieved through multiple interfaces.**

# 5. Hybrid Inheritance

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

```
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


### Diamond problem


# 4. Why is it called the Diamond Problem?

Consider:

```
       A
      / \
     B   C
      \ /
       D
```

`A` contains:

```
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



