---
layout: single
title:  "Entity Framework 6 Refresher: Many To Many Referencing Self"
date:   2018-05-28 09:30:00 -0500
categories: entity-framework
comments: true
published: true
---

Occasionally a many-to-many relationship needs to be established with an entity pointing to itself;  In the case of the small domain we are working with, we can examine this with the concept of a `Family` - `Person` referencing `Person`.   

{% highlight csharp %}
public class Person
{
    public int PersonId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public virtual ICollection<Person> Family { get; set; }
}
{% endhighlight %}

In this case, we are going to need mapping information as the default migration built for this makes each person belong to one, and only one family (we're doing a learning exercise here, ok?)

{% highlight csharp %}
public override void Up()
{
    AddColumn("dbo.People", "Person_PersonId", c => c.Int());
    CreateIndex("dbo.People", "Person_PersonId");
    AddForeignKey("dbo.People", "Person_PersonId", "dbo.People", "PersonId");
}
{% endhighlight %}

A mapping that you might think would work will cause an error `The navigation property 'Family' declared on type 'LearnMappings.Person' cannot be the inverse of itself.`

{% highlight csharp %}
public PersonMap()
{
    HasMany(x => x.Family)
        .WithMany(x => x.Family);
}
{% endhighlight %}

In this case we need to leave `WithMany` empty.  We also need to setup the `Map` information as the defaults will be kind of funky (`PersonPerson` with columns called `Person_PersonId1` and `Person_PersonId`)

{% highlight csharp %}
public class PersonMap : EntityTypeConfiguration<Person>
{
    public PersonMap()
    {
        HasMany(x => x.Family)
            .WithMany()
            .Map(x =>
            {
                x.ToTable("Family");
                x.MapLeftKey("PersonId");
                x.MapRightKey("OtherPersonId");
            });
    }
}
{% endhighlight %}

This gets us a migration that looks like this.

{% highlight csharp %}
public override void Up()
{
    CreateTable(
        "dbo.PersonPersons",
        c => new
            {
                Person_PersonId = c.Int(nullable: false),
                Person_PersonId1 = c.Int(nullable: false),
            })
        .PrimaryKey(t => new { t.Person_PersonId, t.Person_PersonId1 })
        .ForeignKey("dbo.People", t => t.Person_PersonId)
        .ForeignKey("dbo.People", t => t.Person_PersonId1)
        .Index(t => t.Person_PersonId)
        .Index(t => t.Person_PersonId1);
    
}
{% endhighlight %}
