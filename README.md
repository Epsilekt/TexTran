# TexTran

TexTran is a project that contains `T4 templates` to generate classes from simple definitions. I started this project to discover what `T4` has to offer and how I can use this in future projects. 

## Frameworks

- .NET Core 2.2
- Entity Framework Core 2 
- TTExecuter 1.9
- AutoMapper 8.1.0

## Usage

Define classes in the definition files, and build the solution choose `Build - Transform All T4 Templates`.
Add `ViewModels` to the project and a `AutoMapper Mapping Profile` is also created for the `ViewModel` and the matching `Entity`.

### Entity example

Add a definition for a entity to the definition file (`Entities.definition`). Save the file, build the solution and following files are generated:
- {Entity}.cs inside `TexTran.Data.Abstractions\Entities`
- {Entity}Repository.cs inside `TexTran.Data.Access\Repositories`
- I{Entity}Repository.cs inside `TexTran.Data.Abstractions\Interfaces`
- DbContext.cs + DbSet<{Entity}> inside `TexTran.Data.Access\Context` 
- If you add a `IEntityTypeConfiguration<{Entity}>` this will also be added to the `OnModelCreating()` of the generated `DbContext` 

Example Entity definition file:

``` txt
User // Description goes here
	FirstName : string
	LastName : string
	EmailAddress : string
	DateOfBirth : DateTimeOffset
	Gender: Gender
	Group : Group
	NullableNumber : int?

Group
	Name : string
	Members : List<User>

```

Generated Entity example:

``` csharp
// This file is auto generated. Changes to this file will be lost!
using System;
using System.Collections.Generic;
using TextTran.Transformations.Enums;
	
namespace TexTran.Data.Abstractions.Entities
{
	/// <summery>
	/// Description goes here
	/// </summary>");
	public class User : BaseEntity
	{
		public string FirstName  { get; set; }

		public string LastName  { get; set; }

		public string EmailAddress  { get; set; }

		public DateTimeOffset DateOfBirth  { get; set; }

		public Gender Gender { get; set; }

		public Group Group  { get; set; }

		public int? NullableNumber  { get; set; }

	}
}
```
Generated Repository:

``` csharp
//This code is auto generated. Changes to this file will be lost!
using System;
using System.Linq;
using TexTran.Data.Abstractions.Interfaces;
using TexTran.Data.Abstractions.Entities;
using TextTran.Data.Access.Context;

namespace TexTran.Data.Access.Repositories
{
	public class UserRepository: IUserRepository
	{
		private readonly TexTranContext _context;

		public UserRepository(TexTranContext context)
		{
			_context = context;
		}

		public void Add(User user)
		{
			_context.Users.Add(user);
		}

		public void Delete(User user)
		{
			_context.Users.Remove(user);
		}

		public void Edit(User user)
		{
			var editedEntity = _context.Users.FirstOrDefault(e => e.Id == user.Id);
			editedEntity = user;
		}

		public User GetById(Guid id)
		{
			return _context.Users.FirstOrDefault(e => e.Id == id);
		}

		public void SaveChanges() => _context.SaveChanges();
	}
}
```

Generated DbContext :
``` csharp
// This code is auto generated. Changes to this file will be lost!
using Microsoft.EntityFrameworkCore;
using TexTran.Data.Abstractions.Entities;
using TexTran.Data.Access.Configurations;

namespace TextTran.Data.Access.Context
{
	public class TexTranContext : DbContext
	{
		public TexTranContext(DbContextOptions options) 
			: base(options) 
		{
		}

		protected override void OnModelCreating(ModelBuilder modelBuilder)
		{
			modelBuilder.ApplyConfiguration(new UserConfiguration());
		}

		public DbSet<Group> Groups { get; set; }
		public DbSet<User> Users { get; set; }
	}
}
```

After adding more content to the definition files, just run `Build > Transform all T4 templates` or build the solution to re-generate all code. If you add an `Entity`, there will also be a `DbSet<Entity>` added to the generated DbContext. `EntityTypeConfigurations` have to be written manually, but these are also added to the `DbContext` when transforming the templates or just building the solution. 

### Set up DB

You can set the DbContextName in `TexTran.Transformer.TransformSetup.ttinclude`.
Add your entities to the definition files, add `EntityTypeConfiguration<Entity>` and run a `Add-Migration Initial` command in the Package Manager Console followed by `Update-Database` to create the database. Don't forget to create a migration every time you change the database model.
