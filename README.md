# Entity Framework Core Code First to a New Database

_Disclaimer: This is an re-implementation of the **Entity Framework Code First to a New Database** article (https://msdn.microsoft.com/en-gb/data/jj193542) but now using Entity Framework Core._

## Prerequisites
You will need to have the following installed to complete this walkthrough:
* Visual Studio 2017 with the .NET Core cross-platform development toolset 
* .Net Core SDK

## 1. Create the Application

* Open Visual Studio
* **File -> New -> Project...** (Ctrl+Shift+N by default)
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
Now it’s time to define a derived context, which represents a session with the database, allowing us to query and save data. We define a context that derives from Microsoft.EntityFrameworkCore.DbContext and exposes a typed DbSet<TEntity> for each class in our model.
    
We’re now starting to use types from the Entity Framework Core so we need to add the EntityFrameworkCore NuGet packages.

* **Tools -> NuGet Package Manager -> Package Manager Console...**

* Run these commands:
    * ```Install-Package Microsoft.EntityFrameworkCore.Design```
    * ```Install-Package Microsoft.EntityFrameworkCore.SqlServer```
    * ```Install-Package Microsoft.EntityFrameworkCore.Tools.DotNet```
    
 * Finally add these lines to your CodeFirstNewDatabaseSample.csproj file:

```xml
<ItemGroup>
    <DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet">
        <Version>1.0.0-*</Version>
    </DotNetCliToolReference>
</ItemGroup>
```

* Add the following using statements to the top of your Project.cs:
```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.System.Collections.Generic;
```

* Now add the following DbContext under your Post class: 
```csharp
public class BloggingContext : DbContext
{
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer("Data Source=(localdb)\\MSSQLLocalDB; Initial Catalog=CodeFirstNewDatabaseSample;");
    }
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }
}
```

Here is a complete listing of what Program.cs should now contain.

```csharp
using Microsoft.EntityFrameworkCore;
using System;
using System.Collections.Generic;

namespace CodeFirstNewDatabaseSample
{
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

    public class BloggingContext : DbContext
    {
        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            optionsBuilder.UseSqlServer("Data Source=(localdb)\\MSSQLLocalDB; Initial Catalog=CodeFirstNewDatabaseSample;");
        }

        public DbSet<Blog> Blogs { get; set; }
        public DbSet<Post> Posts { get; set; }
    }

    internal class Program
    {
        private static void Main(string[] args)
        {
            Console.WriteLine("Hello World!");
        }
    }
}
```

Now we need to run a few commands to initialise our database according to our entities:

* Open a command line window in the project folder
* Run ```dotnet ef migrations add Initial``` to create our first migration, which we will discuss later
* Run ```dotnet ef database update``` to initialise our database according to the migration

That is all we need to start storing and retrieving data. Obviously there is quite a bit going on behind the scenes and we’ll take a look at that in a moment but first let’s see it in action.

## 4. Reading & Writing Data
Implement the Main method in Program.cs as shown below. This code creates a new instance of our context and then uses it to insert a new Blog. Then it uses a LINQ query to retrieve all Blogs from the database ordered alphabetically by Title.

```csharp
    internal class Program
    {
        private static void Main(string[] args)
        {
            using (var db = new BloggingContext())
            {
                // Create and save a new Blog 
                Console.Write("Enter a name for a new Blog: ");
                var name = Console.ReadLine();
                
                db.Blogs.Add(new Blog { Name = name });
                db.SaveChanges();

                // Display all Blogs from the database 
                var blogsOrderedByName = db.Blogs.OrderBy(blog => blog.Name);

                Console.WriteLine("All blogs in the database:");
                foreach (var blog in blogsOrderedByName)
                {
                    Console.WriteLine(blog.Name);
                }

                Console.WriteLine("Press any key to exit...");
                Console.ReadKey();
            }
        }
    }
```
You can now run the application and test it out.

```
Enter a name for a new Blog: AppFactoryBlog
All blogs in the database:
AppFactoryBlog
Press any key to exit...

```
## Where's My Data?
According to the statement ``` optionsBuilder.UseSqlServer("Data Source=(localdb)\\MSSQLLocalDB; Initial Catalog=CodeFirstNewDatabaseSample;");``` a SQL LocalDB database with the name **"CodeFirstNewDatabaseSample"** has been created.

You can connect to this database using SQL Server Object Explorer in Visual Studio 2017.

* **View -> SQL Server Object Explorer...** (Ctrl+\, Ctrl+S)
* Expand **SQL Server -> (localdb)\MSSQLLocalDB -> Databases -> CodeFirstNewDatabaseSample -> Tables**

We can now inspect the tables that Code First created.

DbContext worked out what classes to include in the model by looking at the DbSet properties that we defined. It then uses the default set of Code First conventions to determine table and column names, determine data types, find primary keys, etc. Later in this walkthrough we’ll look at how you can override these conventions.

## 5. Dealing with Model Changes

## 6. Data Annotations

## 7. Fluent API

## Summary
In this walkthrough we looked at Code First development using a new database. We defined a model using classes then used that model to create a database and store and retrieve data. Once the database was created we used Code First Migrations to change the schema as our model evolved. We also saw how to configure a model using Data Annotations and the Fluent API.
