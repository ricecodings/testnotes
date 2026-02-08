# FULLSD Practical Test 2 – Personal Revision Notes (Weeks 1–18)
(Blazor Web App + Web API) | Lab-style patterns only

> IMPORTANT COMPLIANCE NOTE (Open Book Rules)
- These notes are generic revision templates based on weekly labs and official docs.
- They do NOT contain any practical test questions, screenshots, files, or answers.
- During a test: do NOT communicate with anyone, do NOT use generative AI, do NOT save/share test materials.
- Use this as read-only reference.

------------------------------------------------------------
TABLE OF CONTENTS
1) Fast Workflow (what to do in order)
2) Folder Structure (where things go)
3) Entity Templates (Domain)
4) Adding Methods in Entities (business logic)
5) DbContext Templates (Data)
6) Migrations and Database Commands
7) Seeding Templates (Configurations/Entities)
8) Scaffold Razor Components using EF (CRUD) (Visual Studio steps)
9) Identity Scaffold (Register/Login) (Visual Studio steps)
10) Auth UI Patterns (NavMenu AuthorizeView, Authorize attribute)
11) Web API Setup (Program.cs + scaffold controller)
12) Common Errors and Fixes (fast)
13) Quick Copy-Paste Snippets Pack

============================================================
1) FAST WORKFLOW (WHAT TO DO IN ORDER)

If you see a question like “Build full stack app with CRUD + Identity + Web API”, do this:

A) Create Project (Blazor Web App)
B) Scaffold Identity (Register/Login)
C) Create Entity class in Domain
D) Scaffold Razor Components CRUD for entity (creates/updates DbContext)
E) Run migrations (Add-Migration, Update-Database)
F) Add seeding configuration (IEntityTypeConfiguration + HasData)
G) Apply seed in DbContext (OnModelCreating), migrate again
H) Enable Web API controllers in Program.cs
I) Scaffold API controller with actions using EF
J) Test quickly in browser or Swagger (if enabled) or simple GET in address bar

============================================================
2) FOLDER STRUCTURE (WHERE THINGS GO)

Recommended lab-style structure:

YourProject/
  Domain/
    MyTask.cs
    AnotherEntity.cs
  Data/
    YourDbContext.cs
  Configurations/
    Entities/
      MyTaskSeed.cs
  Components/Pages/
    (scaffolded Razor components go here, usually <Entity>Pages folder)
  Controllers/
    MyTasksController.cs

Identity pages (after scaffolding) commonly appear under:
  Components/
    Account/
      Pages/
        Login.razor
        Register.razor

============================================================
3) ENTITY TEMPLATES (DOMAIN)

3.1 Minimal Entity (properties only)
Create: Domain/MyTask.cs
"
using System;

namespace YourProjectName.Domain
{
    public class MyTask
    {
        public int Id { get; set; }
        public string? TaskName { get; set; }
        public bool IsCompleted { get; set; }
        public DateTime DueDate { get; set; }
        public DateTime CreatedDate { get; set; }
        public string? CreatedBy { get; set; }
    }
}
"

3.2 Another common entity template (Name + Description + DateCreated)
"
using System;

namespace YourProjectName.Domain
{
    public class Category
    {
        public int Id { get; set; }
        public string? Name { get; set; }
        public string? Description { get; set; }
        public DateTime DateCreated { get; set; }
    }
}
"

3.3 Entity with foreign key pattern (lab style)
Example: Task belongs to Category
"
using System;

namespace YourProjectName.Domain
{
    public class MyTask
    {
        public int Id { get; set; }
        public string? TaskName { get; set; }
        public bool IsCompleted { get; set; }
        public DateTime DueDate { get; set; }
        public int CategoryId { get; set; }
        public Category? Category { get; set; }
    }
}
"
============================================================

4) ADDING METHODS IN ENTITIES (BUSINESS LOGIC)

4.1 HasExpired() example (no parameters, returns bool)

using System;

namespace YourProjectName.Domain
{
    public class MyTask
    {
        public int Id { get; set; }
        public string? TaskName { get; set; }
        public bool IsCompleted { get; set; }
        public DateTime DueDate { get; set; }
        public DateTime CreatedDate { get; set; }
        public string? CreatedBy { get; set; }
        public bool HasExpired()
        {
            return DueDate.Date < DateTime.Now.Date;
        }
    }
}


4.2 Helper: DaysLeft() (returns int)

public int DaysLeft()
{
    return (DueDate.Date - DateTime.Now.Date).Days;
}


4.3 Helper: MarkComplete() (sets bool)

public void MarkComplete()
{
    IsCompleted = true;
}


============================================================
5) DBCONTEXT TEMPLATES (DATA)

You usually get DbContext created automatically when you scaffold Razor Components CRUD.
If you need to add manually or check it, use:

Create/Check: Data/YourProjectContext.cs

using Microsoft.EntityFrameworkCore;
using YourProjectName.Domain;

namespace YourProjectName.Data
{
    public class YourProjectContext : DbContext
    {
        public YourProjectContext(DbContextOptions<YourProjectContext> options)
            : base(options)
        {
        }

        public DbSet<MyTask> MyTask { get; set; } = default!;

        protected override void OnModelCreating(ModelBuilder builder)
        {
            base.OnModelCreating(builder);

            // ApplyConfiguration goes here when you add seeding
            // builder.ApplyConfiguration(new MyTaskSeed());
        }
    }
}


5.1 appsettings.json connection string (SQL Server typical)

{
  "ConnectionStrings": {
    "YourProjectContext": "Server=(localdb)\\mssqllocaldb;Database=YourProjectContext;Trusted_Connection=True;MultipleActiveResultSets=true"
  }
}


5.2 Program.cs register DbContext (lab pattern)

builder.Services.AddDbContext<YourProjectContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("YourProjectContext")));


============================================================
6) MIGRATIONS AND DATABASE COMMANDS (PACKAGE MANAGER CONSOLE)

Open:
Tools -> NuGet Package Manager -> Package Manager Console

6.1 First migration
Add-Migration Initial
Update-Database

6.2 After you add seeding or change entity properties
Add-Migration SeedData
Update-Database

6.3 If migration fails due to multiple DbContexts

Make sure you are selecting the correct Default Project in Package Manager Console

If needed, specify context:
Add-Migration Initial -Context YourProjectContext
Update-Database -Context YourProjectContext
============================================================
7) SEEDING TEMPLATES (CONFIGURATIONS/ENTITIES)

7.1 Seed class template (IEntityTypeConfiguration + HasData)

Create: Configurations/Entities/MyTaskSeed.cs

using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;
using YourProjectName.Domain;
using System;

namespace YourProjectName.Configurations.Entities
{
    public class MyTaskSeed : IEntityTypeConfiguration<MyTask>
    {
        public void Configure(EntityTypeBuilder<MyTask> builder)
        {
            builder.HasData(
                new MyTask
                {
                    Id = 1,
                    TaskName = "Order Groceries",
                    IsCompleted = true,
                    DueDate = DateTime.Today.AddDays(-1),
                    CreatedDate = DateTime.Today,
                    CreatedBy = "System"
                },
                new MyTask
                {
                    Id = 2,
                    TaskName = "Pick up laundry",
                    IsCompleted = false,
                    DueDate = DateTime.Today.AddDays(7),
                    CreatedDate = DateTime.Today,
                    CreatedBy = "System"
                },
                new MyTask
                {
                    Id = 3,
                    TaskName = "Practical Test 2",
                    IsCompleted = false,
                    DueDate = DateTime.Today,
                    CreatedDate = DateTime.Today,
                    CreatedBy = "System"
                }
            );
        }
    }
}


7.2 Apply seed in DbContext (OnModelCreating)
In Data/YourProjectContext.cs:

using YourProjectName.Configurations.Entities;

protected override void OnModelCreating(ModelBuilder builder)
{
    base.OnModelCreating(builder);
    builder.ApplyConfiguration(new MyTaskSeed());
}


Then run:
Add-Migration SeedMyTask
Update-Database

============================================================
8) SCAFFOLD RAZOR COMPONENTS USING EF (CRUD) (VISUAL STUDIO STEPS)

Goal: Generate Index/Create/Edit/Details/Delete pages for each entity

Step-by-step (lab style):

Right-click the Pages folder (or Components/Pages depending on your template)

Add -> New Scaffolded Item…

Select: Razor Component using Entity Framework (CRUD)

Template: CRUD

Model class: choose your entity (MyTask)

Data context class:

First time: click + to create a new DbContext, accept the generated name

Next entities: reuse the SAME DbContext (do not create a new one)

Database provider: SQL Server (if lab uses SQL Server)

Click Add

After scaffolding:

You should see a new folder like: Components/Pages/MyTaskPages/ or Pages/MyTaskPages/

It contains: Index.razor, Create.razor, Edit.razor, Details.razor, Delete.razor

DbContext is updated with DbSet<MyTask>

Repeat the same steps for ALL entities.

<img width="1026" height="887" alt="image" src="https://github.com/user-attachments/assets/fe4ecbc1-3655-4e48-b9b4-36307da70406" />
<img width="735" height="953" alt="image" src="https://github.com/user-attachments/assets/396b53bd-e37e-4064-baf0-07cc3b7a2eec" />
<img width="736" height="952" alt="image" src="https://github.com/user-attachments/assets/016e56af-7491-41be-9d9f-36db7778f9e5" />
<img width="732" height="952" alt="image" src="https://github.com/user-attachments/assets/d8ae5169-b86c-4bf7-881e-bba3eba7dbe4" />


============================================================

9) IDENTITY SCAFFOLD (REGISTER/LOGIN) (VISUAL STUDIO STEPS)

Goal: Add user registration + login pages

Step-by-step:

Right-click Project

Add -> New Scaffolded Item…

Select: Identity

Choose the pages you need:

Account/Login

Account/Register
(Some labs scaffold all Identity pages. Follow your lab instruction.)

Select or create Identity DbContext (IdentityContext / ApplicationDbContext)

Click Add

After scaffolding you should see:
Components/Account/Pages/Login.razor
Components/Account/Pages/Register.razor

Program.cs checklist:

app.UseAuthentication();

app.UseAuthorization();

Run and test:

Register an account

Login with the account

============================================================

10) AUTH UI PATTERNS (NAVMENU + AUTHORIZE ATTRIBUTE)

10.1 NavMenu.razor pattern (AuthorizeView)
Show Login/Register when not signed in, show Email/Logout when signed in.

Pseudo-pattern:

<AuthorizeView>
  <Authorized>
    <!-- Show user name and logout -->
  </Authorized>
  <NotAuthorized>
    <!-- Show Register and Login -->
  </NotAuthorized>
</AuthorizeView>


10.2 Secure a Razor page (role or authenticated users)
At top of page:

@using Microsoft.AspNetCore.Authorization
@attribute [Authorize]


Or role-based:

@attribute [Authorize(Roles = "Administrator")]
============================================================

11) WEB API SETUP (PROGRAM.CS + SCAFFOLD CONTROLLER)

11.1 Program.cs enable controllers
Add:

builder.Services.AddControllers();


And later before app.Run():

app.MapDefaultControllerRoute();


11.2 Scaffold API Controller with actions using Entity Framework (VS steps)

Right-click Controllers folder

Add -> Controller

Choose API

Select: API Controller with actions, using Entity Framework

Model class: MyTask

DbContext class: YourProjectContext (the same one that has DbSet<MyTask>)

Add

Result:

A controller like MyTasksController.cs with GET, POST, PUT, DELETE

Common route:

/api/mytasks

============================================================
12) COMMON ERRORS AND FIXES (FAST)

A) “Cannot resolve DbContext” during scaffolding

You created multiple DbContexts. Reuse the first one for all scaffolds.

Check which context contains DbSet<MyTask>.

B) “No such table AspNetUsers” or login fails

You did not run Update-Database after Identity scaffolding.

Run Add-Migration IdentityInit (or similar) then Update-Database.

C) “HasData requires key values”

Ensure every seeded entity has Id set (1,2,3 etc).

D) Date comparisons wrong for “today”

Use DueDate.Date < DateTime.Now.Date (strictly less than)

E) API 404

Forgot builder.Services.AddControllers() or app.MapDefaultControllerRoute()

============================================================
13) QUICK COPY-PASTE SNIPPETS PACK

13.1 MyTask minimal entity

using System;

namespace YourProjectName.Domain
{
    public class MyTask
    {
        public int Id { get; set; }
        public string? TaskName { get; set; }
        public bool IsCompleted { get; set; }
        public DateTime DueDate { get; set; }
        public DateTime CreatedDate { get; set; }
        public string? CreatedBy { get; set; }
    }
}


13.2 MyTask with HasExpired()

using System;

namespace YourProjectName.Domain
{
    public class MyTask
    {
        public int Id { get; set; }
        public string? TaskName { get; set; }
        public bool IsCompleted { get; set; }
        public DateTime DueDate { get; set; }
        public DateTime CreatedDate { get; set; }
        public string? CreatedBy { get; set; }

        public bool HasExpired()
        {
            return DueDate.Date < DateTime.Now.Date;
        }
    }
}


13.3 MyTaskSeed

using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;
using YourProjectName.Domain;
using System;

namespace YourProjectName.Configurations.Entities
{
    public class MyTaskSeed : IEntityTypeConfiguration<MyTask>
    {
        public void Configure(EntityTypeBuilder<MyTask> builder)
        {
            builder.HasData(
                new MyTask
                {
                    Id = 1,
                    TaskName = "Order Groceries",
                    IsCompleted = true,
                    DueDate = DateTime.Today.AddDays(-1),
                    CreatedDate = DateTime.Today,
                    CreatedBy = "System"
                },
                new MyTask
                {
                    Id = 2,
                    TaskName = "Pick up laundry",
                    IsCompleted = false,
                    DueDate = DateTime.Today.AddDays(7),
                    CreatedDate = DateTime.Today,
                    CreatedBy = "System"
                },
                new MyTask
                {
                    Id = 3,
                    TaskName = "Practical Test 2",
                    IsCompleted = false,
                    DueDate = DateTime.Today,
                    CreatedDate = DateTime.Today,
                    CreatedBy = "System"
                }
            );
        }
    }
}


13.4 ApplyConfiguration in DbContext

using YourProjectName.Configurations.Entities;

protected override void OnModelCreating(ModelBuilder builder)
{
    base.OnModelCreating(builder);
    builder.ApplyConfiguration(new MyTaskSeed());
}


13.5 PMC Commands

Add-Migration Initial
Update-Database
Add-Migration SeedMyTask
Update-Database


13.6 Enable controllers (Program.cs)

builder.Services.AddControllers();
app.MapDefaultControllerRoute();


END OF README
