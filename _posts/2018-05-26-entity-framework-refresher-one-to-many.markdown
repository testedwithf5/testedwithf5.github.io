---
layout: single
title:  "Entity Framework 6 Refresher: One To Many"
date:   2018-05-26 20:30:00 -0500
categories: entity-framework
---

To demonstrate a one-to-many relationship, we can associate `Person` with `Hobby`.   This means that each hobby will belong to one and only one person (our people are hipsters who quickly move on at the first whiff of someone else enjoying what they enjoy).

All examples were run using the following test.

{% highlight csharp %}
[Test]
public void HasMany()
{
    int id;
    using (var cx = new LearnMappingsContext())
    {
        var person = new Person {FirstName = "Bob", LastName = "Maxwell"};
        var gardening = new Hobby {HobbyName = "Gardening"};
        var running = new Hobby {HobbyName = "Running"};

        person.Hobbies = new List<Hobby>
        {
            gardening,
            running
        };

        cx.People.Add(person);
        cx.SaveChanges();
        id = person.PersonId;
        Assert.Greater(id, 0);
    }

    using (var cx = new LearnMappingsContext())
    {
        var p = cx.People.FirstOrDefault(x => x.PersonId == id);
        Assert.AreEqual(2, p.Hobbies.Count);
    }


    using (var cx = new LearnMappingsContext())
    {
        var h = cx.Hobbies.FirstOrDefault();
    }
}
{% endhighlight %}

First, we can add a `Hobbies` property to `Person` and make no changes to any mapping files.


{% highlight csharp %}
public class Person
{
    public int PersonId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public virtual ICollection<Hobby> Hobbies { get; set; }
}
{% endhighlight %}

This creates the following migration:

{% highlight csharp %}
public override void Up()
{
    AddColumn("dbo.Hobbies", "Person_PersonId", c => c.Int());
    CreateIndex("dbo.Hobbies", "Person_PersonId");
    AddForeignKey("dbo.Hobbies", "Person_PersonId", "dbo.People", "PersonId");
}

{% endhighlight %}

After running the migration and our test, we see the following the database.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/ef-refresh-one-to-many1.png" alt="">

Next, we can try adding this relationship via the  mapping files.

{% highlight csharp %}
public class PersonMap : EntityTypeConfiguration<Person>
{
    public PersonMap()
    {
        HasMany(x => x.Hobbies);
    }
}
{% endhighlight %}

Unsurpisingly, this creates a migration that is identical to the one above.

{% highlight csharp %}
public override void Up()
{
    AddColumn("dbo.Hobbies", "Person_PersonId", c => c.Int());
    CreateIndex("dbo.Hobbies", "Person_PersonId");
    AddForeignKey("dbo.Hobbies", "Person_PersonId", "dbo.People", "PersonId");
}
{% endhighlight %}

While both of these work, `Person_PersonId` isn't the column name I would like; `PersonId` would be ideal.

If we simply add `PersonId` to `Hobby`, we get the database structure that we would expect.

{% highlight csharp %}
public class Hobby
{
    public int HobbyId { get; set; }
    public string HobbyName { get; set; }
    public int PersonId { get; set; }
}

//From the migraiton 
public override void Up()
{
    AddColumn("dbo.Hobbies", "PersonId", c => c.Int(nullable: false));
    CreateIndex("dbo.Hobbies", "PersonId");
    AddForeignKey("dbo.Hobbies", "PersonId", "dbo.People", "PersonId", cascadeDelete: true);
}
{% endhighlight %}

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/ef-refresh-one-to-many2.png" alt="">

However, I have a personal preference for a more explicit approach.  The following mapping will produce an identical migration with the added benefit of leaving nothing ambiguous.

{% highlight csharp %}
public class PersonMap : EntityTypeConfiguration<Person>
{
    public PersonMap()
    {
        HasMany(x => x.Hobbies)
            .WithRequired()
            .HasForeignKey(x => x.PersonId);
    }
}
{% endhighlight %}

If we want a `Person` navigation propert on our `Hobby` class, we can update the mappings accordingly, and once again end up with an identical migration.

{% highlight csharp %}
public class Hobby
{
    public int HobbyId { get; set; }
    public string HobbyName { get; set; }
    public int PersonId { get; set; }
    public virtual Person Person { get; set; }
}

public class PersonMap : EntityTypeConfiguration<Person>
{
    public PersonMap()
    {
        HasMany(x => x.Hobbies)
            .WithRequired(x => x.Person)
            .HasForeignKey(x => x.PersonId);
    }
}
{% endhighlight %}

Somwhat interestingly, if we choose to think of our data from the perspective of hobbies instead of people, we can add the mapping to the `Hobby` configuration and still achieve the same result.

{% highlight csharp %}
public class HobbyMap : EntityTypeConfiguration<Hobby>
{
    public HobbyMap()
    {
        HasKey(x => x.HobbyId);
        Property(x => x.HobbyName).IsRequired().HasMaxLength(25);

        HasRequired(x => x.Person)
            .WithMany(x => x.Hobbies)
            .HasForeignKey(x => x.PersonId);
    }
}
{% endhighlight %}
