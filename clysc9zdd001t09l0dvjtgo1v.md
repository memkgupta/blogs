---
title: "How I implemented Class To Table Mapping from scratch  using Reflections and Annotations in JAVA ?"
seoTitle: "Implementing ORM functionalities in Java from scratch"
seoDescription: "In this blog post, I will be showing you how i implemented ORM functionalities in java from scratch using custom annotations and reflections"
datePublished: Fri Jul 19 2024 06:48:39 GMT+0000 (Coordinated Universal Time)
cuid: clysc9zdd001t09l0dvjtgo1v
slug: how-i-implemented-class-to-table-mapping-from-scratch-using-reflections-and-annotations-in-java
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1721373258150/1a174a9d-bf0f-4ad6-806b-681760ac0c0e.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1721371458051/295f5c73-4394-48da-94ca-1e8ede1cfeec.png

---

In this article i will be showing and explaining you how i implemented Class to Table Mapping in java using Reflections.

### What is Reflections in JAVA ?

In Java, reflection is a feature that allows a program to inspect and manipulate the internal properties of classes, interfaces, fields, and methods at runtime. This powerful tool is part of the `java.lang.reflect` package and provides the ability to perform operations such as:

1. **Inspecting Classes, Methods, and Fields**: Reflection allows you to examine the structure of classes, methods, and fields, including their names, modifiers (e.g., public, private), types, and annotations.
    
2. **Instantiating Objects**: You can create instances of classes dynamically at runtime using reflection, even if the class names are not known at compile time.
    
3. **Invoking Methods**: Reflection enables you to invoke methods on objects dynamically, bypassing compile-time checks.
    
4. **Accessing Fields**: You can get and set the values of fields in objects, even if the fields are private.
    
5. **Accessing Constructors**: Reflection allows you to access and invoke constructors.
    

### Annotations

Annotations in Java are a form of metadata that provide data about a program but are not part of the program itself. They have no direct effect on the operation of the code they annotate. Annotations can be used for a variety of purposes, such as providing information to the compiler, runtime processing, or to generate code, XML files, and other resources.

**We will be using annotations to mark the classes which are to mapped to a table.**

Before exploring how to define our own custom annotations first let's understand some predefined annotations for defining annotations

1\. `@Retention`

It is used to specify for how long the defined annotation will be retained.

It takes a single parameter from the `java.lang.annotation.RetentionPolicy` enumeration, which includes:

* `RetentionPolicy.SOURCE`: The annotation is retained only in the source code and is discarded during compilation.
    
* `RetentionPolicy.CLASS`: The annotation is retained in the compiled class file but is not available at runtime. This is the default retention policy.
    
* `RetentionPolicy.RUNTIME`: The annotation is retained at runtime, so it can be read reflectively. (Will be using this for accessing classes using reflections )
    

2. `@Target`
    
    It indicates the kinds of program elements to which an annotation type is applicable. It takes an array of constants from the `java.lang.annotation.ElementType` enumeration, including:
    
    * `ElementType.TYPE`: Can be applied to any element of a class (e.g., class, interface, enum).
        
    * `ElementType.FIELD`: Can be applied to fields or properties.
        
    * `ElementType.METHOD`: Can be applied to methods.
        
    * `ElementType.PARAMETER`: Can be applied to method parameters.
        
    * `ElementType.CONSTRUCTOR`: Can be applied to constructors.
        
    * `ElementType.LOCAL_VARIABLE`: Can be applied to local variables.
        
    * `ElementType.ANNOTATION_TYPE`: Can be applied to annotation types.
        
    * `ElementType.PACKAGE`: Can be applied to package declarations.
        
    * `ElementType.TYPE_PARAMETER`: Can be applied to type parameters (since Java 8).
        
    * `ElementType.TYPE_USE`: Can be applied to any use of a type (since Java 8). It basically means that on every usecase of a class i.e Either implementation, typecasting or type-declaration.We can also use this for annotating while creating an instance of a class , i.e all type of use of classes( TYPE)
        

## **Defining custom annotations**

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MyCustomAnnotation {
    String value();
    int number() default 0;
}
```

We can add any number of properties in this annotation and can also specify the default value.

### Using Custom Annotations

```java
public class MyClass {
    @MyCustomAnnotation(value = "Example", number = 10)
    public void myMethod() {
        System.out.println("My method");
    }
}
```

I hope from this you had got the idea what is annotation . Now we will be discussing the **application of Reflection in java and our usecase**.

So we will be using reflections to get all the classes with particular annotation and will get the fields or variables of that class which are to be added into table and then generating a SQL query dynamically and executing it.

Let's begin with creating our `DbConnection` class which will return the Singleton object of the connection to the database , so that there are not many connections made.

```java
package org.game.database;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.util.Arrays;

public class Connector {
    private static Connection connection;
    private static final String URL = "jdbc:postgresql://localhost:5432/game";
    private static final String USERNAME = "username";
    private static final String PASSWORD = "password";
    public static Connection connect(){
if(connection!=null){
    return connection;
}
else{
    try {
        connection = DriverManager.getConnection(URL,USERNAME,PASSWORD);

    }
    catch (SQLException e){
        System.out.println(e.getMessage()+"\n");
         e.printStackTrace();
         System.exit(1);
    }
    return connection;
}
    }
}
```

And we will also need our custom annotations like this

**For the column of a table**

```java
package org.game.annotations;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface Column {
    String name() default "" ;
    String type() default "";
    boolean primaryKey() default  false;
}
```

**For the table**

```java
package org.game.annotations;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Table {
String name();
}
```

Now after the annotations the most important thing is our tables so let's create a class which will be mapped to the table

```java
package org.game.entities.Characters;

import org.game.annotations.Column;
import org.game.annotations.Table;

@Table(name = "_character")
public class Character {
    @Column(name = "id" , type = "INT" , primaryKey = true)
    int id;
    @Column(name="name" , type = "VARCHAR(100)",primaryKey = false)
    String name;
  public Character(int id,String name) {
this.id = id;
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

Now it's time to play with reflections. For table creation i will be creating another class as a utility you can create the same function directly in the main file .

```java
package org.game.utils;

import org.game.annotations.Column;
import org.game.annotations.Table;
import org.reflections.Reflections;

import java.lang.reflect.Field;
import java.util.Objects;
import java.util.Set;

public class TableCreator {
    public static void createTables(){
        Reflections reflections = new Reflections("org.game.entities");
// getting the all classes annotated with @Table ans storing it into a set to ensure no duplicate table creation
        Set<Class<?>> annotatedClass 
      = reflections.getTypesAnnotatedWith(Table.class);

  // for checking i will be generating all the queries and printing     
 for(Class<?> clazz:annotatedClass){
               String query = generateSQLqueries(clazz);

            System.out.println(query);
            Statement statement = null;
            Connection connection = Connector.connect();
            try{
                statement = connection.createStatement();
                statement.executeUpdate(query);
            }
          catch (SQLException e){
e.printStackTrace();
          }
            finally {
                try {
                    if (statement != null) statement.close();
                    if (connection != null) connection.close();
                } catch (SQLException se) {
                    se.printStackTrace();
                }
            }
        }
    } 


    public static String generateSQLqueries(Class<?> clazz){
if(!clazz.isAnnotationPresent(Table.class)){
    throw new RuntimeException("Class is not annotated with @Table");
}
Table table = clazz.getAnnotation(Table.class); // for getting the properties or values we defined in the annotation
StringBuilder query = new StringBuilder("CREATE TABLE ").append(table.name()).append(" ( ");
        Field[] fields = clazz.getDeclaredFields(); // all the declared fields of the class
        for(Field field : fields){
            if(field.isAnnotationPresent(Column.class)){ //checking wether the field is annotated with @Column
                Column column = field.getAnnotation(Column.class);
                String columnName = column.name();
                String columnType = column.type();
// in the case column name is not specified then the field name converted to snake_case will be used as default
                if(Objects.equals(columnName, "")){
                    columnName = StringUtils.convertToSnakeCase(field.getName());
                }
// in the case column type is not specified then the type corresponding to field type will be used 
                if(Objects.equals(columnType, "")){
                    columnType = correspondingColumnTypeOfField(field.getClass());
                }
                query.append(columnName).append(" ").append(columnType);
                if(column.primaryKey()){
                    query.append(" PRIMARY KEY");
                }
                query.append(" ,");
            }

        }
        query.setLength(query.length() - 2);
        query.append(" );");
        return query.toString();
    }

    public static String correspondingColumnTypeOfField(Class<?> javaType){
        if (javaType == String.class) {
            return "VARCHAR(255)";
        } else if (javaType == int.class || javaType == Integer.class) {
            return "INT";
        } else if (javaType == long.class || javaType == Long.class) {
            return "BIGINT";
        } else if (javaType == double.class || javaType == Double.class) {
            return "DOUBLE";
        } else if (javaType == float.class || javaType == Float.class) {
            return "FLOAT";
        } else if (javaType == boolean.class || javaType == Boolean.class) {
            return "BOOLEAN";
        } else if (javaType == java.util.Date.class || javaType == java.sql.Date.class) {
            return "DATE";
        } else if (javaType == java.sql.Timestamp.class) {
            return "TIMESTAMP";
        } else if (javaType == java.math.BigDecimal.class) {
            return "DECIMAL";
        } else {
            throw new IllegalArgumentException("Unsupported Java type: " + javaType.getName());
        }

    }
}
```

Now if i call this createTables() in our main function and run our main function what you will see that a new table will be created with the name \_character

I have deleted the preexisting table

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721340835215/83a03c22-be3e-46c2-8254-34ee38edeb4d.png align="left")

```java
package org.game;

import org.game.entities.Characters.Player;

import org.game.utils.TableCreator;

public class Main{
public static void main(String[] args){
TableCreator.createTables();
}
}
```

Output:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721341046028/39671fb0-c2a0-41f4-9601-f646ce26b556.png align="center")

Now if we check the database

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721341117754/0bf75516-b81d-44d3-b197-d0b4042dc46f.png align="center")

So our one part is done i.e Class to table mapping is done no remaining part is the Table row to Class Object mapping.

**Now for TableRow to Object mapping we will require a generic Class which will handle our Fetching the data from database for all it's Child classes .**

***Generic Interface defining all the required methods of every DAO class***

```java
package org.game.database.dao;

import java.util.List;

public interface GenericDAO<T,K> {
   boolean add(T entity);
    T findById(K id);
    List<T> getAll();
    boolean update(T entity);
   boolean delete(T entity);
}
```

***GenericDAO Implementation***

```java
package org.game.database.daoImpl;

import org.game.database.Connector;
import org.game.database.dao.GenericDAO;
import org.game.utils.StringUtils;

import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;

public class GenericDAOImpl<T> implements GenericDAO<T,Integer> {
    Class<T> entityClass;
    private String tableName ;
    public GenericDAOImpl(String tableName,Class<T> entityClass) {
        this.tableName = tableName;
        this.entityClass = entityClass;
    }
    public GenericDAOImpl(Class<T> entityClass){
        this.tableName = StringUtils.convertToSnakeCase(entityClass.getName());
        this.entityClass = entityClass;
    }
    private final Connection connection = Connector.connect();
    @Override
    public boolean add(T entity) {
        try {
            StringBuilder query = new StringBuilder("INSERT INTO ")
                                     .append(tableName).append(" (");
            StringBuilder values = new StringBuilder("VALUES (");

            Field[] fields = entityClass.getDeclaredFields();
            for (Field field : fields) {
                field.setAccessible(true);
                query.append(StringUtils.convertToSnakeCase(field.getName()))
                .append(",");
                values.append("'").append(field.get(entity)).append("',");
            }

            query.setLength(query.length() - 1); // Remove last comma
            values.setLength(values.length() - 1); // Remove last comma

            query.append(") ").append(values).append(")");
            try (PreparedStatement statement = connection
                                        .prepareStatement(query.toString())) 
                        {
                            statement.executeUpdate();
                        }
            return true;
        } catch (SQLException | IllegalAccessException e) {

            e.printStackTrace();
            return false;
        }
    }

    @Override
    public T findById(Integer id) {
        T entity = null;

        try {
            String query = "SELECT * FROM " + tableName + " WHERE id = ?";
            try (PreparedStatement statement = connection.prepareStatement(query)) {
                statement.setInt(1, id);
                try (ResultSet resultSet = statement.executeQuery()) {
                    if (resultSet.next()) {
                        entity = entityClass.getDeclaredConstructor().newInstance();
             for (Field field : entityClass.getDeclaredFields()) {
            field.setAccessible(true);
            field.set(entity, resultSet.getObject(StringUtils.convertToSnakeCase(field.getName())));
                        }
                    }
                }
            }
        } catch (SQLException | IllegalAccessException | InstantiationException | NoSuchMethodException |
                 InvocationTargetException e) {
            e.printStackTrace();
        }
        return entity;
    }

    @Override
    public List<T> getAll() {
        List<T> entities = new ArrayList<T>();
        String query = "SELECT * FROM "+tableName;
        try(PreparedStatement statement = connection.prepareStatement(query)){
            ResultSet resultSet = statement.executeQuery();
            while(resultSet.next()){
                T entity = entityClass.getDeclaredConstructor().newInstance();
                for(Field field: entityClass.getFields()){
                    field.setAccessible(true);
                    field.set(entity,resultSet.getObject(StringUtils.convertToSnakeCase(field.getName())));
                }
                entities.add(entity);
            }
        }
        catch (SQLException |IllegalAccessException | InstantiationException | NoSuchMethodException | InvocationTargetException e){

            e.printStackTrace();
        }
        return entities;
    }

    @Override
    public boolean update(T entity) {
        try{
            StringBuilder query = new StringBuilder().append("UPDATE ").append(tableName).append(" SET");
            Field idField = null;
            Object idValue = null;

            for(Field field: entityClass.getDeclaredFields()){
                field.setAccessible(true);
                if (field.getName().equalsIgnoreCase("id")) {
                    idField = field;
                    idValue = field.get(entity);
                } else {
                    query.append(StringUtils.convertToSnakeCase(field.getName())).append("='").append(field.get(entity)).append("',");
                }
            }
            query.setLength(query.length() - 1); // Remove last comma
            query.append(" WHERE id=").append(idValue);
            try(PreparedStatement statement = connection.prepareStatement(query.toString())){
                statement.executeUpdate();
            }
        }
        catch (SQLException | IllegalAccessException e){
e.printStackTrace();
return false;
        }
return true;
    }

    @Override
    public boolean delete(T entity) {
        try {
            Field idField = entityClass.getDeclaredField("id");
            idField.setAccessible(true);
            Object idValue = idField.get(entity);

            String query = "DELETE FROM " + tableName + " WHERE id = ?";
            try (PreparedStatement statement = connection.prepareStatement(query)) {
                statement.setObject(1, idValue);
                statement.executeUpdate();
            }
        } catch (SQLException | IllegalAccessException | NoSuchFieldException e) {
            e.printStackTrace();
            return false;
        }
        return true;
    }
}
```

The `GenericDAOImpl<T>` class implements the `GenericDAO<T, Integer>` interface and provides a generic solution for database operations using JDBC. It leverages Java reflection to dynamically handle entity fields, making it flexible and reusable across different types of entities.

1. #### **Class Definition**
    

```java
public class GenericDAOImpl<T> implements GenericDAO<T, Integer>
```

**This class is a generic DAO implementation for entities of type**`T`**with**`Integer`**as the ID type.**

2. **Fields and Constructor**
    

```java
Class<T> entityClass;
private String tableName;
private final Connection connection = Connector.connect();
```

* `entityClass`: Holds the class type of the entity.
    
* `tableName`: Name of the database table.
    
* `connection`: Manages the database connection.
    

3. **Add Method**
    
    ```java
    @Override
    public boolean add(T entity) {
        try {
            StringBuilder query = new StringBuilder("INSERT INTO ").append(tableName).append(" (");
            StringBuilder values = new StringBuilder("VALUES (");
    
            Field[] fields = entityClass.getDeclaredFields();
            for (Field field : fields) {
                field.setAccessible(true);
                query.append(StringUtils.convertToSnakeCase(field.getName())).append(",");
                values.append("'").append(field.get(entity)).append("',");
            }
    
            query.setLength(query.length() - 1); // Remove last comma
            values.setLength(values.length() - 1); // Remove last comma
    
            query.append(") ").append(values).append(")");
            try (PreparedStatement statement = connection.prepareStatement(query.toString())) {
                statement.executeUpdate();
            }
            return true;
        } catch (SQLException | IllegalAccessException e) {
            e.printStackTrace();
            return false;
        }
    }
    ```
    
    ***This method constructs and executes an***`INSERT`***SQL query using reflection to dynamically generate the query based on the entity's fields and values.***
    
4. **FindById Method**
    
    ```java
    @Override
    public T findById(Integer id) {
        T entity = null;
    
        try {
            String query = "SELECT * FROM " + tableName + " WHERE id = ?";
            try (PreparedStatement statement = connection.prepareStatement(query)) {
                statement.setInt(1, id);
                try (ResultSet resultSet = statement.executeQuery()) {
                    if (resultSet.next()) {
                        entity = entityClass.getDeclaredConstructor().newInstance();
                        for (Field field : entityClass.getDeclaredFields()) {
                            field.setAccessible(true);
                            field.set(entity, resultSet.getObject(StringUtils.convertToSnakeCase(field.getName())));
                        }
                    }
                }
            }
        } catch (SQLException | IllegalAccessException | InstantiationException | NoSuchMethodException | InvocationTargetException e) {
            e.printStackTrace();
        }
        return entity;
    }
    ```
    
    ***This method retrieves an entity by its ID. It uses reflection to map the result set's columns to the entity's fields.***
    
5. **GetAll Method**
    
    ```java
    @Override
    public List<T> getAll() {
        List<T> entities = new ArrayList<>();
        String query = "SELECT * FROM " + tableName;
        try (PreparedStatement statement = connection.prepareStatement(query)) {
            ResultSet resultSet = statement.executeQuery();
            while (resultSet.next()) {
                T entity = entityClass.getDeclaredConstructor().newInstance();
                for (Field field : entityClass.getDeclaredFields()) {
                    field.setAccessible(true);
                    field.set(entity, resultSet.getObject(StringUtils.convertToSnakeCase(field.getName())));
                }
                entities.add(entity);
            }
        } catch (SQLException | IllegalAccessException | InstantiationException | NoSuchMethodException | InvocationTargetException e) {
            e.printStackTrace();
        }
        return entities;
    }
    ```
    
    ***This method retrieves all entities from the table, mapping each row of the result set to an entity object.***
    
6. **Update Method**
    
    ```java
    @Override
    public boolean update(T entity) {
        try {
            StringBuilder query = new StringBuilder("UPDATE ").append(tableName).append(" SET ");
            Field idField = null;
            Object idValue = null;
    
            for (Field field : entityClass.getDeclaredFields()) {
                field.setAccessible(true);
                if (field.getName().equalsIgnoreCase("id")) {
                    idField = field;
                    idValue = field.get(entity);
                } else {
                    query.append(StringUtils.convertToSnakeCase(field.getName())).append("='").append(field.get(entity)).append("',");
                }
            }
            query.setLength(query.length() - 1); // Remove last comma
            query.append(" WHERE id=").append(idValue);
            try (PreparedStatement statement = connection.prepareStatement(query.toString())) {
                statement.executeUpdate();
            }
            return true;
        } catch (SQLException | IllegalAccessException e) {
            e.printStackTrace();
            return false;
        }
    }
    ```
    
    ***Constructs and executes an***`UPDATE`***SQL query, using reflection to set field values and update the entity based on its ID.***
    
7. **Delete Method**
    
    ```java
    @Override
    public boolean delete(T entity) {
        try {
            Field idField = entityClass.getDeclaredField("id");
            idField.setAccessible(true);
            Object idValue = idField.get(entity);
    
            String query = "DELETE FROM " + tableName + " WHERE id = ?";
            try (PreparedStatement statement = connection.prepareStatement(query)) {
                statement.setObject(1, idValue);
                statement.executeUpdate();
            }
            return true;
        } catch (SQLException | IllegalAccessException | NoSuchFieldException e) {
            e.printStackTrace();
            return false;
        }
    }
    ```
    
    ***Constructs and executes a***`DELETE`***SQL query to remove an entity from the table using its ID.***
    

So yeah this was our GenericDAOClassImpl now what we finally need is creating a child class for the CharacterEntity which will extend GenericDAOClassImpl because for updating that entity we will be using the child class not the GenericDAO class .

```java
public class CharacterDAOImpl extends GenericDAOImpl<Character>  {
    private final Connection connection = Connector.connect();

      public CharacterDAOImpl(){
        super("_character",Character.class);

    }
}
```

Here we are making our CharacterDaoImpl extend the GenericDAOImpl by providing the Class of the entity so all the methods of GenericDAOImpl will be now accessed for the Character class.

### **Our Final Main Class will be looking like this**

```java
public class Main {
    private static final CharacterDAOImpl dao = new CharacterDAOImpl();

    public static void main(String[] args) {
        TableCreator.createTables();
//
        Character character = new Character(234,"Mayank");
        dao.add(character); // adding the character to database
        List<Character> fetched = dao.getAll(); //fetching all the characters saved
fetched.forEach((character1)->{
    System.out.println(character1.getId()+" "+character1.getName());
});
    }
}
```

**Testing Time**

Here is the before i.e our table is empty

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721369669217/a76a5f11-8a73-4775-95e2-d1ed6a8fd41b.png align="center")

  
Now After Running the code

### Output

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721370320337/30a842d1-4cc1-40f5-abef-fb62192a9141.png align="center")

**Now if we look into our database**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721370425598/de527858-114c-40d8-a664-c16e5cfd7db0.png align="center")

So this was my approach and solution for implementing it whether my approach and solution is correct or wrong i don't know if anyone have any suggestion please leave it in the comment section . Thanks for reading . Bye 🫡 will meet in next one.