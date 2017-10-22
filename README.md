# Entity Framework Core Code First to a New Database

_Disclaimer: This is an re-implementation of the **Entity Framework Code First to a New Database** article (https://msdn.microsoft.com/en-gb/data/jj193542) but now using Entity Framework Core._

## Prerequisites
You will need to have Visual Studio 2017 installed to complete this walkthrough.

## 1. Create the Application

* Open Visual Studio
* **File -> New -> Project...**
* Select **.NET Core** from the left menu and **Console App (.NET Core)**
* Enter **CodeFirstNewDatabaseSample** as the name
* Select **OK**

## 2. Create the Model
Let's define a very simple model using classes. We're just defining them in the Program.cs file but in a real world application you would split your classes out into separate files and potentially a separate project.

Below the Program class definition in Program.cs add the following two classes.

```csharp
public class Blog
{
    public int BlogId { get; set; }
    public string Name { get; set; }

    public List<Post> Posts { get; set; }
}

public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }

    public int BlogId { get; set; }
    public Blog Blog { get; set; }
}
```

## 3. Create a Context

```csharp
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }
}
```

## 4. Reading & Writing Data

## Where's My Data?
By convention DbContext has created a database for you.
* Code First will try and use LocalDb (installed by default with Visual Studio 2017)
* The database is named after the fully qualified name of the derived context, in our case that is **CodeFirstNewDatabaseSample.BloggingContext**

## 5. Dealing with Model Changes

## 6. Data Annotations

## 7. Fluent API

## Summary
In this walkthrough we looked at Code First development using a new database. We defined a model using classes then used that model to create a database and store and retrieve data. Once the database was created we used Code First Migrations to change the schema as our model evolved. We also saw how to configure a model using Data Annotations and the Fluent API.