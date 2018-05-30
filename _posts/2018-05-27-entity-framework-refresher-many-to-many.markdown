---
layout: single
title:  "Entity Framework 6 Refresher: Many To Many"
date:   2018-05-27 09:30:00 -0500
categories: entity-framework
comments: true
tags: entity-framework
---

To demonstrate a many to many relationship, we can look at a two-way relationship between `Person` and `Hobby`.  We start by adding a collection property to each class.

{% highlight csharp %}
public class Person
{
    public int PersonId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public virtual ICollection<Hobby> Hobbies { get; set; }
}

public class Hobby
{
    public int HobbyId { get; set; }
    public string HobbyName { get; set; }
    public virtual ICollection<Person> People { get; set; }
}
{% endhighlight %}

With no mapping file, EF will create a migration that combines the two class's names and an id column for each.

{% highlight csharp %}
public override void Up()
{
    CreateTable(
        "dbo.PersonHobbies",
        c => new
            {
                Person_PersonId = c.Int(nullable: false),
                Hobby_HobbyId = c.Int(nullable: false),
            })
        .PrimaryKey(t => new { t.Person_PersonId, t.Hobby_HobbyId })
        .ForeignKey("dbo.People", t => t.Person_PersonId, cascadeDelete: true)
        .ForeignKey("dbo.Hobbies", t => t.Hobby_HobbyId, cascadeDelete: true)
        .Index(t => t.Person_PersonId)
        .Index(t => t.Hobby_HobbyId);
    
}
{% endhighlight %}

This migration is identical to what would generated were we to use the most simple mapping available - `HasMany(x => x.Hobbies).WithMany(x => x.People)`

Optionally, we can add more mapping information to this to ensure the database is updated to match our needs.  In this case we want to follow a convention of `EntityId` and we would like the table name to be `HobbyPerson`.

{% highlight csharp %}
public PersonMap()
{
    HasMany(x => x.Hobbies)
        .WithMany(x => x.People)
        .Map(map =>
        {
            map.MapLeftKey("PersonId");
            map.MapRightKey("HobbyId");
            map.ToTable("HobbyPerson");
        });
}
{% endhighlight %}

As is always the case with string literals, be careful with spelling.  The following is completely valid, but probably not something that will make a team member sing your praises.

{% highlight csharp %}
public PersonMap()
{
    HasMany(x => x.Hobbies)
        .WithMany(x => x.People)
        .Map(map =>
        {
            map.MapLeftKey("ParsonId");
            map.MapRightKey("HubbyId");
            map.ToTable("PrsonHobbies");
        });
}
{% endhighlight %}

An additional option in many-to-many scenarios is to model the join table.  You may want to store data other than just foreign keys here, or it may just make more sense in your application to work with this table rather than the root elements.  Inserting a record here will take care of inserting the related `Person` or `Hobby` record, should they not exist.  In this case, we are using a composite primary key and <em>must</em> specify this in the mapping file. 

{% highlight csharp %}
public class PersonHobby
{
    public int PersonId { get; set; }
    public int HobbyId { get; set; }
    public DateTime UpdateDate { get; set; }
    public virtual Person Person { get; set; }
    public virtual Hobby Hobby { get; set; }
}

public class PersonHobbyMap : EntityTypeConfiguration<PersonHobby>
{
    public PersonHobbyMap()
    {
        HasKey(x => new {x.PersonId, x.HobbyId});
    }
}

public override void Up()
{
    CreateTable(
        "dbo.PersonHobbies",
        c => new
            {
                PersonId = c.Int(nullable: false),
                HobbyId = c.Int(nullable: false),
                UpdateDate = c.DateTime(nullable: false),
            })
        .PrimaryKey(t => new { t.PersonId, t.HobbyId })
        .ForeignKey("dbo.Hobbies", t => t.HobbyId, cascadeDelete: true)
        .ForeignKey("dbo.People", t => t.PersonId, cascadeDelete: true)
        .Index(t => t.PersonId)
        .Index(t => t.HobbyId);
}
{% endhighlight %}