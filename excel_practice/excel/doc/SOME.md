# python

# 类

# 实例

# 变量variable类型

## 类内部变量

实例变量instance variable、类变量class variable

### 实例变量

每个实例变量在同一个类的不同实例中各自独立存储，只能通过实例引用来定义声明和访问，所以一般在类成员方法内定义实例变量。

比如声明实例变量variable_A
```
class A():
    def function(self):
        self.variable_A = 0
```

### 类变量

每个类变量在同一个类的不同实例中共享，可以通过实例引用或者类引用访问。在类成员方法内外都可以声明定义，但在实例方法内声明定义类变量需要通过实例的类引用属性__class__。

比如声明类变量variable_A、variable_B
```
class A
    variable_A = 0

    def function(self):
        self.__class__.variable_B = 0
```

## 类外部变量

### 全局变量

定义全局变量的方式有二，一个是在函数外声明定义变量，二是在函数内使用global修饰符来声明定义变量。
在函数内需要对全局变量赋值或者更新时，必须先在函数内使用global声明该变量，否则如果只需要读取全局变量，可以直接引用。

### 本地变量

在函数内定义声明的变量，如果没有使用到global修饰符，则是本地变量。本地变量生命周期仅在函数内。

# 类成员方法 method 类型

实例方法instance method、类方法class method、静态方法static method

实例方法与实例绑定，内部隐式传入实例引用，一般命名为self，可以用来引用实例变量和类变量，通过self.__class__可以得到类引用。实例方法可以被动态添加和删除，添加用到types模块，删除可以有两种方法（操作符del、函数delattr()）。

类方法，需要用修饰器@classmethod声明，内部隐式传入类引用，一般命名为cls，仅可以用来引用类变量。

静态方法，需要用修饰器@staticmethod声明，不会隐式传入类引用和实例引用。

语法格式：
```
className.static_method_name()
```

/////////////////////////////////
# 关于 try 用法

```
import sys

try:
    num = int(input("Enter a number: "))
    # raise RuntimeError("raise special exception manually")
    assert num % 2 == 0
except RuntimeError as e:
    print(f"{e}, except xxx: run when special exceptions xxx happern with info.")
except:
    print(f"{sys.exc_info()}: Not an even number!, except: run when any other exceptions happern")
else:
    reciprocal = 1/num
    print(f'{reciprocal},else: no exception happen')
finally:
    print('finally：no matter whether any exception happen')
```
/////////////////////////////////



/////////////////////////////////
# Python: with 语句只是个语法糖？

In Python, with statement is used in exception handling to make the code cleaner and much more readable. It simplifies the management of common resources like file streams. Observe the following code example on how the use of with statement makes code cleaner. 

Python3
# file handling
 
# 1) without using with statement
file = open('file_path', 'w')
file.write('hello world !')
file.close()
 
# 2) without using with statement
file = open('file_path', 'w')
try:
    file.write('hello world')
finally:
    file.close()
  

Python3
# using with statement
with open('file_path', 'w') as file:
    file.write('hello world !')
Notice that unlike the first two implementations, there is no need to call file.close() when using with statement. The with statement itself ensures proper acquisition and release of resources. An exception during the file.write() call in the first implementation can prevent the file from closing properly which may introduce several bugs in the code, i.e. many changes in files do not go into effect until the file is properly closed. The second approach in the above example takes care of all the exceptions but using the with statement makes the code compact and much more readable. Thus, with statement helps avoiding bugs and leaks by ensuring that a resource is properly released when the code using the resource is completely executed. The with statement is popularly used with file streams, as shown above and with Locks, sockets, subprocesses and telnets etc.

Supporting the 鈥渨ith鈥?statement in user defined objects
There is nothing special in open() which makes it usable with the with statement and the same functionality can be provided in user defined objects. Supporting with statement in your objects will ensure that you never leave any resource open. To use with statement in user defined objects you only need to add the methods __enter__() and __exit__() in the object methods. Consider the following example for further clarification. 

Python3
# a simple file writer object
 
class MessageWriter(object):
    def __init__(self, file_name):
        self.file_name = file_name
     
    def __enter__(self):
        self.file = open(self.file_name, 'w')
        return self.file
 
    def __exit__(self, *args):
        self.file.close()
 
# using with statement with MessageWriter
 
with MessageWriter('my_file.txt') as xfile:
    xfile.write('hello world')
Let鈥檚 examine the above code. If you notice, what follows the with keyword is the constructor of MessageWriter. As soon as the execution enters the context of the with statement a MessageWriter object is created and python then calls the __enter__() method. In this __enter__() method, initialize the resource you wish to use in the object. This __enter__() method should always return a descriptor of the acquired resource. What are resource descriptors? These are the handles provided by the operating system to access the requested resources. In the following code block, file is a descriptor of the file stream resource. 

Python
file = open('hello.txt')
In the MessageWriter example provided above, the __enter__() method creates a file descriptor and returns it. The name xfile here is used to refer to the file descriptor returned by the __enter__() method. The block of code which uses the acquired resource is placed inside the block of the with statement. As soon as the code inside the with block is executed, the __exit__() method is called. All the acquired resources are released in the __exit__() method. This is how we use the with statement with user defined objects. This interface of __enter__() and __exit__() methods which provides the support of with statement in user defined objects is called Context Manager.

The contextlib module
A class based context manager as shown above is not the only way to support the with statement in user defined objects. The contextlib module provides a few more abstractions built upon the basic context manager interface. Here is how we can rewrite the context manager for the MessageWriter object using the contextlib module. 

Python3
from contextlib import contextmanager
 
 
class MessageWriter(object):
    def __init__(self, filename):
        self.file_name = filename
 
    @contextmanager
    def open_file(self):
        try:
            file = open(self.file_name, 'w')
            yield file
        finally:
            file.close()
 
 
# usage
message_writer = MessageWriter('hello.txt')
with message_writer.open_file() as my_file:
    my_file.write('hello world')
In this code example, because of the yield statement in its definition, the function open_file() is a generator function. When this open_file() function is called, it creates a resource descriptor named file. This resource descriptor is then passed to the caller and is represented here by the variable my_file. After the code inside the with block is executed the program control returns back to the open_file() function. The open_file() function resumes its execution and executes the code following the yield statement. This part of code which appears after the yield statement releases the acquired resources. The @contextmanager here is a decorator. The previous class-based implementation and this generator-based implementation of context managers is internally the same. While the later seems more readable, it requires the knowledge of generators, decorators and yield.
/////////////////////////////////


/////////////////////////////////
# Python: 给你讲透 yield 的用法

In this article, we will cover the yield keyword in Python. Before starting, let鈥檚 understand the yield keyword definition.

Syntax of the Yield Keyword in Python

def gen_func(x):
    for i in range(x):
        yield i
What does the Yield Keyword do?
yield keyword is used to create a generator function. A type of function that is memory efficient and can be used like an iterator object.

In layman terms, the yield keyword will turn any expression that is given with it into a generator object and return it to the caller. Therefore, you must iterate over the generator object if you wish to obtain the values stored there. we will see the yield python example.

Difference between return and yield Python
The yield keyword in Python is similar to a return statement used for returning values in Python which returns a generator object to the one who calls the function which contains yield, instead of simply returning a value. The main difference between them is, the return statement terminates the execution of the function. Whereas, the yield statement only pauses the execution of the function. Another difference is return statements are never executed. whereas, yield statements are executed when the function resumes its execution.

Advantages of yield:

Using yield keyword is highly memory efficient, since the execution happens only when the caller iterates over the object.
As the variables states are saved, we can pause and resume from the same point, thus saving time.
Disadvantages of yield: 

Sometimes it becomes hard to understand the flow of code due to multiple times of value return from the function generator.
Calling of generator functions must be handled properly, else might cause errors in program.

Example 1: Generator functions and yield Keyword in Python

Generator functions behave and look just like normal functions, but with one defining characteristic. Instead of returning data, Python generator functions use the yield keyword. Generators鈥?main benefit is that they automatically create the functions __iter__() and next (). Generators offer a very tidy technique to produce data that is enormous or limitless.

Python3
def fun_generator():
    yield "Hello world!!"
    yield "Geeksforgeeks"
 
 
obj = fun_generator()
 
print(type(obj))
 
print(next(obj))
print(next(obj))
Output: 

<class 'generator'>
Hello world!!
Geeksforgeeks

Example 2: Generating an Infinite Sequence

Here, we are generating an infinite sequence of numbers with yield, yield returns the number and increments the num by + 1. 

Note: Here we can observe that num+=1 is executed after yield but in the case of a return, no execution takes place after the return keyword.

Python3
def inf_sequence():
    num = 0
    while True:
        yield num
        num += 1
         
for i in inf_sequence():
    print(i, end=" ")
Output:

0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 
26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 
49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 
72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 88 89 90 91 92 93 94 95 96.......

Example 3:  Demonstrating yield working with a list.

Here, we are extracting the even number from the list.

Python3
# generator to print even numbers
def print_even(test_list):
    for i in test_list:
        if i % 2 == 0:
            yield i
 
# initializing list
test_list = [1, 4, 5, 6, 7]
 
# printing initial list
print("The original list is : " + str(test_list))
 
# printing even numbers
print("The even numbers in list are : ", end=" ")
for j in print_even(test_list):
    print(j, end=" ")
Output: 

The original list is : [1, 4, 5, 6, 7]
The even numbers in list are :  4 6 

Example 4: Use of yield Keyword as Boolean

The possible practical application is that when handling the last amount of data and searching particulars from it, yield can be used as we don鈥檛 need to look up again from start and hence would save time. There can possibly be many applications of yield depending upon the use cases. 

Python3
# func to count number of given word
def print_even(test_string):
    for i in test_string:
        if i == "geeks":
            yield i
 
 
# initializing string
test_string = " The are many geeks around you, \
              geeks are known for teaching other geeks"
 
# count numbers of geeks used in string
count = 0
print("The number of geeks in string is : ", end="")
test_string = test_string.split()
 
for j in print_even(test_string):
    count = count + 1
 
print(count)
Output

The number of geeks in string is: 3
/////////////////////////////////

excel

import openpyxl
from openpyxl.drawing.image import Image

wb = openpyxl.Workbook()

sheet = wb.active

# Adding a row of data to the worksheet (used to
# distinguish previous excel data from the image)
sheet.append([10, 2010, "Geeks", 4, "life"])

# A wrapper over PIL.Image, used to provide image
# inclusion properties to openpyxl library
img = Image("geek.jpg")

# Adding the image to the worksheet
# (with attributes like position)
sheet.add_image(img, 'A2')

# Saving the workbook created
wb.save('sample.xlsx')

【文末送图书门票】Python:Excel 自动化实践 openpyxl入门篇

【文末送图书门票】头痛,这么多Excel表怎么弄嘛？

/////////////////////////////////
Excel spreadsheets are one of those things you might have to deal with at some point. Either it’s because your boss loves them or because marketing needs them, you might have to learn how to work with spreadsheets, and that’s when knowing openpyxl comes in handy!

Spreadsheets are a very intuitive and user-friendly way to manipulate large datasets without any prior technical background. That’s why they’re still so commonly used today.

In this article, you’ll learn how to use openpyxl to:

Manipulate Excel spreadsheets with confidence
Extract information from spreadsheets
Create simple or more complex spreadsheets, including adding styles, charts, and so on
This article is written for intermediate developers who have a pretty good knowledge of Python data structures, such as dicts and lists, but also feel comfortable around OOP and more intermediate level topics.

Download Dataset: Click here to download the dataset for the openpyxl exercise you’ll be following in this tutorial.

Before You Begin
If you ever get asked to extract some data from a database or log file into an Excel spreadsheet, or if you often have to convert an Excel spreadsheet into some more usable programmatic form, then this tutorial is perfect for you. Let’s jump into the openpyxl caravan!

Practical Use Cases
First things first, when would you need to use a package like openpyxl in a real-world scenario? You’ll see a few examples below, but really, there are hundreds of possible scenarios where this knowledge could come in handy.

# 

Importing New Products Into a Database
You are responsible for tech in an online store company, and your boss doesn’t want to pay for a cool and expensive CMS system.

Every time they want to add new products to the online store, they come to you with an Excel spreadsheet with a few hundred rows and, for each of them, you have the product name, description, price, and so forth.

Now, to import the data, you’ll have to iterate over each spreadsheet row and add each product to the online store.

Exporting Database Data Into a Spreadsheet
Say you have a Database table where you record all your users’ information, including name, phone number, email address, and so forth.

Now, the Marketing team wants to contact all users to give them some discounted offer or promotion. However, they don’t have access to the Database, or they don’t know how to use SQL to extract that information easily.

What can you do to help? Well, you can make a quick script using openpyxl that iterates over every single User record and puts all the essential information into an Excel spreadsheet.

That’s gonna earn you an extra slice of cake at your company’s next birthday party!

Appending Information to an Existing Spreadsheet
You may also have to open a spreadsheet, read the information in it and, according to some business logic, append more data to it.

For example, using the online store scenario again, say you get an Excel spreadsheet with a list of users and you need to append to each row the total amount they’ve spent in your store.

This data is in the Database and, in order to do this, you have to read the spreadsheet, iterate through each row, fetch the total amount spent from the Database and then write back to the spreadsheet.

Not a problem for openpyxl!

Learning Some Basic Excel Terminology
Here’s a quick list of basic terms you’ll see when you’re working with Excel spreadsheets:

Term	Explanation
Spreadsheet or Workbook	A Spreadsheet is the main file you are creating or working with.
Worksheet or Sheet	A Sheet is used to split different kinds of content within the same spreadsheet. A Spreadsheet can have one or more Sheets.
Column	A Column is a vertical line, and it’s represented by an uppercase letter: A.
Row	A Row is a horizontal line, and it’s represented by a number: 1.
Cell	A Cell is a combination of Column and Row, represented by both an uppercase letter and a number: A1.
Getting Started With openpyxl
Now that you’re aware of the benefits of a tool like openpyxl, let’s get down to it and start by installing the package. For this tutorial, you should use Python 3.7 and openpyxl 2.6.2. To install the package, you can do the following:

$ pip install openpyxl
After you install the package, you should be able to create a super simple spreadsheet with the following code:

from openpyxl import Workbook

workbook = Workbook()
sheet = workbook.active

sheet["A1"] = "hello"
sheet["B1"] = "world!"

workbook.save(filename="hello_world.xlsx")
The code above should create a file called hello_world.xlsx in the folder you are using to run the code. If you open that file with Excel you should see something like this:

A Simple Hello World Spreadsheet
Woohoo, your first spreadsheet created!


Remove ads
Reading Excel Spreadsheets With openpyxl
Let’s start with the most essential thing one can do with a spreadsheet: read it.

You’ll go from a straightforward approach to reading a spreadsheet to more complex examples where you read the data and convert it into more useful Python structures.

Dataset for This Tutorial
Before you dive deep into some code examples, you should download this sample dataset and store it somewhere as sample.xlsx:

Download Dataset: Click here to download the dataset for the openpyxl exercise you’ll be following in this tutorial.

This is one of the datasets you’ll be using throughout this tutorial, and it’s a spreadsheet with a sample of real data from Amazon’s online product reviews. This dataset is only a tiny fraction of what Amazon provides, but for testing purposes, it’s more than enough.

A Simple Approach to Reading an Excel Spreadsheet
Finally, let’s start reading some spreadsheets! To begin with, open our sample spreadsheet:

>>> from openpyxl import load_workbook
>>> workbook = load_workbook(filename="sample.xlsx")
>>> workbook.sheetnames
['Sheet 1']

>>> sheet = workbook.active
>>> sheet
<Worksheet "Sheet 1">

>>> sheet.title
'Sheet 1'
In the code above, you first open the spreadsheet sample.xlsx using load_workbook(), and then you can use workbook.sheetnames to see all the sheets you have available to work with. After that, workbook.active selects the first available sheet and, in this case, you can see that it selects Sheet 1 automatically. Using these methods is the default way of opening a spreadsheet, and you’ll see it many times during this tutorial.

Now, after opening a spreadsheet, you can easily retrieve data from it like this:

>>> sheet["A1"]
<Cell 'Sheet 1'.A1>

>>> sheet["A1"].value
'marketplace'

>>> sheet["F10"].value
"G-Shock Men's Grey Sport Watch"
To return the actual value of a cell, you need to do .value. Otherwise, you’ll get the main Cell object. You can also use the method .cell() to retrieve a cell using index notation. Remember to add .value to get the actual value and not a Cell object:

>>> sheet.cell(row=10, column=6)
<Cell 'Sheet 1'.F10>

>>> sheet.cell(row=10, column=6).value
"G-Shock Men's Grey Sport Watch"
You can see that the results returned are the same, no matter which way you decide to go with. However, in this tutorial, you’ll be mostly using the first approach: ["A1"].

Note: Even though in Python you’re used to a zero-indexed notation, with spreadsheets you’ll always use a one-indexed notation where the first row or column always has index 1.

The above shows you the quickest way to open a spreadsheet. However, you can pass additional parameters to change the way a spreadsheet is loaded.

Additional Reading Options
There are a few arguments you can pass to load_workbook() that change the way a spreadsheet is loaded. The most important ones are the following two Booleans:

read_only loads a spreadsheet in read-only mode allowing you to open very large Excel files.
data_only ignores loading formulas and instead loads only the resulting values.

Remove ads
Importing Data From a Spreadsheet
Now that you’ve learned the basics about loading a spreadsheet, it’s about time you get to the fun part: the iteration and actual usage of the values within the spreadsheet.

This section is where you’ll learn all the different ways you can iterate through the data, but also how to convert that data into something usable and, more importantly, how to do it in a Pythonic way.

Iterating Through the Data
There are a few different ways you can iterate through the data depending on your needs.

You can slice the data with a combination of columns and rows:

>>> sheet["A1:C2"]
((<Cell 'Sheet 1'.A1>, <Cell 'Sheet 1'.B1>, <Cell 'Sheet 1'.C1>),
 (<Cell 'Sheet 1'.A2>, <Cell 'Sheet 1'.B2>, <Cell 'Sheet 1'.C2>))
You can get ranges of rows or columns:

>>> # Get all cells from column A
>>> sheet["A"]
(<Cell 'Sheet 1'.A1>,
 <Cell 'Sheet 1'.A2>,
 ...
 <Cell 'Sheet 1'.A99>,
 <Cell 'Sheet 1'.A100>)

>>> # Get all cells for a range of columns
>>> sheet["A:B"]
((<Cell 'Sheet 1'.A1>,
  <Cell 'Sheet 1'.A2>,
  ...
  <Cell 'Sheet 1'.A99>,
  <Cell 'Sheet 1'.A100>),
 (<Cell 'Sheet 1'.B1>,
  <Cell 'Sheet 1'.B2>,
  ...
  <Cell 'Sheet 1'.B99>,
  <Cell 'Sheet 1'.B100>))

>>> # Get all cells from row 5
>>> sheet[5]
(<Cell 'Sheet 1'.A5>,
 <Cell 'Sheet 1'.B5>,
 ...
 <Cell 'Sheet 1'.N5>,
 <Cell 'Sheet 1'.O5>)

>>> # Get all cells for a range of rows
>>> sheet[5:6]
((<Cell 'Sheet 1'.A5>,
  <Cell 'Sheet 1'.B5>,
  ...
  <Cell 'Sheet 1'.N5>,
  <Cell 'Sheet 1'.O5>),
 (<Cell 'Sheet 1'.A6>,
  <Cell 'Sheet 1'.B6>,
  ...
  <Cell 'Sheet 1'.N6>,
  <Cell 'Sheet 1'.O6>))
You’ll notice that all of the above examples return a tuple. If you want to refresh your memory on how to handle tuples in Python, check out the article on Lists and Tuples in Python.

There are also multiple ways of using normal Python generators to go through the data. The main methods you can use to achieve this are:

.iter_rows()
.iter_cols()
Both methods can receive the following arguments:

min_row
max_row
min_col
max_col
These arguments are used to set boundaries for the iteration:

>>> for row in sheet.iter_rows(min_row=1,
...                            max_row=2,
...                            min_col=1,
...                            max_col=3):
...     print(row)
(<Cell 'Sheet 1'.A1>, <Cell 'Sheet 1'.B1>, <Cell 'Sheet 1'.C1>)
(<Cell 'Sheet 1'.A2>, <Cell 'Sheet 1'.B2>, <Cell 'Sheet 1'.C2>)


>>> for column in sheet.iter_cols(min_row=1,
...                               max_row=2,
...                               min_col=1,
...                               max_col=3):
...     print(column)
(<Cell 'Sheet 1'.A1>, <Cell 'Sheet 1'.A2>)
(<Cell 'Sheet 1'.B1>, <Cell 'Sheet 1'.B2>)
(<Cell 'Sheet 1'.C1>, <Cell 'Sheet 1'.C2>)
You’ll notice that in the first example, when iterating through the rows using .iter_rows(), you get one tuple element per row selected. While when using .iter_cols() and iterating through columns, you’ll get one tuple per column instead.

One additional argument you can pass to both methods is the Boolean values_only. When it’s set to True, the values of the cell are returned, instead of the Cell object:

>>> for value in sheet.iter_rows(min_row=1,
...                              max_row=2,
...                              min_col=1,
...                              max_col=3,
...                              values_only=True):
...     print(value)
('marketplace', 'customer_id', 'review_id')
('US', 3653882, 'R3O9SGZBVQBV76')
If you want to iterate through the whole dataset, then you can also use the attributes .rows or .columns directly, which are shortcuts to using .iter_rows() and .iter_cols() without any arguments:

>>> for row in sheet.rows:
...     print(row)
(<Cell 'Sheet 1'.A1>, <Cell 'Sheet 1'.B1>, <Cell 'Sheet 1'.C1>
...
<Cell 'Sheet 1'.M100>, <Cell 'Sheet 1'.N100>, <Cell 'Sheet 1'.O100>)
These shortcuts are very useful when you’re iterating through the whole dataset.

Manipulate Data Using Python’s Default Data Structures
Now that you know the basics of iterating through the data in a workbook, let’s look at smart ways of converting that data into Python structures.

As you saw earlier, the result from all iterations comes in the form of tuples. However, since a tuple is nothing more than an immutable list, you can easily access its data and transform it into other structures.

For example, say you want to extract product information from the sample.xlsx spreadsheet and into a dictionary where each key is a product ID.

A straightforward way to do this is to iterate over all the rows, pick the columns you know are related to product information, and then store that in a dictionary. Let’s code this out!

First of all, have a look at the headers and see what information you care most about:

>>> for value in sheet.iter_rows(min_row=1,
...                              max_row=1,
...                              values_only=True):
...     print(value)
('marketplace', 'customer_id', 'review_id', 'product_id', ...)
This code returns a list of all the column names you have in the spreadsheet. To start, grab the columns with names:

product_id
product_parent
product_title
product_category
Lucky for you, the columns you need are all next to each other so you can use the min_column and max_column to easily get the data you want:

>>> for value in sheet.iter_rows(min_row=2,
...                              min_col=4,
...                              max_col=7,
...                              values_only=True):
...     print(value)
('B00FALQ1ZC', 937001370, 'Invicta Women\'s 15150 "Angel" 18k Yellow...)
('B00D3RGO20', 484010722, "Kenneth Cole New York Women's KC4944...)
...
Nice! Now that you know how to get all the important product information you need, let’s put that data into a dictionary:

import json
from openpyxl import load_workbook

workbook = load_workbook(filename="sample.xlsx")
sheet = workbook.active

products = {}

# Using the values_only because you want to return the cells' values
for row in sheet.iter_rows(min_row=2,
                           min_col=4,
                           max_col=7,
                           values_only=True):
    product_id = row[0]
    product = {
        "parent": row[1],
        "title": row[2],
        "category": row[3]
    }
    products[product_id] = product

# Using json here to be able to format the output for displaying later
print(json.dumps(products))
The code above returns a JSON similar to this:

{
  "B00FALQ1ZC": {
    "parent": 937001370,
    "title": "Invicta Women's 15150 ...",
    "category": "Watches"
  },
  "B00D3RGO20": {
    "parent": 484010722,
    "title": "Kenneth Cole New York ...",
    "category": "Watches"
  }
}
Here you can see that the output is trimmed to 2 products only, but if you run the script as it is, then you should get 98 products.

Convert Data Into Python Classes
To finalize the reading section of this tutorial, let’s dive into Python classes and see how you could improve on the example above and better structure the data.

For this, you’ll be using the new Python Data Classes that are available from Python 3.7. If you’re using an older version of Python, then you can use the default Classes instead.

So, first things first, let’s look at the data you have and decide what you want to store and how you want to store it.

As you saw right at the start, this data comes from Amazon, and it’s a list of product reviews. You can check the list of all the columns and their meaning on Amazon.

There are two significant elements you can extract from the data available:

Products
Reviews
A Product has:

ID
Title
Parent
Category
The Review has a few more fields:

ID
Customer ID
Stars
Headline
Body
Date
You can ignore a few of the review fields to make things a bit simpler.

So, a straightforward implementation of these two classes could be written in a separate file classes.py:

import datetime
from dataclasses import dataclass

@dataclass
class Product:
    id: str
    parent: str
    title: str
    category: str

@dataclass
class Review:
    id: str
    customer_id: str
    stars: int
    headline: str
    body: str
    date: datetime.datetime
After defining your data classes, you need to convert the data from the spreadsheet into these new structures.

Before doing the conversion, it’s worth looking at our header again and creating a mapping between columns and the fields you need:

>>> for value in sheet.iter_rows(min_row=1,
...                              max_row=1,
...                              values_only=True):
...     print(value)
('marketplace', 'customer_id', 'review_id', 'product_id', ...)

>>> # Or an alternative
>>> for cell in sheet[1]:
...     print(cell.value)
marketplace
customer_id
review_id
product_id
product_parent
...
Let’s create a file mapping.py where you have a list of all the field names and their column location (zero-indexed) on the spreadsheet:

# Product fields
PRODUCT_ID = 3
PRODUCT_PARENT = 4
PRODUCT_TITLE = 5
PRODUCT_CATEGORY = 6

# Review fields
REVIEW_ID = 2
REVIEW_CUSTOMER = 1
REVIEW_STARS = 7
REVIEW_HEADLINE = 12
REVIEW_BODY = 13
REVIEW_DATE = 14
You don’t necessarily have to do the mapping above. It’s more for readability when parsing the row data, so you don’t end up with a lot of magic numbers lying around.

Finally, let’s look at the code needed to parse the spreadsheet data into a list of product and review objects:

from datetime import datetime
from openpyxl import load_workbook
from classes import Product, Review
from mapping import PRODUCT_ID, PRODUCT_PARENT, PRODUCT_TITLE, \
    PRODUCT_CATEGORY, REVIEW_DATE, REVIEW_ID, REVIEW_CUSTOMER, \
    REVIEW_STARS, REVIEW_HEADLINE, REVIEW_BODY

# Using the read_only method since you're not gonna be editing the spreadsheet
workbook = load_workbook(filename="sample.xlsx", read_only=True)
sheet = workbook.active

products = []
reviews = []

# Using the values_only because you just want to return the cell value
for row in sheet.iter_rows(min_row=2, values_only=True):
    product = Product(id=row[PRODUCT_ID],
                      parent=row[PRODUCT_PARENT],
                      title=row[PRODUCT_TITLE],
                      category=row[PRODUCT_CATEGORY])
    products.append(product)

    # You need to parse the date from the spreadsheet into a datetime format
    spread_date = row[REVIEW_DATE]
    parsed_date = datetime.strptime(spread_date, "%Y-%m-%d")

    review = Review(id=row[REVIEW_ID],
                    customer_id=row[REVIEW_CUSTOMER],
                    stars=row[REVIEW_STARS],
                    headline=row[REVIEW_HEADLINE],
                    body=row[REVIEW_BODY],
                    date=parsed_date)
    reviews.append(review)

print(products[0])
print(reviews[0])
After you run the code above, you should get some output like this:

Product(id='B00FALQ1ZC', parent=937001370, ...)
Review(id='R3O9SGZBVQBV76', customer_id=3653882, ...)
That’s it! Now you should have the data in a very simple and digestible class format, and you can start thinking of storing this in a Database or any other type of data storage you like.

Using this kind of OOP strategy to parse spreadsheets makes handling the data much simpler later on.

写入数据

Remove ads
Appending New Data
Before you start creating very complex spreadsheets, have a quick look at an example of how to append data to an existing spreadsheet.

Go back to the first example spreadsheet you created (hello_world.xlsx) and try opening it and appending some data to it, like this:

from openpyxl import load_workbook

# Start by opening the spreadsheet and selecting the main sheet
workbook = load_workbook(filename="hello_world.xlsx")
sheet = workbook.active

# Write what you want into a specific cell
sheet["C1"] = "writing ;)"

# Save the spreadsheet
workbook.save(filename="hello_world_append.xlsx")
Et voilà, if you open the new hello_world_append.xlsx spreadsheet, you’ll see the following change:

Appending Data to a Spreadsheet
Notice the additional writing ;) on cell C1.

Writing Excel Spreadsheets With openpyxl
There are a lot of different things you can write to a spreadsheet, from simple text or number values to complex formulas, charts, or even images.

Let’s start creating some spreadsheets!

Creating a Simple Spreadsheet
Previously, you saw a very quick example of how to write “Hello world!” into a spreadsheet, so you can start with that:

from openpyxl import Workbook

filename = "hello_world.xlsx"

workbook = Workbook()
sheet = workbook.active

sheet["A1"] = "hello"
sheet["B1"] = "world!"

workbook.save(filename=filename)
The highlighted lines in the code above are the most important ones for writing. In the code, you can see that:

Line 5 shows you how to create a new empty workbook.
Lines 8 and 9 show you how to add data to specific cells.
Line 11 shows you how to save the spreadsheet when you’re done.
Even though these lines above can be straightforward, it’s still good to know them well for when things get a bit more complicated.

Note: You’ll be using the hello_world.xlsx spreadsheet for some of the upcoming examples, so keep it handy.

One thing you can do to help with coming code examples is add the following method to your Python file or console:

>>> def print_rows():
...     for row in sheet.iter_rows(values_only=True):
...         print(row)
It makes it easier to print all of your spreadsheet values by just calling print_rows().


Remove ads
Basic Spreadsheet Operations
Before you get into the more advanced topics, it’s good for you to know how to manage the most simple elements of a spreadsheet.

Adding and Updating Cell Values
You already learned how to add values to a spreadsheet like this:

>>> sheet["A1"] = "value"
There’s another way you can do this, by first selecting a cell and then changing its value:

>>> cell = sheet["A1"]
>>> cell
<Cell 'Sheet'.A1>

>>> cell.value
'hello'

>>> cell.value = "hey"
>>> cell.value
'hey'
The new value is only stored into the spreadsheet once you call workbook.save().

The openpyxl creates a cell when adding a value, if that cell didn’t exist before:

>>> # Before, our spreadsheet has only 1 row
>>> print_rows()
('hello', 'world!')

>>> # Try adding a value to row 10
>>> sheet["B10"] = "test"
>>> print_rows()
('hello', 'world!')
(None, None)
(None, None)
(None, None)
(None, None)
(None, None)
(None, None)
(None, None)
(None, None)
(None, 'test')
As you can see, when trying to add a value to cell B10, you end up with a tuple with 10 rows, just so you can have that test value.

Managing Rows and Columns
One of the most common things you have to do when manipulating spreadsheets is adding or removing rows and columns. The openpyxl package allows you to do that in a very straightforward way by using the methods:

.insert_rows()
.delete_rows()
.insert_cols()
.delete_cols()
Every single one of those methods can receive two arguments:

idx
amount
Using our basic hello_world.xlsx example again, let’s see how these methods work:

>>> print_rows()
('hello', 'world!')

>>> # Insert a column before the existing column 1 ("A")
>>> sheet.insert_cols(idx=1)
>>> print_rows()
(None, 'hello', 'world!')

>>> # Insert 5 columns between column 2 ("B") and 3 ("C")
>>> sheet.insert_cols(idx=3, amount=5)
>>> print_rows()
(None, 'hello', None, None, None, None, None, 'world!')

>>> # Delete the created columns
>>> sheet.delete_cols(idx=3, amount=5)
>>> sheet.delete_cols(idx=1)
>>> print_rows()
('hello', 'world!')

>>> # Insert a new row in the beginning
>>> sheet.insert_rows(idx=1)
>>> print_rows()
(None, None)
('hello', 'world!')

>>> # Insert 3 new rows in the beginning
>>> sheet.insert_rows(idx=1, amount=3)
>>> print_rows()
(None, None)
(None, None)
(None, None)
(None, None)
('hello', 'world!')

>>> # Delete the first 4 rows
>>> sheet.delete_rows(idx=1, amount=4)
>>> print_rows()
('hello', 'world!')
The only thing you need to remember is that when inserting new data (rows or columns), the insertion happens before the idx parameter.

So, if you do insert_rows(1), it inserts a new row before the existing first row.

It’s the same for columns: when you call insert_cols(2), it inserts a new column right before the already existing second column (B).

However, when deleting rows or columns, .delete_... deletes data starting from the index passed as an argument.

For example, when doing delete_rows(2) it deletes row 2, and when doing delete_cols(3) it deletes the third column (C).

Managing Sheets
Sheet management is also one of those things you might need to know, even though it might be something that you don’t use that often.

If you look back at the code examples from this tutorial, you’ll notice the following recurring piece of code:

sheet = workbook.active
This is the way to select the default sheet from a spreadsheet. However, if you’re opening a spreadsheet with multiple sheets, then you can always select a specific one like this:

>>> # Let's say you have two sheets: "Products" and "Company Sales"
>>> workbook.sheetnames
['Products', 'Company Sales']

>>> # You can select a sheet using its title
>>> products_sheet = workbook["Products"]
>>> sales_sheet = workbook["Company Sales"]
You can also change a sheet title very easily:

>>> workbook.sheetnames
['Products', 'Company Sales']

>>> products_sheet = workbook["Products"]
>>> products_sheet.title = "New Products"

>>> workbook.sheetnames
['New Products', 'Company Sales']
If you want to create or delete sheets, then you can also do that with .create_sheet() and .remove():

>>> workbook.sheetnames
['Products', 'Company Sales']

>>> operations_sheet = workbook.create_sheet("Operations")
>>> workbook.sheetnames
['Products', 'Company Sales', 'Operations']

>>> # You can also define the position to create the sheet at
>>> hr_sheet = workbook.create_sheet("HR", 0)
>>> workbook.sheetnames
['HR', 'Products', 'Company Sales', 'Operations']

>>> # To remove them, just pass the sheet as an argument to the .remove()
>>> workbook.remove(operations_sheet)
>>> workbook.sheetnames
['HR', 'Products', 'Company Sales']

>>> workbook.remove(hr_sheet)
>>> workbook.sheetnames
['Products', 'Company Sales']
One other thing you can do is make duplicates of a sheet using copy_worksheet():

>>> workbook.sheetnames
['Products', 'Company Sales']

>>> products_sheet = workbook["Products"]
>>> workbook.copy_worksheet(products_sheet)
<Worksheet "Products Copy">

>>> workbook.sheetnames
['Products', 'Company Sales', 'Products Copy']
If you open your spreadsheet after saving the above code, you’ll notice that the sheet Products Copy is a duplicate of the sheet Products.

Freezing Rows and Columns
Something that you might want to do when working with big spreadsheets is to freeze a few rows or columns, so they remain visible when you scroll right or down.

Freezing data allows you to keep an eye on important rows or columns, regardless of where you scroll in the spreadsheet.

Again, openpyxl also has a way to accomplish this by using the worksheet freeze_panes attribute. For this example, go back to our sample.xlsx spreadsheet and try doing the following:

>>> workbook = load_workbook(filename="sample.xlsx")
>>> sheet = workbook.active
>>> sheet.freeze_panes = "C2"
>>> workbook.save("sample_frozen.xlsx")
If you open the sample_frozen.xlsx spreadsheet in your favorite spreadsheet editor, you’ll notice that row 1 and columns A and B are frozen and are always visible no matter where you navigate within the spreadsheet.

This feature is handy, for example, to keep headers within sight, so you always know what each column represents.

Here’s how it looks in the editor:

Example Spreadsheet With Frozen Rows and Columns
Notice how you’re at the end of the spreadsheet, and yet, you can see both row 1 and columns A and B.

Adding Filters
You can use openpyxl to add filters and sorts to your spreadsheet. However, when you open the spreadsheet, the data won’t be rearranged according to these sorts and filters.

At first, this might seem like a pretty useless feature, but when you’re programmatically creating a spreadsheet that is going to be sent and used by somebody else, it’s still nice to at least create the filters and allow people to use it afterward.

The code below is an example of how you would add some filters to our existing sample.xlsx spreadsheet:

>>> # Check the used spreadsheet space using the attribute "dimensions"
>>> sheet.dimensions
'A1:O100'

>>> sheet.auto_filter.ref = "A1:O100"
>>> workbook.save(filename="sample_with_filters.xlsx")
You should now see the filters created when opening the spreadsheet in your editor:

Example Spreadsheet With Filters
You don’t have to use sheet.dimensions if you know precisely which part of the spreadsheet you want to apply filters to.


Remove ads
Adding Formulas
Formulas (or formulae) are one of the most powerful features of spreadsheets.

They gives you the power to apply specific mathematical equations to a range of cells. Using formulas with openpyxl is as simple as editing the value of a cell.

You can see the list of formulas supported by openpyxl:

>>> from openpyxl.utils import FORMULAE
>>> FORMULAE
frozenset({'ABS',
           'ACCRINT',
           'ACCRINTM',
           'ACOS',
           'ACOSH',
           'AMORDEGRC',
           'AMORLINC',
           'AND',
           ...
           'YEARFRAC',
           'YIELD',
           'YIELDDISC',
           'YIELDMAT',
           'ZTEST'})
Let’s add some formulas to our sample.xlsx spreadsheet.

Starting with something easy, let’s check the average star rating for the 99 reviews within the spreadsheet:

>>> # Star rating is column "H"
>>> sheet["P2"] = "=AVERAGE(H2:H100)"
>>> workbook.save(filename="sample_formulas.xlsx")
If you open the spreadsheet now and go to cell P2, you should see that its value is: 4.18181818181818. Have a look in the editor:

Example Spreadsheet With Average Formula
You can use the same methodology to add any formulas to your spreadsheet. For example, let’s count the number of reviews that had helpful votes:

>>> # The helpful votes are counted on column "I"
>>> sheet["P3"] = '=COUNTIF(I2:I100, ">0")'
>>> workbook.save(filename="sample_formulas.xlsx")
You should get the number 21 on your P3 spreadsheet cell like so:

Example Spreadsheet With Average and CountIf Formula
You’ll have to make sure that the strings within a formula are always in double quotes, so you either have to use single quotes around the formula like in the example above or you’ll have to escape the double quotes inside the formula: "=COUNTIF(I2:I100, \">0\")".

There are a ton of other formulas you can add to your spreadsheet using the same procedure you tried above. Give it a go yourself!

Adding Styles
Even though styling a spreadsheet might not be something you would do every day, it’s still good to know how to do it.

Using openpyxl, you can apply multiple styling options to your spreadsheet, including fonts, borders, colors, and so on. Have a look at the openpyxl documentation to learn more.

You can also choose to either apply a style directly to a cell or create a template and reuse it to apply styles to multiple cells.

Let’s start by having a look at simple cell styling, using our sample.xlsx again as the base spreadsheet:

>>> # Import necessary style classes
>>> from openpyxl.styles import Font, Color, Alignment, Border, Side

>>> # Create a few styles
>>> bold_font = Font(bold=True)
>>> big_red_text = Font(color="00FF0000", size=20)
>>> center_aligned_text = Alignment(horizontal="center")
>>> double_border_side = Side(border_style="double")
>>> square_border = Border(top=double_border_side,
...                        right=double_border_side,
...                        bottom=double_border_side,
...                        left=double_border_side)

>>> # Style some cells!
>>> sheet["A2"].font = bold_font
>>> sheet["A3"].font = big_red_text
>>> sheet["A4"].alignment = center_aligned_text
>>> sheet["A5"].border = square_border
>>> workbook.save(filename="sample_styles.xlsx")
If you open your spreadsheet now, you should see quite a few different styles on the first 5 cells of column A:

Example Spreadsheet With Simple Cell Styles
There you go. You got:

A2 with the text in bold
A3 with the text in red and bigger font size
A4 with the text centered
A5 with a square border around the text
Note: For the colors, you can also use HEX codes instead by doing Font(color="C70E0F").

You can also combine styles by simply adding them to the cell at the same time:

>>> # Reusing the same styles from the example above
>>> sheet["A6"].alignment = center_aligned_text
>>> sheet["A6"].font = big_red_text
>>> sheet["A6"].border = square_border
>>> workbook.save(filename="sample_styles.xlsx")
Have a look at cell A6 here:

Example Spreadsheet With Coupled Cell Styles
When you want to apply multiple styles to one or several cells, you can use a NamedStyle class instead, which is like a style template that you can use over and over again. Have a look at the example below:

>>> from openpyxl.styles import NamedStyle

>>> # Let's create a style template for the header row
>>> header = NamedStyle(name="header")
>>> header.font = Font(bold=True)
>>> header.border = Border(bottom=Side(border_style="thin"))
>>> header.alignment = Alignment(horizontal="center", vertical="center")

>>> # Now let's apply this to all first row (header) cells
>>> header_row = sheet[1]
>>> for cell in header_row:
...     cell.style = header

>>> workbook.save(filename="sample_styles.xlsx")
If you open the spreadsheet now, you should see that its first row is bold, the text is aligned to the center, and there’s a small bottom border! Have a look below:

Example Spreadsheet With Named Styles
As you saw above, there are many options when it comes to styling, and it depends on the use case, so feel free to check openpyxl documentation and see what other things you can do.


Remove ads
Conditional Formatting
This feature is one of my personal favorites when it comes to adding styles to a spreadsheet.

It’s a much more powerful approach to styling because it dynamically applies styles according to how the data in the spreadsheet changes.

In a nutshell, conditional formatting allows you to specify a list of styles to apply to a cell (or cell range) according to specific conditions.

For example, a widespread use case is to have a balance sheet where all the negative totals are in red, and the positive ones are in green. This formatting makes it much more efficient to spot good vs bad periods.

Without further ado, let’s pick our favorite spreadsheet—sample.xlsx—and add some conditional formatting.

You can start by adding a simple one that adds a red background to all reviews with less than 3 stars:

>>> from openpyxl.styles import PatternFill
>>> from openpyxl.styles.differential import DifferentialStyle
>>> from openpyxl.formatting.rule import Rule

>>> red_background = PatternFill(fgColor="00FF0000")
>>> diff_style = DifferentialStyle(fill=red_background)
>>> rule = Rule(type="expression", dxf=diff_style)
>>> rule.formula = ["$H1<3"]
>>> sheet.conditional_formatting.add("A1:O100", rule)
>>> workbook.save("sample_conditional_formatting.xlsx")
Now you’ll see all the reviews with a star rating below 3 marked with a red background:

Example Spreadsheet With Simple Conditional Formatting
Code-wise, the only things that are new here are the objects DifferentialStyle and Rule:

DifferentialStyle is quite similar to NamedStyle, which you already saw above, and it’s used to aggregate multiple styles such as fonts, borders, alignment, and so forth.
Rule is responsible for selecting the cells and applying the styles if the cells match the rule’s logic.
Using a Rule object, you can create numerous conditional formatting scenarios.

However, for simplicity sake, the openpyxl package offers 3 built-in formats that make it easier to create a few common conditional formatting patterns. These built-ins are:

ColorScale
IconSet
DataBar
The ColorScale gives you the ability to create color gradients:

>>> from openpyxl.formatting.rule import ColorScaleRule
>>> color_scale_rule = ColorScaleRule(start_type="min",
...                                   start_color="00FF0000",  # Red
...                                   end_type="max",
...                                   end_color="0000FF00")  # Green

>>> # Again, let's add this gradient to the star ratings, column "H"
>>> sheet.conditional_formatting.add("H2:H100", color_scale_rule)
>>> workbook.save(filename="sample_conditional_formatting_color_scale.xlsx")
Now you should see a color gradient on column H, from red to green, according to the star rating:

Example Spreadsheet With Color Scale Conditional Formatting
You can also add a third color and make two gradients instead:

>>> from openpyxl.formatting.rule import ColorScaleRule
>>> color_scale_rule = ColorScaleRule(start_type="num",
...                                   start_value=1,
...                                   start_color="00FF0000",  # Red
...                                   mid_type="num",
...                                   mid_value=3,
...                                   mid_color="00FFFF00",  # Yellow
...                                   end_type="num",
...                                   end_value=5,
...                                   end_color="0000FF00")  # Green

>>> # Again, let's add this gradient to the star ratings, column "H"
>>> sheet.conditional_formatting.add("H2:H100", color_scale_rule)
>>> workbook.save(filename="sample_conditional_formatting_color_scale_3.xlsx")
This time, you’ll notice that star ratings between 1 and 3 have a gradient from red to yellow, and star ratings between 3 and 5 have a gradient from yellow to green:

Example Spreadsheet With 2 Color Scales Conditional Formatting
The IconSet allows you to add an icon to the cell according to its value:

>>> from openpyxl.formatting.rule import IconSetRule

>>> icon_set_rule = IconSetRule("5Arrows", "num", [1, 2, 3, 4, 5])
>>> sheet.conditional_formatting.add("H2:H100", icon_set_rule)
>>> workbook.save("sample_conditional_formatting_icon_set.xlsx")
You’ll see a colored arrow next to the star rating. This arrow is red and points down when the value of the cell is 1 and, as the rating gets better, the arrow starts pointing up and becomes green:

Example Spreadsheet With Icon Set Conditional Formatting
The openpyxl package has a full list of other icons you can use, besides the arrow.

Finally, the DataBar allows you to create progress bars:

>>> from openpyxl.formatting.rule import DataBarRule

>>> data_bar_rule = DataBarRule(start_type="num",
...                             start_value=1,
...                             end_type="num",
...                             end_value="5",
...                             color="0000FF00")  # Green
>>> sheet.conditional_formatting.add("H2:H100", data_bar_rule)
>>> workbook.save("sample_conditional_formatting_data_bar.xlsx")
You’ll now see a green progress bar that gets fuller the closer the star rating is to the number 5:

Example Spreadsheet With Data Bar Conditional Formatting
As you can see, there are a lot of cool things you can do with conditional formatting.

Here, you saw only a few examples of what you can achieve with it, but check the openpyxl documentation to see a bunch of other options.


Remove ads
Adding Images
Even though images are not something that you’ll often see in a spreadsheet, it’s quite cool to be able to add them. Maybe you can use it for branding purposes or to make spreadsheets more personal.

To be able to load images to a spreadsheet using openpyxl, you’ll have to install Pillow:

$ pip install Pillow
Apart from that, you’ll also need an image. For this example, you can grab the Real Python logo below and convert it from .webp to .png using an online converter such as cloudconvert.com, save the final file as logo.png, and copy it to the root folder where you’re running your examples:

Real Python Logo
Afterward, this is the code you need to import that image into the hello_word.xlsx spreadsheet:

from openpyxl import load_workbook
from openpyxl.drawing.image import Image

# Let's use the hello_world spreadsheet since it has less data
workbook = load_workbook(filename="hello_world.xlsx")
sheet = workbook.active

logo = Image("logo.png")

# A bit of resizing to not fill the whole spreadsheet with the logo
logo.height = 150
logo.width = 150

sheet.add_image(logo, "A3")
workbook.save(filename="hello_world_logo.xlsx")
You have an image on your spreadsheet! Here it is:

Example Spreadsheet With Image
The image’s left top corner is on the cell you chose, in this case, A3.

Adding Pretty Charts
Another powerful thing you can do with spreadsheets is create an incredible variety of charts.

Charts are a great way to visualize and understand loads of data quickly. There are a lot of different chart types: bar chart, pie chart, line chart, and so on. openpyxl has support for a lot of them.

Here, you’ll see only a couple of examples of charts because the theory behind it is the same for every single chart type:

Note: A few of the chart types that openpyxl currently doesn’t have support for are Funnel, Gantt, Pareto, Treemap, Waterfall, Map, and Sunburst.

For any chart you want to build, you’ll need to define the chart type: BarChart, LineChart, and so forth, plus the data to be used for the chart, which is called Reference.

Before you can build your chart, you need to define what data you want to see represented in it. Sometimes, you can use the dataset as is, but other times you need to massage the data a bit to get additional information.

Let’s start by building a new workbook with some sample data:

from openpyxl import Workbook
from openpyxl.chart import BarChart, Reference

workbook = Workbook()
sheet = workbook.active

# Let's create some sample sales data
rows = [
    ["Product", "Online", "Store"],
    [1, 30, 45],
    [2, 40, 30],
    [3, 40, 25],
    [4, 50, 30],
    [5, 30, 25],
    [6, 25, 35],
    [7, 20, 40],
]

for row in rows:
    sheet.append(row)
Now you’re going to start by creating a bar chart that displays the total number of sales per product:

chart = BarChart()
data = Reference(worksheet=sheet,
                 min_row=1,
                 max_row=8,
                 min_col=2,
                 max_col=3)

chart.add_data(data, titles_from_data=True)
sheet.add_chart(chart, "E2")

workbook.save("chart.xlsx")
There you have it. Below, you can see a very straightforward bar chart showing the difference between online product sales online and in-store product sales:

Example Spreadsheet With Bar Chart
Like with images, the top left corner of the chart is on the cell you added the chart to. In your case, it was on cell E2.

Note: Depending on whether you’re using Microsoft Excel or an open-source alternative (LibreOffice or OpenOffice), the chart might look slightly different.

Try creating a line chart instead, changing the data a bit:

import random
from openpyxl import Workbook
from openpyxl.chart import LineChart, Reference

workbook = Workbook()
sheet = workbook.active

# Let's create some sample sales data
rows = [
    ["", "January", "February", "March", "April",
    "May", "June", "July", "August", "September",
     "October", "November", "December"],
    [1, ],
    [2, ],
    [3, ],
]

for row in rows:
    sheet.append(row)

for row in sheet.iter_rows(min_row=2,
                           max_row=4,
                           min_col=2,
                           max_col=13):
    for cell in row:
        cell.value = random.randrange(5, 100)
With the above code, you’ll be able to generate some random data regarding the sales of 3 different products across a whole year.

Once that’s done, you can very easily create a line chart with the following code:

chart = LineChart()
data = Reference(worksheet=sheet,
                 min_row=2,
                 max_row=4,
                 min_col=1,
                 max_col=13)

chart.add_data(data, from_rows=True, titles_from_data=True)
sheet.add_chart(chart, "C6")

workbook.save("line_chart.xlsx")
Here’s the outcome of the above piece of code:

Example Spreadsheet With Line Chart
One thing to keep in mind here is the fact that you’re using from_rows=True when adding the data. This argument makes the chart plot row by row instead of column by column.

In your sample data, you see that each product has a row with 12 values (1 column per month). That’s why you use from_rows. If you don’t pass that argument, by default, the chart tries to plot by column, and you’ll get a month-by-month comparison of sales.

Another difference that has to do with the above argument change is the fact that our Reference now starts from the first column, min_col=1, instead of the second one. This change is needed because the chart now expects the first column to have the titles.

There are a couple of other things you can also change regarding the style of the chart. For example, you can add specific categories to the chart:

cats = Reference(worksheet=sheet,
                 min_row=1,
                 max_row=1,
                 min_col=2,
                 max_col=13)
chart.set_categories(cats)
Add this piece of code before saving the workbook, and you should see the month names appearing instead of numbers:

Example Spreadsheet With Line Chart and Categories
Code-wise, this is a minimal change. But in terms of the readability of the spreadsheet, this makes it much easier for someone to open the spreadsheet and understand the chart straight away.

Another thing you can do to improve the chart readability is to add an axis. You can do it using the attributes x_axis and y_axis:

chart.x_axis.title = "Months"
chart.y_axis.title = "Sales (per unit)"
This will generate a spreadsheet like the below one:

Example Spreadsheet With Line Chart, Categories and Axis Titles
As you can see, small changes like the above make reading your chart a much easier and quicker task.

There is also a way to style your chart by using Excel’s default ChartStyle property. In this case, you have to choose a number between 1 and 48. Depending on your choice, the colors of your chart change as well:

# You can play with this by choosing any number between 1 and 48
chart.style = 24
With the style selected above, all lines have some shade of orange:

Example Spreadsheet With Line Chart, Categories, Axis Titles and Style
There is no clear documentation on what each style number looks like, but this spreadsheet has a few examples of the styles available.


There are a lot more chart types and customization you can apply, so be sure to check out the package documentation on this if you need some specific formatting.


Remove ads
Convert Python Classes to Excel Spreadsheet
You already saw how to convert an Excel spreadsheet’s data into Python classes, but now let’s do the opposite.

Let’s imagine you have a database and are using some Object-Relational Mapping (ORM) to map DB objects into Python classes. Now, you want to export those same objects into a spreadsheet.

Let’s assume the following data classes to represent the data coming from your database regarding product sales:

from dataclasses import dataclass
from typing import List

@dataclass
class Sale:
    quantity: int

@dataclass
class Product:
    id: str
    name: str
    sales: List[Sale]
Now, let’s generate some random data, assuming the above classes are stored in a db_classes.py file:

import random

# Ignore these for now. You'll use them in a sec ;)
from openpyxl import Workbook
from openpyxl.chart import LineChart, Reference

from db_classes import Product, Sale

products = []

# Let's create 5 products
for idx in range(1, 6):
    sales = []

    # Create 5 months of sales
    for _ in range(5):
        sale = Sale(quantity=random.randrange(5, 100))
        sales.append(sale)

    product = Product(id=str(idx),
                      name="Product %s" % idx,
                      sales=sales)
    products.append(product)
By running this piece of code, you should get 5 products with 5 months of sales with a random quantity of sales for each month.

Now, to convert this into a spreadsheet, you need to iterate over the data and append it to the spreadsheet:

workbook = Workbook()
sheet = workbook.active

# Append column names first
sheet.append(["Product ID", "Product Name", "Month 1",
              "Month 2", "Month 3", "Month 4", "Month 5"])

# Append the data
for product in products:
    data = [product.id, product.name]
    for sale in product.sales:
        data.append(sale.quantity)
    sheet.append(data)
That’s it. That should allow you to create a spreadsheet with some data coming from your database.

However, why not use some of that cool knowledge you gained recently to add a chart as well to display that data more visually?

All right, then you could probably do something like this:

chart = LineChart()
data = Reference(worksheet=sheet,
                 min_row=2,
                 max_row=6,
                 min_col=2,
                 max_col=7)

chart.add_data(data, titles_from_data=True, from_rows=True)
sheet.add_chart(chart, "B8")

cats = Reference(worksheet=sheet,
                 min_row=1,
                 max_row=1,
                 min_col=3,
                 max_col=7)
chart.set_categories(cats)

chart.x_axis.title = "Months"
chart.y_axis.title = "Sales (per unit)"

workbook.save(filename="oop_sample.xlsx")
Now we’re talking! Here’s a spreadsheet generated from database objects and with a chart and everything:

Example Spreadsheet With Conversion from Python Data Classes
That’s a great way for you to wrap up your new knowledge of charts!

Bonus: Working With Pandas
Even though you can use Pandas to handle Excel files, there are few things that you either can’t accomplish with Pandas or that you’d be better off just using openpyxl directly.

For example, some of the advantages of using openpyxl are the ability to easily customize your spreadsheet with styles, conditional formatting, and such.

But guess what, you don’t have to worry about picking. In fact, openpyxl has support for both converting data from a Pandas DataFrame into a workbook or the opposite, converting an openpyxl workbook into a Pandas DataFrame.

Note: If you’re new to Pandas, check our course on Pandas DataFrames beforehand.

First things first, remember to install the pandas package:

$ pip install pandas
Then, let’s create a sample DataFrame:

import pandas as pd

data = {
    "Product Name": ["Product 1", "Product 2"],
    "Sales Month 1": [10, 20],
    "Sales Month 2": [5, 35],
}
df = pd.DataFrame(data)
Now that you have some data, you can use .dataframe_to_rows() to convert it from a DataFrame into a worksheet:

from openpyxl import Workbook
from openpyxl.utils.dataframe import dataframe_to_rows

workbook = Workbook()
sheet = workbook.active

for row in dataframe_to_rows(df, index=False, header=True):
    sheet.append(row)

workbook.save("pandas.xlsx")
You should see a spreadsheet that looks like this:

Example Spreadsheet With Data from Pandas Data Frame
If you want to add the DataFrame’s index, you can change index=True, and it adds each row’s index into your spreadsheet.

On the other hand, if you want to convert a spreadsheet into a DataFrame, you can also do it in a very straightforward way like so:

import pandas as pd
from openpyxl import load_workbook

workbook = load_workbook(filename="sample.xlsx")
sheet = workbook.active

values = sheet.values
df = pd.DataFrame(values)
Alternatively, if you want to add the correct headers and use the review ID as the index, for example, then you can also do it like this instead:

import pandas as pd
from openpyxl import load_workbook
from mapping import REVIEW_ID

workbook = load_workbook(filename="sample.xlsx")
sheet = workbook.active

data = sheet.values

# Set the first row as the columns for the DataFrame
cols = next(data)
data = list(data)

# Set the field "review_id" as the indexes for each row
idx = [row[REVIEW_ID] for row in data]

df = pd.DataFrame(data, index=idx, columns=cols)
Using indexes and columns allows you to access data from your DataFrame easily:

>>> df.columns
Index(['marketplace', 'customer_id', 'review_id', 'product_id',
       'product_parent', 'product_title', 'product_category', 'star_rating',
       'helpful_votes', 'total_votes', 'vine', 'verified_purchase',
       'review_headline', 'review_body', 'review_date'],
      dtype='object')

>>> # Get first 10 reviews' star rating
>>> df["star_rating"][:10]
R3O9SGZBVQBV76    5
RKH8BNC3L5DLF     5
R2HLE8WKZSU3NL    2
R31U3UH5AZ42LL    5
R2SV659OUJ945Y    4
RA51CP8TR5A2L     5
RB2Q7DLDN6TH6     5
R2RHFJV0UYBK3Y    1
R2Z6JOQ94LFHEP    5
RX27XIIWY5JPB     4
Name: star_rating, dtype: int64

>>> # Grab review with id "R2EQL1V1L6E0C9", using the index
>>> df.loc["R2EQL1V1L6E0C9"]
marketplace               US
customer_id         15305006
review_id     R2EQL1V1L6E0C9
product_id        B004LURNO6
product_parent     892860326
review_headline   Five Stars
review_body          Love it
review_date       2015-08-31
Name: R2EQL1V1L6E0C9, dtype: object
There you go, whether you want to use openpyxl to prettify your Pandas dataset or use Pandas to do some hardcore algebra, you now know how to switch between both packages.


Remove ads
Conclusion
Phew, after that long read, you now know how to work with spreadsheets in Python! You can rely on openpyxl, your trustworthy companion, to:

Extract valuable information from spreadsheets in a Pythonic manner
Create your own spreadsheets, no matter the complexity level
Add cool features such as conditional formatting or charts to your spreadsheets
There are a few other things you can do with openpyxl that might not have been covered in this tutorial, but you can always check the package’s official documentation website to learn more about it. You can even venture into checking its source code and improving the package further.

Feel free to leave any comments below if you have any questions, or if there’s any section you’d love to hear more about.
/////////////////////////////////


/////////////////////////////////
# 打包输出可执行文件exe

使用如下指令
```
pyinstaller -F Main.py -n AutoTestTool -i AMAutoTest.ico -w
```
`-F` 指定启动脚本源文件
`-n` 指定目标输出exe文件名
`-i` 指定目标输出exe文件的图标文件
`-w` 指定启动exe文件时隐藏终端窗口
/////////////////////////////////


/////////////////////////////////
# Python: 入坑的第一天你会想到什么？

Python 是什么呢？

Python 可以帮你省去大量的繁琐步骤，避免重复无意义的劳动，是一把日常办公的好工具，也是一门低门槛的优秀而现代的编程语言。有句俏皮语：人生苦短，请用Python。

# 一把好工具

大家多是上班族，如果你日常工作都是对着电脑做事的话，那么你一定很需要一些自动化的工具软件，把各种繁琐的重复动作一键自动处理，帮你剩下大量的时间。

比如，搞电商运营的同学喜欢去收集同行的各类商品信息，那么你可以使用 python 写一些脚本，每次需要获取更多信息的时候就点击脚本开始运行，让它给你从指定的网址收集到本地的文档供你分析查阅。

比如，搞新媒体的同学喜欢收集网名的阅读喜好，那么你可以使用 python 写一些脚本，从各个排名靠前的网站爬取一些流量爆文，自己再在其中提取一些自己可以hold得住的内容再创作，这样子一篇新爆文不就轻轻松松有着落了吗？

比如，做文员的同学刚从领导那里领了一份差事，需要重一堆电子文档内容中，提取手机号等等信息，那么你可以使用 python 写一些脚本，让它给你代劳去检阅各个文档内容，然后把需求信息重新输出到本地新文档中，免去购买眼药水，老花眼来得也晚一些。

比如，... （我嫌浪费口水，脑部一下）

所以说，python 真是一把好工具。

*专业来讲，python 语言属于解释型动态类型语言，运行的时候依赖一个叫做解释器的程序对源码执行解读并运行，所以也会把 python 源码文件称呼为 python 脚本文件。*

# 低门槛入门

先不说太多的理由，你看小学生都提倡要学习 python 了，明显它就是低门槛的。

别让自己一直在门外边傻楞着，进来吧！

如果你在大学里或者自学过C语言，应该想到C语言的语法其实不是那么复杂。如果你是专业人士，可以说C语言的语法就是简单到令人发指的地步了。而 python 的语法和C语言何其相似，当然为了表示区别，python 的创始人还是给了它不一样的五官，比如国人和洋人虽然作为人基本都是一样的，不过五官还是具有很好辨识度的。

让我们先来过目一下一个非常简单的 python 程序长什么样子，先新建一个文件，后续名是(.py)。

```
if __name__ == '__main__':
    print('你好呀！')
```

在开发平台的输出窗口里运行，输出是这样子的:
```
你好呀！
```

# 配置环境

做一件事情总有个起始条件，比如我要种花，那么是不是需要准备一个花盘，一堆适合养花的泥土，一把小铲子，一颗种子，一杯水，以及一个有阳光照射的地方等。

先说一下准备 python 开发环境需要准备好哪些东东吧！

普通电脑：个人电脑，一般配置
电脑系统：windows
开发语言支持包：python 安装包
开发环境：VSCODE

准备好条件后，接着就是开始种花的步骤了。。。

## 安装 python 语言支持包

用你最喜欢的浏览器，先是去官网下载安装包
```
https://www.python.org/downloads/
```

安装的过程不需要特别的设定，和普通的安装软件没啥区别，就是一路确定就ok。

*由于这是开发工具包，安装的最后还是要留意一下把安装路径添加到环境变量path中，这个过程无需填写内容，有个勾选框勾上即可，安装包会自动完成。*

这一步做完后，开发的基本环境就已经具备了。可以打开系统 cmd 窗口，输入下面的指令看看 python 的安装版本如何

```
python --version
```

我安装的版本是3.11，如下
```
3.11.0
```

## 配置集成开发环境IDE：VSCODE

// TODO.

## 创建虚拟环境

参考其它博文 // TODO. 链接

## 上代码运行

让我们走一遍最经典的编程入门第一课 hello world !

新建文件main.py，输入下面的内容

```
if __name__ == '__main__':
    print('hello world ！')
```

点击文件右上角的 // TODO. 执行输出时，弹出 terminal 窗口

// TODO. terminal 图片

/////////////////////////////////

Working with Excel Spreadsheets in Python
Last Updated : 12 May, 2021
Read
Discuss
Courses
Practice
Video

You all must have worked with Excel at some time in your life and must have felt the need for automating some repetitive or tedious task. Don’t worry in this tutorial we are going to learn about how to work with Excel using Python, or automating Excel using Python. We will be covering this with the help of the Openpyxl module.

Getting Started
Openpyxl is a Python library that provides various methods to interact with Excel Files using Python. It allows operations like reading, writing, arithmetic operations, plotting graphs, etc.

This module does not come in-built with Python. To install this type the below command in the terminal.

pip install openpyxl
Python Excel tutorial openpyxl install

Reading from Spreadsheets
To read an Excel file you have to open the spreadsheet using the load_workbook() method. After that, you can use the active to select the first sheet available and the cell attribute to select the cell by passing the row and column parameter. The value attribute prints the value of the particular cell. See the below example to get a better understanding. 

Note: The first row or column integer is 1, not 0.


Dataset Used: It can be downloaded from here.

python excel readin excel openpyxl

Example:

# Python program to read an excel file 
  
# import openpyxl module 
import openpyxl 
  
# Give the location of the file 
path = "gfg.xlsx"
  
# To open the workbook 
# workbook object is created 
wb_obj = openpyxl.load_workbook(path) 
  
# Get workbook active sheet object 
# from the active attribute 
sheet_obj = wb_obj.active 
  
# Cell objects also have a row, column, 
# and coordinate attributes that provide 
# location information for the cell. 
  
# Note: The first row or 
# column integer is 1, not 0. 
  
# Cell object is created by using 
# sheet object's cell() method. 
cell_obj = sheet_obj.cell(row = 1, column = 1) 
  
# Print value of cell object 
# using the value attribute 
print(cell_obj.value) 
Output:

Name
Reading from Multiple Cells
There can be two ways of reading from multiple cells. 

Method 1: We can get the count of the total rows and columns using the max_row and max_column respectively. We can use these values inside the for loop to get the value of the desired row or column or any cell depending upon the situation. Let’s see how to get the value of the first column and first row.

Example:

# Python program to read an excel file 
  
# import openpyxl module 
import openpyxl 
  
# Give the location of the file 
path = "gfg.xlsx"
  
# To open the workbook 
# workbook object is created 
wb_obj = openpyxl.load_workbook(path) 
  
# Get workbook active sheet object 
# from the active attribute 
sheet_obj = wb_obj.active 
  
# Getting the value of maximum rows
# and column
row = sheet_obj.max_row
column = sheet_obj.max_column
  
print("Total Rows:", row)
print("Total Columns:", column)
  
# printing the value of first column
# Loop will print all values 
# of first column  
print("\nValue of first column")
for i in range(1, row + 1): 
    cell_obj = sheet_obj.cell(row = i, column = 1) 
    print(cell_obj.value) 
      
# printing the value of first column
# Loop will print all values 
# of first row
print("\nValue of first row")
for i in range(1, column + 1): 
    cell_obj = sheet_obj.cell(row = 2, column = i) 
    print(cell_obj.value, end = " ")
Output:

Total Rows: 6
Total Columns: 4

Value of first column
Name
Ankit
Rahul
Priya
Nikhil
Nisha

Value of first row
Ankit  B.Tech CSE 4 
Method 2: We can also read from multiple cells using the cell name. This can be seen as the list slicing of Python.

# Python program to read an excel file 
  
# import openpyxl module 
import openpyxl 
  
# Give the location of the file 
path = "gfg.xlsx"
  
# To open the workbook 
# workbook object is created 
wb_obj = openpyxl.load_workbook(path) 
  
# Get workbook active sheet object 
# from the active attribute 
sheet_obj = wb_obj.active 
  
# Cell object is created by using 
# sheet object's cell() method. 
cell_obj = sheet_obj['A1': 'B6']
  
# Print value of cell object 
# using the value attribute 
for cell1, cell2 in cell_obj:
    print(cell1.value, cell2.value)
Output:

Name Course
Ankit  B.Tech
Rahul M.Tech
Priya MBA
Nikhil B.Tech
Nisha B.Tech
Refer to the below article to get detailed information about reading excel files using openpyxl.

Reading an excel file using Python openpyxl module


Writing to Spreadsheets
First, let’s create a new spreadsheet, and then we will write some data to the newly created file. An empty spreadsheet can be created using the Workbook() method. Let’s see the below example.

Example:

from openpyxl import Workbook
  
# Call a Workbook() function of openpyxl  
# to create a new blank Workbook object 
workbook = Workbook()
  
# Anytime you modify the Workbook object 
# or its sheets and cells, the spreadsheet 
# file will not be saved until you call 
# the save() workbook method. 
workbook.save(filename="sample.xlsx")
Output:

empty spreadsheet using Python

After creating an empty file, let’s see how to add some data to it using Python. To add data first we need to select the active sheet and then using the cell() method we can select any particular cell by passing the row and column number as its parameter. We can also write using cell names. See the below example for a better understanding.

Example:

# import openpyxl module 
import openpyxl 
  
# Call a Workbook() function of openpyxl 
# to create a new blank Workbook object 
wb = openpyxl.Workbook() 
  
# Get workbook active sheet 
# from the active attribute 
sheet = wb.active 
  
# Cell objects also have row, column 
# and coordinate attributes that provide 
# location information for the cell. 
  
# Note: The first row or column integer 
# is 1, not 0. Cell object is created by 
# using sheet object's cell() method. 
c1 = sheet.cell(row = 1, column = 1) 
  
# writing values to cells 
c1.value = "Hello"
  
c2 = sheet.cell(row= 1 , column = 2) 
c2.value = "World"
  
# Once have a Worksheet object, one can 
# access a cell object by its name also. 
# A2 means column = 1 & row = 2. 
c3 = sheet['A2'] 
c3.value = "Welcome"
  
# B2 means column = 2 & row = 2. 
c4 = sheet['B2'] 
c4.value = "Everyone"
  
# Anytime you modify the Workbook object 
# or its sheets and cells, the spreadsheet 
# file will not be saved until you call 
# the save() workbook method. 
wb.save("sample.xlsx") 
Output:

python excel writing to file

Refer to the below article to get detailed information about writing to excel.

Writing to an excel file using openpyxl module
Appending to the Spreadsheet
In the above example, you will see that every time you try to write to a spreadsheet the existing data gets overwritten, and the file is saved as a new file. This happens because the Workbook() method always creates a new workbook file object. To write to an existing workbook you must open the file with the load_workbook() method. We will use the above-created workbook.

Example:

# import openpyxl module 
import openpyxl 
  
wb = openpyxl.load_workbook("sample.xlsx") 
  
sheet = wb.active 
  
c = sheet['A3'] 
c.value = "New Data"
  
wb.save("sample.xlsx")
Output:

append data excel python

We can also use the append() method to append multiple data at the end of the sheet.

Example:

# import openpyxl module 
import openpyxl 
  
wb = openpyxl.load_workbook("sample.xlsx") 
  
sheet = wb.active 
  
data = (
    (1, 2, 3),
    (4, 5, 6)
)
  
for row in data:
    sheet.append(row)
  
wb.save('sample.xlsx')
Output:

append data excel python

Arithmetic Operation on Spreadsheet
Arithmetic operations can be performed by typing the formula in a particular cell of the spreadsheet. For example, if we want to find the sum then =Sum() formula of the excel file is used.

Example:

# import openpyxl module 
import openpyxl 
  
# Call a Workbook() function of openpyxl 
# to create a new blank Workbook object 
wb = openpyxl.Workbook() 
  
# Get workbook active sheet 
# from the active attribute. 
sheet = wb.active 
  
# writing to the cell of an excel sheet 
sheet['A1'] = 200
sheet['A2'] = 300
sheet['A3'] = 400
sheet['A4'] = 500
sheet['A5'] = 600
  
# The value in cell A7 is set to a formula 
# that sums the values in A1, A2, A3, A4, A5 . 
sheet['A7'] = '= SUM(A1:A5)'
  
# save the file 
wb.save("sum.xlsx") 
Output:

finding sum excel python

Refer to the below article to get detailed information about the Arithmetic operations on Spreadsheet.

Arithmetic operations in excel file using openpyxl
Adjusting Rows and Column
Worksheet objects have row_dimensions and column_dimensions attributes that control row heights and column widths. A sheet’s row_dimensions and column_dimensions are dictionary-like values; row_dimensions contains RowDimension objects and column_dimensions contains ColumnDimension objects. In row_dimensions, one can access one of the objects using the number of the row (in this case, 1 or 2). In column_dimensions, one can access one of the objects using the letter of the column (in this case, A or B).

Example:

# import openpyxl module 
import openpyxl 
  
# Call a Workbook() function of openpyxl 
# to create a new blank Workbook object 
wb = openpyxl.Workbook() 
  
# Get workbook active sheet 
# from the active attribute. 
sheet = wb.active 
  
# writing to the specified cell 
sheet.cell(row = 1, column = 1).value = ' hello '
  
sheet.cell(row = 2, column = 2).value = ' everyone '
  
# set the height of the row 
sheet.row_dimensions[1].height = 70
  
# set the width of the column 
sheet.column_dimensions['B'].width = 20
  
# save the file 
wb.save('sample.xlsx') 
Output:

adjusting rows and columns excel python

Merging Cells
A rectangular area of cells can be merged into a single cell with the merge_cells() sheet method. The argument to merge_cells() is a single string of the top-left and bottom-right cells of the rectangular area to be merged.

Example:

import openpyxl 
wb = openpyxl.Workbook() 
sheet = wb.active 
  
# merge cell from A2 to D4 i.e. 
# A2, B2, C2, D2, A3, B3, C3, D3, A4, B4, C4 and D4 . 
# A2:D4' merges 12 cells into a single cell. 
sheet.merge_cells('A2:D4') 
  
sheet.cell(row = 2, column = 1).value = 'Twelve cells join together.'
  
# merge cell C6 and D6 
sheet.merge_cells('C6:D6') 
  
sheet.cell(row = 6, column = 6).value = 'Two merge cells.'
  
wb.save('sample.xlsx')
Output:

merge cells excel python

Unmerging Cells
To unmerge cells, call the unmerge_cells() sheet method.

Example:

import openpyxl 
  
  
wb = openpyxl.load_workbook('sample.xlsx') 
sheet = wb.active 
  
# unmerge the cells 
sheet.unmerge_cells('A2:D4') 
  
sheet.unmerge_cells('C6:D6') 
  
wb.save('sample.xlsx')
Output:

unmerge cells excel python

Setting Font Style
To customize font styles in cells, important, import the Font() function from the openpyxl.styles module.

Example:

import openpyxl 
  
# import Font function from openpyxl 
from openpyxl.styles import Font 
  
  
wb = openpyxl.Workbook() 
sheet = wb.active 
  
sheet.cell(row = 1, column = 1).value = "GeeksforGeeks"
  
# set the size of the cell to 24 
sheet.cell(row = 1, column = 1).font = Font(size = 24 ) 
  
sheet.cell(row = 2, column = 2).value = "GeeksforGeeks"
  
# set the font style to italic 
sheet.cell(row = 2, column = 2).font = Font(size = 24, italic = True) 
  
sheet.cell(row = 3, column = 3).value = "GeeksforGeeks"
  
# set the font style to bold 
sheet.cell(row = 3, column = 3).font = Font(size = 24, bold = True) 
  
sheet.cell(row = 4, column = 4).value = "GeeksforGeeks"
  
# set the font name to 'Times New Roman' 
sheet.cell(row = 4, column = 4).font = Font(size = 24, name = 'Times New Roman') 
  
wb.save('sample.xlsx') 
Output:

setting style excel python

Refer to the below article to get detailed information about adjusting rows and columns.

Adjusting rows and columns of an excel file using openpyxl module
Plotting Charts
Charts are composed of at least one series of one or more data points. Series themselves are comprised of references to cell ranges. For plotting the charts on an excel sheet, firstly, create chart objects of specific chart class( i.e BarChart, LineChart, etc.). After creating chart objects, insert data in it, and lastly, add that chart object in the sheet object.

Example 1:

# import openpyxl module
import openpyxl
  
# import BarChart class from openpyxl.chart sub_module
from openpyxl.chart import BarChart, Reference
  
# Call a Workbook() function of openpyxl
# to create a new blank Workbook object
wb = openpyxl.Workbook()
  
# Get workbook active sheet
# from the active attribute.
sheet = wb.active
  
# write o to 9 in 1st column of the active sheet
for i in range(10):
    sheet.append([i])
  
# create data for plotting
values = Reference(sheet, min_col=1, min_row=1,
                   max_col=1, max_row=10)
  
# Create object of BarChart class
chart = BarChart()
  
# adding data to the Bar chart object
chart.add_data(values)
  
# set the title of the chart
chart.title = " BAR-CHART "
  
# set the title of the x-axis
chart.x_axis.title = " X_AXIS "
  
# set the title of the y-axis
chart.y_axis.title = " Y_AXIS "
  
# add chart to the sheet
# the top-left corner of a chart
# is anchored to cell E2 .
sheet.add_chart(chart, "E2")
  
# save the file
wb.save("sample.xlsx")
Output:

create chart excel python

Example 2:

# import openpyxl module
import openpyxl
  
# import LineChart class from openpyxl.chart sub_module
from openpyxl.chart import LineChart, Reference
  
wb = openpyxl.Workbook()
sheet = wb.active
  
# write o to 9 in 1st column of the active sheet
for i in range(10):
    sheet.append([i])
  
values = Reference(sheet, min_col=1, min_row=1,
                   max_col=1, max_row=10)
  
# Create object of LineChart class
chart = LineChart()
  
chart.add_data(values)
  
# set the title of the chart
chart.title = " LINE-CHART "
  
# set the title of the x-axis
chart.x_axis.title = " X-AXIS "
  
# set the title of the y-axis
chart.y_axis.title = " Y-AXIS "
  
# add chart to the sheet
# the top-left corner of a chart
# is anchored to cell E2 .
sheet.add_chart(chart, "E2")
  
# save the file
wb.save("sample.xlsx")
Output:

create chart excel python 2

Refer to the below articles to get detailed information about plotting in excel using Python.

Plotting charts in excel sheet using openpyxl module | Set  1
Plotting charts in excel sheet using openpyxl module | Set  2
Plotting charts in excel sheet using openpyxl module | Set 3
Adding Images
For the purpose of importing images inside our worksheet, we would be using openpyxl.drawing.image.Image. The method is a wrapper over PIL.Image method found in PIL (pillow) library. Due to which it is necessary for the PIL (pillow) library to be installed in order to use this method.

Image Used:



Example:

import openpyxl 
from openpyxl.drawing.image import Image
  
wb = openpyxl.Workbook() 
  
sheet = wb.active
  
# Adding a row of data to the worksheet (used to 
# distinguish previous excel data from the image) 
sheet.append([10, 2010, "Geeks", 4, "life"]) 
  
# A wrapper over PIL.Image, used to provide image 
# inclusion properties to openpyxl library 
img = Image("geek.jpg")
  
# Adding the image to the worksheet 
# (with attributes like position) 
sheet.add_image(img, 'A2') 
  
# Saving the workbook created
wb.save('sample.xlsx')
Output:

add image excel python

Refer to the below article to get detailed information about adding images.

Openpyxl – Adding Image







