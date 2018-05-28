---
layout: single
title:  "Entity Framework 6 Refresher: One To One"
date:   2018-05-29 09:30:00 -0500
categories: entity-framework
comments: true
---

Often a one-to-one relationship is required.   Mapping these in EF is very similar to mapping any other relationships.  For this example each person can have one car, and each car must belong to one and only one person.

{% highlight csharp %}
public class Person
{
    public int PersonId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public virtual Car Car { get; set; }
}

public class Car
{
    public int CarId { get; set; }
    public string Make { get; set; }
    public virtual Person Person { get; set; }
}

public class PersonMap : EntityTypeConfiguration<Person>
{
    public PersonMap()
    {

        HasOptional(x => x.Car)
            .WithRequired(x => x.Person);
    }
}
{% endhighlight %}

This doesn't work.  The generated migration wants `CarId` to be a foreign key.

{% highlight csharp %}
public override void Up()
{
    CreateTable(
        "dbo.Cars",
        c => new
            {
                CarId = c.Int(nullable: false),
                Make = c.String(),
            })
        .PrimaryKey(t => t.CarId)
        .ForeignKey("dbo.People", t => t.CarId)
        .Index(t => t.CarId);
}
{% endhighlight %}

If we update the mapping to include a key name, we get the correct result.  A column named `PersonId` is created even though it is not specified in our model.

{% highlight csharp %}
public class PersonMap : EntityTypeConfiguration<Person>
{
    public PersonMap()
    {

        HasOptional(x => x.Car)
            .WithRequired(x => x.Person)
            .Map(x => x.MapKey("PersonId"));
    }
}

public override void Up()
{
    CreateTable(
        "dbo.Cars",
        c => new
            {
                CarId = c.Int(nullable: false, identity: true),
                Make = c.String(),
                PersonId = c.Int(nullable: false),
            })
        .PrimaryKey(t => t.CarId)
        .ForeignKey("dbo.People", t => t.PersonId)
        .Index(t => t.PersonId);
}
{% endhighlight %}

An alternative approach to this is to use  `PersonId` as the primary key on the `cars` table.  This more than likely will not make much sense in your business domain, but it is possible.

{% highlight csharp %}
public class Car
{
    public int PersonId { get; set; }
    public string Make { get; set; }
    public virtual Person Person { get; set; }
}

public class PersonMap : EntityTypeConfiguration<Person>
{
    public PersonMap()
    {

        HasOptional(x => x.Car)
            .WithRequired(x => x.Person);
    }
}

public class CarMap : EntityTypeConfiguration<Car>
{
    public CarMap()
    {
        HasKey(x => x.PersonId);
    }
}

public override void Up()
{
    CreateTable(
        "dbo.Cars",
        c => new
            {
                PersonId = c.Int(nullable: false),
                Make = c.String(),
            })
        .PrimaryKey(t => t.PersonId)
        .ForeignKey("dbo.People", t => t.PersonId)
        .Index(t => t.PersonId);
    
}
{% endhighlight %}