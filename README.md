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
Let's define a very simple model using classes. We're just defining them in the **Program.cs** file but in a real world application you would split your classes out into separate files and potentially a separate project.

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
Now it's time to define a derived context, which represents a session with the database, allowing us to query and save data. We define a context that derives from Microsoft.EntityFrameworkCore.DbContext and exposes a typed DbSet<TEntity> for each class in our model.
    
We're now starting to use types from the Entity Framework Core so we need to add the EntityFrameworkCore NuGet packages.

* Open **PowerShell**
* Run these commands to include the Entity Framework Core package references:
    * ```dotnet add package Microsoft.EntityFrameworkCore.SqlServer```
    * ```dotnet add package Microsoft.EntityFrameworkCore.Design```
* Insert these lines to your **CodeFirstNewDatabaseSample.csproj** file to include the CLI tool reference:
    * **Microsoft.EntityFrameworkCore.Tools.DotNet** must be added as a CLI tool reference (currently manually) and not as a package reference.
```xml
<ItemGroup>
    <DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet">
        <Version>2.0.0-*</Version>
    </DotNetCliToolReference>
</ItemGroup>
```

* Confirm the Entity Framework Core CLI has been included correctly by typing ```dotnet ef``` in Powershell and seeing the _unicorn_.

```
                     _/\__       
               ---==/    \\      
         ___  ___   |.    \|\    
        | __|| __|  |  )   \\\   
        | _| | _|   \_/ |  //|\\ 
        |___||_|       /   \\\/\\
```

* Add the following _using_ statements to the top of **Project.cs**:
```csharp
using System.Collections.Generic;
using Microsoft.EntityFrameworkCore;
```

* Create the following **DbContext** under your Post class: 
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

Here is a complete listing of what **Program.cs** should look like at this time.

```csharp
using System;
using System.Collections.Generic;
using Microsoft.EntityFrameworkCore;

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

    class Program
    {
        private static void Main(string[] args)
        {
            Console.WriteLine("Hello World!");
        }
    }
}
```

Now we need to run a few commands to initialise our database according to our entities:

* Re-open **Powershell**
* Run ```dotnet ef migrations add Initial``` to create our first migration, which we will discuss later
* Run ```dotnet ef database update``` to initialise our database according to the migration

That is all we need to start storing and retrieving data. Obviously there is quite a bit going on behind the scenes and we’ll take a look at that in a moment but first let’s see it in action.

## 4. Reading & Writing Data
Implement the Main method in Program.cs as shown below. This code creates a new instance of our context and then uses it to insert a new Blog. Then it uses a LINQ query to retrieve all Blogs from the database ordered alphabetically by Title.

**.OrderBy()** is an extension method found in **System.Linq**. Do not forget to include it in a using statement at the top of the file.

```csharp
using System.Linq;

class Program
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
According to the statement ```optionsBuilder.UseSqlServer("Data Source=(localdb)\\MSSQLLocalDB; Initial Catalog=CodeFirstNewDatabaseSample;");``` a SQL LocalDB database with the name **"CodeFirstNewDatabaseSample"** has been created.

You can connect to this database using SQL Server Object Explorer in Visual Studio 2017.

* **View -> SQL Server Object Explorer...** (Ctrl+\, Ctrl+S)
* Expand **SQL Server -> (localdb)\MSSQLLocalDB -> Databases -> CodeFirstNewDatabaseSample -> Tables**

We can now inspect the tables that Code First created.

![Tables](https://i.imgur.com/EtJWUo7.png)

DbContext worked out what classes to include in the model by looking at the DbSet properties that we defined. It then uses the default set of Code First conventions to determine table and column names, determine data types, find primary keys, etc. Later in this walkthrough we’ll look at how you can override these conventions.

## 5. Dealing with Model Changes
Now it’s time to make some changes to our model, when we make these changes we also need to update the database schema. To do this we are going to use a feature called Code First Migrations, or Migrations for short.

Now let’s make a change to our model, add a Url property to the Blog class:

```csharp
public class Blog
{
    public int BlogId { get; set; }
    public string Name { get; set; }
    public string Url { get; set; }
    public List<Post> Posts { get; set; }
}
```
Now we need to add a migration for the change:

* Re-open **Powershell**
* Run ```dotnet ef migrations add AddUrlToBlog``` 
    * This command checks for changes since your last migration and scaffolds a new migration with any changes that are found. We can give migrations a name; in this case we are calling the migration ‘AddUrlToBlog’. The scaffolded code is saying that we need to add a Url column, that can hold string data, to the dbo.Blogs table. If needed, we could edit the scaffolded code but that’s not required in this case.

```csharp
namespace CodeFirstNewDatabaseSample.Migrations
{
    public partial class AddUrlToBlog : Migration
    {
        protected override void Up(MigrationBuilder migrationBuilder)
        {
            migrationBuilder.AddColumn<string>(
                name: "Url",
                table: "Blogs",
                nullable: true);
        }

        protected override void Down(MigrationBuilder migrationBuilder)
        {
            migrationBuilder.DropColumn(
                name: "Url",
                table: "Blogs");
        }
    }
}
```

* Run ```dotnet ef database update```
    * This command will apply any pending migrations to the database. Our Initial migration has already been applied so migrations will just apply our new AddUrlToBlog migration. Tip: You can use the –Verbose switch when calling database update to see the SQL that is being executed against the database

The new ```Url``` column is now added to the ```Blogs``` table in the database:

![Tables](https://i.imgur.com/Y2fDHLR.png)

## 6. Data Annotations
So far we’ve just let EF discover the model using its default conventions, but there are going to be times when our classes don’t follow the conventions and we need to be able to perform further configuration. There are two options for this; we’ll look at Data Annotations in this section and then the fluent API in the next section.

* Let’s add a User class to our model
```csharp
public class User
{
    public string UserName { get; set; }
    public string DisplayName { get; set; }
}
```
* We also need to add a set to our derived context
```csharp
public class BloggingContext : DbContext
{
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer("Data Source=(localdb)\\MSSQLLocalDB; Initial Catalog=CodeFirstNewDatabaseSample;");
    }

    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }
    public DbSet<User> Users { get; set; }
}
```
* If we tried to add a migration we’d get an error saying “The entity type 'User' requires a primary key to be defined.” because EF has no way of knowing that UserName should be the primary key for User.
* We’re going to use Data Annotations now so we need to add a using statement at the top of Program.cs
   
   ```using System.ComponentModel.DataAnnotations;```
* Now annotate the Username property to identify that it is the primary key
```csharp
public class User
{
    [Key]
    public string UserName { get; set; }
    public string DisplayName { get; set; }
}
```
* Use the ```dotnet ef migrations add AddUser``` command to scaffold a migration to apply these changes to the database
* Run the ```dotnet ef database update``` command to apply the new migration to the database

The new table is now added to the database:

![Users Table](https://i.imgur.com/fI3Svxg.png)

The full list of annotations supported by EF is:

* [KeyAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.keyattribute "KeyAttribute")
* [StringLengthAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.stringlengthattribute "StringLengthAttribute")
* [MaxLengthAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.maxlengthattribute "MaxLengthAttribute")
* [ConcurrencyCheckAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.concurrencycheckattribute "ConcurrencyCheckAttribute")
* [RequiredAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.requiredattribute "RequiredAttribute")
* [TimestampAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.timestampattribute "TimestampAttribute")
* [ComplexTypeAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.schema.complextypeattribute "ComplexTypeAttribute")
* [ColumnAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.schema.columnattribute "ColumnAttribute")
* [TableAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.schema.tableattribute "TableAttribute")
* [InversePropertyAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.schema.inversepropertyattribute "InversePropertyAttribute")
* [ForeignKeyAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.schema.foreignkeyattribute "ForeignKeyAttribute")
* [DatabaseGeneratedAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.schema.databasegeneratedattribute "DatabaseGeneratedAttribute")
* [NotMappedAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.schema.notmappedattribute "NotMappedAttribute")
## 7. Fluent API
In the previous section we looked at using Data Annotations to supplement or override what was detected by convention. The other way to configure the model is via the Code First fluent API.

Most model configuration can be done using simple data annotations. The fluent API is a more advanced way of specifying model configuration that covers everything that data annotations can do in addition to some more advanced configuration not possible with data annotations. Data annotations and the fluent API can be used together.

To access the fluent API you override the OnModelCreating method in DbContext. Let’s say we wanted to rename the column that User.DisplayName is stored in to display_name.

* Override the OnModelCreating method on BloggingContext with the following code

```csharp
public class BloggingContext : DbContext
{
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer("Data Source=(localdb)\\MSSQLLocalDB; Initial Catalog=CodeFirstNewDatabaseSample;");
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<User>()
            .Property(u => u.DisplayName)
            .HasColumnName("display_name");
    }

    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }
    public DbSet<User> Users { get; set; }
}
```
* Use the ```dotnet ef migrations add ChangeDisplayName``` command to scaffold a migration to apply these changes to the database
* Run the ```dotnet ef database update``` command to apply the new migration to the database

The DisplayName column is now renamed to display_name:

![Display Name](https://i.imgur.com/a0WJBXE.png)

## Summary
In this walkthrough we looked at Code First development using a new database. We defined a model using classes then used that model to create a database and store and retrieve data. Once the database was created we used Code First Migrations to change the schema as our model evolved. We also saw how to configure a model using Data Annotations and the Fluent API.
