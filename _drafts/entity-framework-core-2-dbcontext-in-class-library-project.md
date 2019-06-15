---
layout: single
title:  "Adding DbContext To a Class Library Project"
date:   2018-09-15 19:18:00 -0500
categories: Entity Framework Core 2
comments: true
tags: Entity Framework Core 2
---

On any project larger than XXXXXXX you are probably going to want to move your data access layer to it's own project.  Doing this in EF Core 2 was easier than I remember from using V1 a year or so ago.

On an empty project, we first need to add the EF Core NuGet packages.  

>Microsoft.EntityFrameworkCore
>Microsoft.EntityFrameworkCore.SqlServer
>Microsoft.EntityFrameworkCore.Tools

Then Let's start with a simple `Ticket` object:

    public class Ticket : BaseEntity
    {
        public Guid Id { get; set; }
        public string Title { get; set; }
        public string Description { get; set; }
    }

And add a `DbContext` to work with.  

    public class TicketContext : DbContext
    {
        public TicketContext(DbContextOptions options) : base(options)
        {
        }

        public DbSet<Ticket> Tickets { get; set; }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder);
            modelBuilder.ApplyConfiguration(new TicketMap());
        }
    }

I prefer to use mapping files over attributes for my data classes.

    public class TicketMap : IEntityTypeConfiguration<Ticket>
    {
        public void Configure(EntityTypeBuilder<Ticket> builder)
        {
            builder.HasKey(x => x.Id);
                
            builder.Property(x => x.Title)
                .IsRequired()
                .HasMaxLength(255)
                .IsUnicode();
        }
    }

And in the old world of EF6 that would be enough to get migrations set up, but attempting to add the initial migration I ran into this error (there's so much less swearing and single finger hand gestures when you remember to `CD` to the directory where your project lives...)

> dotnet ef migrations add InitialCreate
The EF Core tools version '2.1.1-rtm-30846' is older than that of the runtime '2.1.3-rtm-32065'. Update the tools for the latest features and bug fixes.
Unable to create an object of type 'TicketContext'. Add an implementation of 'IDesignTimeDbContextFactory<TicketContext>' to the project, or see https://go.microsoft.com/fwlink/?linkid=851728 for additional patterns supported at design time.

Microsoft has *really* stepped up thier documentation game over the past couple of years:

>You can also tell the tools how to create your DbContext by implementing the >IDesignTimeDbContextFactory<TContext> interface: If a class implementing this interface is found in >either the same project as the derived DbContext or in the application's startup project, the tools >bypass the other ways of creating the DbContext and use the design-time factory instead.

Following the example provided, we can add a class implementing `IDesignTimeDbContextFactory<TicketContext>` to our project

    public class TicketContextFactory : IDesignTimeDbContextFactory<TicketContext>
    {
        public TicketContext CreateDbContext(string[] args)
        {
            var optionsBuilder = new DbContextOptionsBuilder<TicketContext>();
            optionsBuilder.UseSqlServer("Server=.\\SQLEXPRESS;Database=Tickx;Trusted_Connection=True;MultipleActiveResultSets=true");

            return new TicketContext(optionsBuilder.Options);
        }
    }

Running the migrations command now works as expected

> dotnet ef migrations add InitialCreate
The EF Core tools version '2.1.1-rtm-30846' is older than that of the runtime '2.1.3-rtm-32065'. Update the tools for the latest features and bug fixes.
Done. To undo this action, use 'ef migrations remove'

Other ides
dotnet ef --help
dotnet ef database --help
