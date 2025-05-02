---
title: "Complete JAVA PART 1"
datePublished: Tue Jan 21 2025 10:20:36 GMT+0000 (Coordinated Universal Time)
cuid: cm66br02j000909jmejncegll
slug: complete-java-part-1
tags: java

---

This is **Part 1** of a comprehensive Java series where we’ll journey from the very basics to an advanced level, culminating in the ability to develop robust web applications using the Java stack, including **Spring Boot**. The primary motivation behind this series is my desire to share knowledge while simultaneously enhancing my own understanding.

In this series, I won’t just focus on the **"what"** and **"how"** of Java concepts; I’ll dive deeper into the **"why"**—the reasoning behind their usage and importance. My goal is to provide you with a solid foundation, practical insights, and a clear understanding of Java development, making your learning experience engaging and meaningful.

So without wasting any time let’s get started. In this part we will cover the core JAVA concepts. But to mention one thing that i will not be giving to much details on the basic programming concepts like lops , control statements and syntax , more focus will be on Java specific concepts like **“Collection Framework”**

# Introduction to JAVA

### What is JAVA ?

**Java** is a high-level, object-oriented programming language developed by **Sun Microsystems** in 1995 (later acquired by **Oracle**). It is widely used for building applications ranging from simple desktop programs to complex enterprise solutions. Its **"write once, run anywhere" (WORA)** capability makes it a preferred choice for developers across the globe.

### How JAVA works ?

Java follows a simple and structured workflow that ensures platform independence and efficient execution

1. You write Java code in a `.java` file using a text editor or an IDE (like IntelliJ IDEA or Eclipse)
    
2. The **Java Compiler (**`javac`) converts the human-readable Java code into **bytecode**, which is stored in a `.class` file.
    
    * **Bytecode**: A platform-independent, intermediate code that can be run on any system with a Java Virtual Machine (JVM).
        
    * Example: Compiling [`HelloWorld.java`](http://HelloWorld.java) produces `HelloWorld.class`.
        
    * ```bash
          javac HelloWorld.java
        ```
        
3. The **Class Loader** in the JVM loads the `.class` file into memory and checks it for validity and security.
    
4. The **Java Virtual Machine (JVM)** interprets the bytecode and executes it on the underlying hardware.
    
    * JVM converts the bytecode into **machine-specific instructions** using either:
        
        * **Interpreter**: Converts bytecode to machine code line-by-line. Slower but simpler. (This makes debugging in java very easy and powerful as the code is executed line by line).
            
        * **Just-In-Time (JIT) Compiler**: Compiles the entire bytecode into native machine code for faster execution.
            

## JVM , JRE and JDK

### JVM (JAVA VIRTUAL MACHINE)

JVM is an abstract machine that provides the runtime environment to execute Java bytecode.  
It executes the compiled Java bytecode and converts it to machine-specific instructions. It includes the class loader, bytecode verifier, and execution engine (interpreter/JIT compiler) . It comes as part of JDK or JRE

### JRE (JAVA RUNTIME ENVIRONMENT)

JRE is the software package which includes the JVM and other standard java libraries which are required to run a java application. This is the software (environment) which even the end-user will need to run the java application.

### JDK ( JAVA DEVELOPMENT KIT)

The **Java Development Kit (JDK)** is a complete software development environment used for developing, compiling, debugging, and running Java applications. It provides all the tools necessary for Java developers to write and test their programs. It comes with the JRE and other development tools like java compiler (javac) , javadoc , jar etc.

# Understanding the Java Basics

Java follows a structured and object oriented approach.

```java

import java.util.Scanner; //(for importing the package and that Class used)

// Class Declaration
public class MyProgram {

    // Main method: Entry point of the program
    public static void main(String[] args) {
        
        // Statements: Instructions to be executed
        System.out.println("Hello, World!");
    }
}
```

Since mostly everything is Class in java program so the main entry point to a JAVA program is the **main method.**

It is a static method which is the starting point of any program. If this method is not present than the class will be compiled as it should be but when you will try to run that compiled byte code a error will be thrown by **JVM** `Error: Main method not found in class ClassName, please define the main method as: public static void main(String[] args)` . Hence it is clear that main method is required to tell JVM that bro from here you have to enter.

### **Classes and Objects**

In JAVA everything resides in the Class and the properties,fields,methods defined in that class can be accessed by the Object of that class.

You can simply understand it by an example that A Car Model (class) ( let’s say Honda Amaze) is defined by the company (programmer) and then the cars of that model following the properties defined in that model ( Objects) distributed to consumers.

---

### Variables

It is used to store the data

```java
data_type variable_name = value;
```

---

### Data types

In Java, **data types** define the type of data a variable can hold. Java is a **strongly-typed** language, meaning variables must be declared with a specific type, and you cannot assign a value of one type to a variable of another type without explicit casting.

Java data types are broadly categorized into two categories:

1. **Primitive Data Types**
    
2. **Reference Data Types**
    

### 1\. **Primitive Data Types**

These are the most basic data types in Java. They represent simple values and are predefined by the language. There are **8 primitive data types** in Java:

| Data Type | Size | Default Value | Description |
| --- | --- | --- | --- |
| `byte` | 1 byte | 0 | Represents integer values from -128 to 127. |
| `short` | 2 bytes | 0 | Represents integer values from -32,768 to 32,767. |
| `int` | 4 bytes | 0 | Represents integer values from -2^31 to 2^31-1. |
| `long` | 8 bytes | 0L | Represents integer values from -2^63 to 2^63-1. |
| `float` | 4 bytes | 0.0f | Represents decimal values (single precision). |
| `double` | 8 bytes | 0.0d | Represents decimal values (double precision). |
| `char` | 2 bytes | '\\u0000' | Represents a single 16-bit Unicode character. |
| `boolean` | 1 byte | `false` | Represents either `true` or `false`. |

#### **Detailed Explanation of Primitive Data Types**

1. `byte`:
    
    * Used for saving memory in large arrays when memory space is a concern.
        
    * Size: 1 byte (8 bits).
        
    * Can store values from -128 to 127.
        
2. `short`:
    
    * A larger integer than `byte`, but smaller than `int`.
        
    * Size: 2 bytes (16 bits).
        
    * Can store values from -32,768 to 32,767.
        
3. `int`:
    
    * The most commonly used integer data type.
        
    * Size: 4 bytes (32 bits).
        
    * Can store values from -2,147,483,648 to 2,147,483,647.
        
4. `long`:
    
    * Used when you need a larger range of integers than `int`.
        
    * Size: 8 bytes (64 bits).
        
    * Can store values from -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807.
        
5. `float`:
    
    * Used for floating-point (decimal) numbers.
        
    * Size: 4 bytes (32 bits).
        
    * Single precision.
        
    * Example: `3.14f`.
        
6. `double`:
    
    * Also used for floating-point numbers, but with double precision.
        
    * Size: 8 bytes (64 bits).
        
    * More accurate for representing decimal values.
        
    * Example: `3.14159`.
        
7. `char`:
    
    * Represents a single 16-bit Unicode character.
        
    * Size: 2 bytes.
        
    * Can store a single character, like `'A'`, `'1'`, or `'%'`.
        
8. `boolean`:
    
    * Represents a true or false value.
        
    * Size: 1 bit (but typically stored as 1 byte).
        
    * Used for conditions and flags.
        

### 2\. **Reference Data Types**

Reference data types are used to store the memory address (reference) of an object. These types do not store the actual data, but instead point to an object in memory.

#### **Common Reference Data Types**

1. `String`:
    
    * Represents a sequence of characters.
        
    * Strings are objects in Java (not primitive types).
        
    * Example: `String greeting = "Hello, World!";`
        
2. **Arrays**:
    
    * A collection of elements of the same data type.
        
    * Can be of any primitive or reference data type.
        
    * Example: `int[] numbers = {1, 2, 3};`
        
3. **Objects**:
    
    * An instance of a class, and objects can have attributes (fields) and behaviors (methods).
        
    * Objects are reference types, so they store the memory address of the instance.
        
    * Example:
        
        ```java
         class Person {
            String name;
            int age;
        }
        Person person1 = new Person();
        ```
        

## Some concepts related to Data types in JAVA

### **Wrapper Classes**

In Java, **wrapper classes** are classes that provide a way to use primitive data types as objects. They are part of the **java.lang** package, and they allow primitives to be treated as objects, which is useful in situations where object manipulation is needed, such as in collections or working with generic types.

Each **primitive data type** in Java has a corresponding **wrapper class**. These wrapper classes provide methods to convert between types, perform operations, and support various utilities for primitive types.

### **Primitive Data Types and Their Wrapper Classes**

| Primitive Type | Wrapper Class | Example Use Case |
| --- | --- | --- |
| `byte` | `Byte` | Used in collections or utilities |
| `short` | `Short` | Useful when performing calculations or comparisons |
| `int` | `Integer` | Commonly used with collections like `ArrayList<Integer>` |
| `long` | `Long` | Can be used to handle large numbers beyond `int` range |
| `float` | `Float` | Used when dealing with decimal numbers in collections |
| `double` | `Double` | Supports higher precision for floating-point values |
| `char` | `Character` | Useful when working with characters as objects |
| `boolean` | `Boolean` | Used in collections or methods requiring an object |

## AutoBoxing and Unboxing

Java automatically converts between primitive types and their corresponding wrapper classes when needed. This is called **autoboxing** (primitive to object) and **unboxing** (object to primitive).

```java
int num = 10; // primitive
Integer numObj = num; // autoboxing (primitive to object)
int numAgain = numObj; // unboxing (object to primitive)
```

## Type Casting

Type casting refers to the conversion of one data type to another. There are two types:

#### **1\. Implicit Casting (Widening)**

* Java automatically converts smaller data types to larger ones.
    
* Example: `int` to `long` or `float` to `double`.
    
    ```java
    javaCopyEditint num = 10;
    double result = num; // Implicit casting
    ```
    

#### **2\. Explicit Casting (Narrowing)**

* You need to manually convert larger data types to smaller ones. This might lead to **loss of precision**.
    
* Example: `double` to `int`.
    
    ```java
    javaCopyEditdouble value = 9.99;
    int intValue = (int) value; // Explicit casting
    ```
    

---

### **Default Values of Data Types**

When variables are declared but not initialized, they have default values based on their data type. For example:

* `int` default value: `0`
    
* `boolean` default value: `false`
    
* `char` default value: `\u0000` (null character)
    
* `Object` (reference type) default value: `null`
    

---

## Control Flow Statements

1. ### Conditional Statements
    
    1\. `if`, `else` ,`else if`
    
    ```java
         if (Condition) {
            // statements
            }
    else if (Condition 2){
    // Statements
    }
     else {
       //Statements
    }
    ```
    
2. `switch`
    
    ```java
    switch(expression) {
      case x:
        // code block
        break;
      case y:
        // code block
        break;
      default:
        // code block
    }
    ```
    

### 2\. Loops

`for`

```java
for (statement 1; statement 2; statement 3) {
  // code block to be executed
}
```

`while`

```java
while (condition) {
  // code block to be executed
}
```

`do-while`

```java
do {
  // code block to be executed
}
while (condition);
```

`for-each`

```java
for (type variableName : arrayName) {
  // code block to be executed
}
```

---

## Console/Terminal Input and Output

In Java, handling input and output (I/O) is done through various classes that belong to the [`java.io`](http://java.io) package. The two main forms of I/O are:

1. **Input**: Getting data from the user or a source like a file.
    
2. **Output**: Displaying data to the console or writing it to a file.
    

#### **1\. Input in Java**

Java provides different ways to read input, including using the `Scanner` class, `BufferedReader`, and `Console` class.

##### **a) Using Scanner Class**

The `Scanner` class is the most commonly used for reading input from the user. It can read different types of data such as strings, integers, and doubles.

**Example:**

```java
import java.util.Scanner;

public class ScannerExample {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        
        // Reading an integer
        System.out.print("Enter an integer: ");
        int num = scanner.nextInt();
        
        // Reading a string
        System.out.print("Enter a string: ");
        scanner.nextLine();  // to consume the newline left by nextInt
        String str = scanner.nextLine();
        
        // Reading a double
        System.out.print("Enter a decimal number: ");
        double dec = scanner.nextDouble();
        
        System.out.println("You entered: " + num + ", " + str + ", " + dec);
        
        scanner.close(); // Always close the scanner after use
    }
}
```

**Explanation:**

* `nextInt()`: Reads an integer.
    
* `nextLine()`: Reads a full line of text (including spaces).
    
* `nextDouble()`: Reads a decimal number.
    

**Output Example:**

```java
Enter an integer: 25
Enter a string: Hello World
Enter a decimal number: 3.14
You entered: 25, Hello World, 3.14
```

##### **b) Using BufferedReader**

The `BufferedReader` class provides a more efficient way to read input, especially when reading large chunks of data. It is typically used with the `InputStreamReader` for reading from the console.

**Example:**

```java
import java.io.*;

public class BufferedReaderExample {
    public static void main(String[] args) throws IOException {
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        
        // Reading a line of text
        System.out.print("Enter a string: ");
        String input = reader.readLine();
        
        System.out.println("You entered: " + input);
        
        reader.close();  // Always close the reader after use
    }
}
```

**Explanation:**

* `readLine()`: Reads a full line of input.
    
* You need to handle `IOException`, which is why `throws IOException` is used in the method signature.
    

#### **2\. Output in Java**

Java provides various ways to display output, including using `System.out.println()`, `System.out.print()`, and `System.out.printf()`.

##### **a) Using** `System.out.println()`

The `println()` method prints text followed by a newline character, meaning the next output will be on the next line.

**Example:**

```java
public class OutputExample {
    public static void main(String[] args) {
        System.out.println("Hello, World!");  // Prints a line followed by a newline
        System.out.println("Welcome to Java.");
    }
}
```

**Output:**

```java
Hello, World!
Welcome to Java.
```

##### **b) Using** `System.out.print()`

The `print()` method prints text without adding a newline at the end. This allows multiple outputs to appear on the same line.

**Example:**

```java
public class OutputExample {
    public static void main(String[] args) {
        System.out.print("Hello, ");
        System.out.print("World!");
    }
}
```

**Output:**

```java
Hello, World!
```

##### **c) Using** `System.out.printf()`

The `printf()` method allows you to format the output using placeholders, similar to C's `printf()`.

**Example:**

```java
public class OutputExample {
    public static void main(String[] args) {
        int num = 10;
        double price = 23.45;
        
        // Printing formatted output
        System.out.printf("The number is: %d\n", num);
        System.out.printf("The price is: %.2f\n", price);
    }
}
```

**Output:**

```java
The number is: 10
The price is: 23.45
```

**Explanation:**

* `%d`: Formats an integer.
    
* `%.2f`: Formats a floating-point number to 2 decimal places.
    

---

## Methods and Functions in JAVA

In Java, methods (or functions, as they are often called in other languages) are blocks of code designed to perform a particular task. They help in reusability, better organization, and abstraction in programming. Methods in Java are defined inside a class and are invoked to perform a specific operation.

#### **1\. What is a Method?**

A method in Java is a collection of statements that are grouped together to perform a specific task. Methods allow you to break down a problem into smaller, more manageable tasks, making your code easier to read, reuse, and maintain

```java
returnType methodName(parameters) {
    // method body
    // statements to perform the task
}
```

* **returnType**: Specifies the type of value the method will return (e.g., `int`, `String`, `void` if no value is returned).
    
* **methodName**: The name of the method. It should follow the naming conventions (lowercase first letter, descriptive name).
    
* **parameters**: The values that are passed to the method. These are optional. A method can have no parameters.
    
* **method body**: The block of code that contains the instructions to be executed.
    

#### **2\. Types of Methods**

Java methods can be broadly classified into two categories:

* **Instance Methods**: Belong to an instance of a class. These methods can access instance variables and other instance methods of the class.
    
* **Static Methods**: Belong to the class itself, rather than an instance. Static methods can only access static variables and methods.
    

#### **3\. Return Types**

A method can return different types of values:

* **void**: When the method does not return any value.
    
* **Primitive data types**: (e.g., `int`, `char`, `boolean`) when the method returns a specific value.
    
* **Object references**: When the method returns a reference to an object (e.g., `String`, `Student`).
    

#### **4\. Method Overloading**

Method overloading occurs when two or more methods in the same class have the same name but different parameter lists. The return type can be the same or different, but the method signature must differ.

**Example of method overloading:**

```java
public class OverloadExample {

    // Method to add two integers
    public int add(int a, int b) {
        return a + b;
    }

    // Overloaded method to add three integers
    public int add(int a, int b, int c) {
        return a + b + c;
    }

    public static void main(String[] args) {
        OverloadExample obj = new OverloadExample();
        
        System.out.println("Sum of two numbers: " + obj.add(5, 10));
        System.out.println("Sum of three numbers: " + obj.add(5, 10, 15));
    }
}
```

The `add()` method is overloaded, one with two parameters and the other with three.

#### **5\. Method Parameters**

* **Formal Parameters**: Parameters specified in the method definition.
    
* **Actual Parameters**: The arguments passed to the method when called.
    
    ```java
    public class ParameterExample {
    
        // Method with parameters
        public void greetUser(String name, int age) {
            System.out.println("Hello " + name + ", you are " + age + " years old.");
        }
    
        public static void main(String[] args) {
            ParameterExample obj = new ParameterExample();
            
            // Passing actual parameters when calling the method
            obj.greetUser("John", 25);
        }
    }
    ```
    
    #### **6\. Method Invocation**
    
    Methods can be invoked in two primary ways:
    
    * **Within the same class**: If the method is in the same class, you can call it directly by its name (if it's in the same method scope).
        
    * **From another class**: If the method is in another class, you need to create an object (for instance methods) or call it using the class name (for static methods).
        
    
    #### **7\. Varargs (Variable Arguments)**
    
    Java allows methods to accept a variable number of arguments using varargs. You can pass any number of arguments to a method by specifying `...` (three dots) before the parameter type.
    
* ```java
    public class VarargsExample {
    
        // Method with varargs
        public void printNumbers(int... numbers) {
            for (int num : numbers) {
                System.out.println(num);
            }
        }
    
        public static void main(String[] args) {
            VarargsExample obj = new VarargsExample();
            
            // Passing different numbers of arguments
            obj.printNumbers(1, 2, 3);
            obj.printNumbers(10, 20, 30, 40, 50);
        }
    }
    ```
    
    The method `printNumbers()` accepts a variable number of integer arguments, and the `for-each` loop is used to print each number.
    

This is it for this part in the next part we will be discussing about Strings in java.