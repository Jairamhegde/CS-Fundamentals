
Python is interpreted language, execute the code line by line
## What happens when we click on run of a python file
        Your Python code
              │
              ▼
       Python interpreter
          (CPython)
              │
              ▼
          Tokenizer
              │
              ▼
            Parser
              │
              ▼
             AST (Abstract Sysntax Tree)
              │
              ▼
       Compiler / compiler
              │
              ▼
          Bytecode
              │
              ▼
     Python Virtual Machine
        (bytecode eval loop)
              │
              ▼
     C implementation / runtime (JIT Compilations)
              │
              ▼
          CPU executes

### JIT Compiler
Just in time compiler which is implemented as experimental in python 3.13 and it is carried out till now. what it does it, when it see some code is repeatedly getting executed, instead of repeating the process each time, it converts them into machine code and executes directly. this makes the compilation process faster. the main word to use is `HOT CODE`

## PEP implementation
is a rule for writing the python code to enhance readability. it suggest using comments, use 4 space indents like that

## Garbage Collection in python

in python Memory Manager is responsible for running garbage collection, the memory manager keeps a heap where it store the function and data which acquire memory.  for garbage collection it uses reference counter, when a function / data is refereced, its count value increases, when then remove reference, its counter value decreases, when the count value becomes 1, it will be released.

Garbage Collection follows a cyclic process


##  OS Module in python
Used to interact with operating system. used when you want to access the data such as files and directories outside the python environment. this is used.

## Decorators in Python
are the language features which allows to extend or modify the functionality and features of a function or class without modifying its source code.

 it is a function which takes another function as input, and returns function as output.

```python
def my_decorator(func):
    def wrapper():
        print("Function started")
		func()
    return wrapper

def greet():
    print("Hello")

greet = my_decorator(greet)

greet()

# the above is equalent to this
@my_decorator
def greet():
    print("Hello")
```

#### Use case of decorators
- authentication
- authorization
- logging 
- caching
Example 
```python
def require_auth(func):

    def wrapper(user):
        if not user.is_authenticated:
            print("Access denied")
            return

        return func(user)

    return wrapper
@require_auth
def delete_user(user):
    print("User deleted")
```
