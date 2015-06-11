# EF Code First project setup

## Install Entity Framework 6

From the **Tools** menu click **NuGet Package Manager** and then click **Package Manager Console**.
In the **Package Manager Console** window enter the following command:

`Install-Package EntityFramework`


## Create the Data Model
In the Models folder, create a class file named Student.cs and replace the template code with the following code:

```csharp
namespace ContosoUniversity.Models
{
	public class Student
	{
		public int ID { get; set; }
		public string LastName { get; set; }
		public string FirstMidName { get; set; }
		public DateTime EnrollmentDate { get; set; }
		
		public virtual ICollection<Enrollment> Enrollments { get; set; }
	}
	
	public enum Grade
	{
		A, B, C, D, F
	}

	public class Enrollment
	{
		public int EnrollmentID { get; set; }
		public int CourseID { get; set; }
		public int StudentID { get; set; }
		public Grade? Grade { get; set; }
		
		public virtual Course Course { get; set; }
		public virtual Student Student { get; set; }
	}
	
	public class Course
	{
		[DatabaseGenerated(DatabaseGeneratedOption.None)]
		public int CourseID { get; set; }
		public string Title { get; set; }
		public int Credits { get; set; }
		
		public virtual ICollection<Enrollment> Enrollments { get; set; }
	}
}
```

## Create the Database Context

```csharp
namespace ContosoUniversity.DAL
{
	public class SchoolContext : DbContext
	{
	
		public SchoolContext() : base("SchoolContext")
		{
		}
		
		public DbSet<Student> Students { get; set; }
		public DbSet<Enrollment> Enrollments { get; set; }
		public DbSet<Course> Courses { get; set; }

		protected override void OnModelCreating(DbModelBuilder modelBuilder)
		{
			modelBuilder.Conventions.Remove<PluralizingTableNameConvention>();
		}
	}
}
```
## Set up EF to initialize the database with test data

```csharp
namespace ContosoUniversity.DAL
{
	public class SchoolInitializer : System.Data.Entity. DropCreateDatabaseIfModelChanges<SchoolContext>
	{
		protected override void Seed(SchoolContext context)
		{
			var students = new List<Student>
			{
			new Student{FirstMidName="Carson",LastName="Alexander",EnrollmentDate=DateTime.Parse("2005-09-01")},
			new Student{FirstMidName="Meredith",LastName="Alonso",EnrollmentDate=DateTime.Parse("2002-09-01")},
			new Student{FirstMidName="Arturo",LastName="Anand",EnrollmentDate=DateTime.Parse("2003-09-01")},
			new Student{FirstMidName="Gytis",LastName="Barzdukas",EnrollmentDate=DateTime.Parse("2002-09-01")},
			new Student{FirstMidName="Yan",LastName="Li",EnrollmentDate=DateTime.Parse("2002-09-01")},
			new Student{FirstMidName="Peggy",LastName="Justice",EnrollmentDate=DateTime.Parse("2001-09-01")},
			new Student{FirstMidName="Laura",LastName="Norman",EnrollmentDate=DateTime.Parse("2003-09-01")},
			new Student{FirstMidName="Nino",LastName="Olivetto",EnrollmentDate=DateTime.Parse("2005-09-01")}
			};

			students.ForEach(s => context.Students.Add(s));
			context.SaveChanges();
			var courses = new List<Course>
			{
			new Course{CourseID=1050,Title="Chemistry",Credits=3,},
			new Course{CourseID=4022,Title="Microeconomics",Credits=3,},
			new Course{CourseID=4041,Title="Macroeconomics",Credits=3,},
			new Course{CourseID=1045,Title="Calculus",Credits=4,},
			new Course{CourseID=3141,Title="Trigonometry",Credits=4,},
			new Course{CourseID=2021,Title="Composition",Credits=3,},
			new Course{CourseID=2042,Title="Literature",Credits=4,}
			};
			courses.ForEach(s => context.Courses.Add(s));
			context.SaveChanges();
			var enrollments = new List<Enrollment>
			{
			new Enrollment{StudentID=1,CourseID=1050,Grade=Grade.A},
			new Enrollment{StudentID=1,CourseID=4022,Grade=Grade.C},
			new Enrollment{StudentID=1,CourseID=4041,Grade=Grade.B},
			new Enrollment{StudentID=2,CourseID=1045,Grade=Grade.B},
			new Enrollment{StudentID=2,CourseID=3141,Grade=Grade.F},
			new Enrollment{StudentID=2,CourseID=2021,Grade=Grade.F},
			new Enrollment{StudentID=3,CourseID=1050},
			new Enrollment{StudentID=4,CourseID=1050,},
			new Enrollment{StudentID=4,CourseID=4022,Grade=Grade.F},
			new Enrollment{StudentID=5,CourseID=4041,Grade=Grade.C},
			new Enrollment{StudentID=6,CourseID=1045},
			new Enrollment{StudentID=7,CourseID=3141,Grade=Grade.A},
			};
			enrollments.ForEach(s => context.Enrollments.Add(s));
			context.SaveChanges();
		}
	}
}
```

To tell Entity Framework to use your initializer class, add an element to the **entityFramework** element in the application **Web.config** file (the one in the root project folder), as shown in the following example:

```XML
<entityFramework>
  <!-- ONLY THIS PART -->
  <contexts>
	<context type="ContosoUniversity.DAL.SchoolContext, ContosoUniversity">
	  <databaseInitializer type="ContosoUniversity.DAL.SchoolInitializer, ContosoUniversity" />
	</context>
  </contexts>
  <!-- ONLY THIS PART -->
  <defaultConnectionFactory type="System.Data.Entity.Infrastructure.LocalDbConnectionFactory, EntityFramework">
	<parameters>
	  <parameter value="v11.0" />
	</parameters>
  </defaultConnectionFactory>
  <providers>
	<provider invariantName="System.Data.SqlClient" type="System.Data.Entity.SqlServer.SqlProviderServices, EntityFramework.SqlServer" />
  </providers>
</entityFramework>
```
The context type specifies the fully qualified context class name and the assembly it's in, and the `databaseinitializer` type specifies the fully qualified name of the initializer class and the assembly it's in. (When you don't want EF to use the initializer, you can set an attribute on the context element: `disableDatabaseInitialization="true"`.)

As an alternative to setting the initializer in the `Web.config` file is to do it in code by adding a `Database.SetInitializer` statement to the `Application_Start` method in the `Global.asax.cs` file.

## Set up EF to use a SQL Server Express LocalDB database
```XML
<connectionStrings>
	<add name="SchoolContext" connectionString="Data Source=(LocalDb)\v11.0;Initial Catalog=ContosoUniversity1;Integrated Security=SSPI;" providerName="System.Data.SqlClient"/>
</connectionStrings>
```

## Enable Code First Migrations

Disable the initializer that you set up earlier by commenting out or deleting the contexts element that you added to the application `Web.config` file.
```XML
<!--<contexts>
    <context type="ContosoUniversity.DAL.SchoolContext, ContosoUniversity">
      <databaseInitializer type="ContosoUniversity.DAL.SchoolInitializer, ContosoUniversity" />
    </context>
  </contexts>-->
```

Also in the application `Web.config` file, change the name of the database in the connection string to `ContosoUniversity2`.

```XML
<connectionStrings>
  <add name="SchoolContext" connectionString="Data Source=(LocalDb)\v11.0;Initial Catalog=ContosoUniversity2;Integrated Security=SSPI;" providerName="System.Data.SqlClient" />
</connectionStrings>
```

From the **Tools** menu, click **Library Package Manager** and then **Package Manager Console**.
At the PM> prompt enter the following commands:

```
enable-migrations
add-migration InitialCreate
```

The `enable-migrations` command creates a _Migrations_ folder in the ContosoUniversity project, and it puts in that folder a _Configuration.cs_ file that you can edit to configure Migrations.

the `Configuration` class includes a `Seed` method. The purpose of the `Seed` method is to enable you to insert or update test data after Code First creates or updates the database. The method is called when the database is created and every time the database schema is updated after a data model change.

```csharp
internal sealed class Configuration : DbMigrationsConfiguration<ContosoUniversity.DAL.SchoolContext>
{
    public Configuration()
    {
        AutomaticMigrationsEnabled = false;
    }

    protected override void Seed(ContosoUniversity.DAL.SchoolContext context)
    {
        //  This method will be called after migrating to the latest version.

        //  You can use the DbSet<T>.AddOrUpdate() helper extension method 
        //  to avoid creating duplicate seed data. E.g.
        //
        //    context.People.AddOrUpdate(
        //      p => p.FullName,
        //      new Person { FullName = "Andrew Peters" },
        //      new Person { FullName = "Brice Lambson" },
        //      new Person { FullName = "Rowan Miller" }
        //    );
        //
    }
}
```

## Set up the Seed Method

Replace the contents of the Configuration.cs file with the following code, which will load test data into the new database. 

```csharp
namespace ContosoUniversity.Migrations
{
    using ContosoUniversity.Models;
    using System;
    using System.Collections.Generic;
    using System.Data.Entity;
    using System.Data.Entity.Migrations;
    using System.Linq;

    internal sealed class Configuration : DbMigrationsConfiguration<ContosoUniversity.DAL.SchoolContext>
    {
        public Configuration()
        {
            AutomaticMigrationsEnabled = false;
        }

        protected override void Seed(ContosoUniversity.DAL.SchoolContext context)
        {
            var students = new List<Student>
            {
                new Student { FirstMidName = "Carson",   LastName = "Alexander", 
                    EnrollmentDate = DateTime.Parse("2010-09-01") },
                new Student { FirstMidName = "Meredith", LastName = "Alonso",    
                    EnrollmentDate = DateTime.Parse("2012-09-01") },
                new Student { FirstMidName = "Arturo",   LastName = "Anand",     
                    EnrollmentDate = DateTime.Parse("2013-09-01") },
                new Student { FirstMidName = "Gytis",    LastName = "Barzdukas", 
                    EnrollmentDate = DateTime.Parse("2012-09-01") },
                new Student { FirstMidName = "Yan",      LastName = "Li",        
                    EnrollmentDate = DateTime.Parse("2012-09-01") },
                new Student { FirstMidName = "Peggy",    LastName = "Justice",   
                    EnrollmentDate = DateTime.Parse("2011-09-01") },
                new Student { FirstMidName = "Laura",    LastName = "Norman",    
                    EnrollmentDate = DateTime.Parse("2013-09-01") },
                new Student { FirstMidName = "Nino",     LastName = "Olivetto",  
                    EnrollmentDate = DateTime.Parse("2005-08-11") }
            };
            students.ForEach(s => context.Students.AddOrUpdate(p => p.LastName, s));
            context.SaveChanges();

            var courses = new List<Course>
            {
                new Course {CourseID = 1050, Title = "Chemistry",      Credits = 3, },
                new Course {CourseID = 4022, Title = "Microeconomics", Credits = 3, },
                new Course {CourseID = 4041, Title = "Macroeconomics", Credits = 3, },
                new Course {CourseID = 1045, Title = "Calculus",       Credits = 4, },
                new Course {CourseID = 3141, Title = "Trigonometry",   Credits = 4, },
                new Course {CourseID = 2021, Title = "Composition",    Credits = 3, },
                new Course {CourseID = 2042, Title = "Literature",     Credits = 4, }
            };
            courses.ForEach(s => context.Courses.AddOrUpdate(p => p.Title, s));
            context.SaveChanges();

            var enrollments = new List<Enrollment>
            {
                new Enrollment { 
                    StudentID = students.Single(s => s.LastName == "Alexander").ID, 
                    CourseID = courses.Single(c => c.Title == "Chemistry" ).CourseID, 
                    Grade = Grade.A 
                },
                 new Enrollment { 
                    StudentID = students.Single(s => s.LastName == "Alexander").ID,
                    CourseID = courses.Single(c => c.Title == "Microeconomics" ).CourseID, 
                    Grade = Grade.C 
                 },                            
                 new Enrollment { 
                    StudentID = students.Single(s => s.LastName == "Alexander").ID,
                    CourseID = courses.Single(c => c.Title == "Macroeconomics" ).CourseID, 
                    Grade = Grade.B
                 },
                 new Enrollment { 
                     StudentID = students.Single(s => s.LastName == "Alonso").ID,
                    CourseID = courses.Single(c => c.Title == "Calculus" ).CourseID, 
                    Grade = Grade.B 
                 },
                 new Enrollment { 
                     StudentID = students.Single(s => s.LastName == "Alonso").ID,
                    CourseID = courses.Single(c => c.Title == "Trigonometry" ).CourseID, 
                    Grade = Grade.B 
                 },
                 new Enrollment {
                    StudentID = students.Single(s => s.LastName == "Alonso").ID,
                    CourseID = courses.Single(c => c.Title == "Composition" ).CourseID, 
                    Grade = Grade.B 
                 },
                 new Enrollment { 
                    StudentID = students.Single(s => s.LastName == "Anand").ID,
                    CourseID = courses.Single(c => c.Title == "Chemistry" ).CourseID
                 },
                 new Enrollment { 
                    StudentID = students.Single(s => s.LastName == "Anand").ID,
                    CourseID = courses.Single(c => c.Title == "Microeconomics").CourseID,
                    Grade = Grade.B         
                 },
                new Enrollment { 
                    StudentID = students.Single(s => s.LastName == "Barzdukas").ID,
                    CourseID = courses.Single(c => c.Title == "Chemistry").CourseID,
                    Grade = Grade.B         
                 },
                 new Enrollment { 
                    StudentID = students.Single(s => s.LastName == "Li").ID,
                    CourseID = courses.Single(c => c.Title == "Composition").CourseID,
                    Grade = Grade.B         
                 },
                 new Enrollment { 
                    StudentID = students.Single(s => s.LastName == "Justice").ID,
                    CourseID = courses.Single(c => c.Title == "Literature").CourseID,
                    Grade = Grade.B         
                 }
            };

            foreach (Enrollment e in enrollments)
            {
                var enrollmentInDataBase = context.Enrollments.Where(
                    s =>
                         s.Student.ID == e.StudentID &&
                         s.Course.CourseID == e.CourseID).SingleOrDefault();
                if (enrollmentInDataBase == null)
                {
                    context.Enrollments.Add(e);
                }
            }
            context.SaveChanges();
        }
    }
}
```

**Build the project**

In the **Package Manager Console** window, enter the following command:

`update-database`

The `update-database` command runs the _Up_ method to create the database and then it runs the _Seed_ method to populate the database. The same process will run automatically in production after you deploy the application, as you'll see in the following section.

## Update database schema - new migration

Whenever you update database schema, you need to call following command:

```
add-migration MaxLengthOnNames
update-database
```
