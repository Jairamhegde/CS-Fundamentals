
Python is interpreted language, execute the code line by line

`json.dumps()` : converts a python dictionary object to json string 
`json.load()` : converts a json object to python dictionary
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


## Decorators with Parameters
#### Creating a retry

```python
import functools
import time
def retry(count,time_sec):
    def decorator(function):
        def wraper(*arg,**kwarg):
            for i in range(count):
                try:
                    if i == 3:
                        print("finish")
                        break
                    time.sleep(time_sec)
                    function(arg)
                except Exception as e:
                    print(e)
                    break
        return wraper
    return decorator
            
@retry(4,3)
def say_hi(name):
    print("Hello ,",name)
    
say_hi("jairam")
```

### Design a decorator for logging the execution time of a function

```python
def log_decorator(function_name):
    def decrator(function):
        def wraper(*arg,**kwargs):
            
            start_time = time.perf_counter()
            print("executtion started",function_name)
            result = function(*arg)
            print("Execution finished")
            end_time = time.perf_counter()
            total = end_time - start_time
            print("total_time_taken:",total)
            print("result :",result)
            
        return wraper
    return decrator
    
@log_decorator("Hello words")
def helloword(name):
    print("Hello",name)
@log_decorator("next_function")
def looping():
    for _ in range(3):
        print("Hello")

helloword("jairam")
looping()
```

## Classes in Python

```python
class Car:
	#class variable
    count = 45
    
    def __init__(self,name): #constructor
        self.name = name # isntance variable
        
	#objects method
    def brand(self):
        print(f"This car name is : {self.name}")
    
    #class method which take one default argument cls, that you have to use , 
    #just as self 
    @classmethod
    def mymethod2(cls):
        print("This is the class method")
        
    #class method which does not take any default argument 
    @staticmethod
    def mymethod():
        print("This is the class method")
        
Car.mymethod()
Car.mymethod2()
c1 = Car()
c1.brand("MG")
```

# FastAPI

## Alembic

`pip install alembic` : install alembic

`alembic init alembic` : create alembic

after this , folder structure will look like this 
```
alembic.ini
alembic/
├── env.py          # Connects Alembic to your models and database
├── script.py.mako  # Template for new migration files
└── versions/       # Directory where your migration scripts are stored
```

`alembic revision --autogenerate -m "add users table"` for auto generation of alembic using models

`alembic revision -m "add users table"` : for manual generations of revisions

`alembic upgrade head`: to apply migration

`alembic downgrade  head`: to revert migration
