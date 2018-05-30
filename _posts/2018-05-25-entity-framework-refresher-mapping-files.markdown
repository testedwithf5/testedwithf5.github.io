---
layout: single
title:  "Entity Framework 6 Refresher: Mapping Files"
date:   2018-05-22 20:30:00 -0500
categories: entity-framework
comments: true
tags: entity-framework
---


Entity Framework can determine a lot of things about your model on it's own, but at some point you need to provide some information about your classes that it has no way of knowing.  There are three ways to do this:  Attributes, via code in the DbContext itself and in separate mapping files.

Using attributes, we can decorate the class with metadata.  

{% highlight csharp %}
[Table("HobbyHorse")]
public class Hobby
{
    [Key]
    public int HobbyId { get; set; }

    [Required]
    [MaxLength(25)]
    public string HobbyName { get; set; }
}
{% endhighlight %}

This shows that we use a table name of "HobbyHorse", that `HobbyId` is the primary key and that `HobbyName` is required and can be a maximum of 25 characters.   This creates the following migration:

{% highlight csharp %}
public override void Up()
{
    CreateTable(
        "dbo.HobbyHorse",
        c => new
            {
                HobbyId = c.Int(nullable: false, identity: true),
                HobbyName = c.String(nullable: false, maxLength: 25),
            })
        .PrimaryKey(t => t.HobbyId);
}
{% endhighlight %}

We can also configure the `Hobby` class in the DbContext:

{% highlight csharp %}
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    modelBuilder.Entity<Hobby>()
        .ToTable("HobbyHorse")
        .HasKey(x => x.HobbyId)
        .Property(x => x.HobbyName).IsRequired().HasMaxLength(25);

    base.OnModelCreating(modelBuilder);
}
{% endhighlight %}

The migration created from this is identical to the migration created when using attributes.

Finally (and preferably, IMO), we can configure `Hobby` in it's own mapping file.

{% highlight csharp %}
public class HobbyMap : EntityTypeConfiguration<Hobby>
{
    public HobbyMap()
    {
        HasKey(x => x.HobbyId);
        Property(x => x.HobbyName).IsRequired().HasMaxLength(25);
    }
}
{% endhighlight %}

This mapping file needs to be wired up in the DbContext:

{% highlight csharp %}
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    modelBuilder.Configurations.Add(new HobbyMap());
    base.OnModelCreating(modelBuilder);
}
{% endhighlight %}

Again, this creates an identical migration.
