---
layout: single
title:  "Entity Framework 6 Refresher: Seed Data"
date:   2018-05-22 20:30:00 -0500
categories: entity-framework
---

"Seed Data" is data that is inserted into the database when migrations are run.  Generally, this would be relatively static look-up data such as `States` or `OrderStatuses`. but it can be anything you want.

Using the example of `Person` in the previous post, we can use the seed functionality to created some records.  To do this, open `Configuration.cs` in the `Migrations` folder.  There will be `Seed` method.

{% highlight csharp %}
protected override void Seed(LearnMappings.LearnMappingsContext context)
{
    //  This method will be called after migrating to the latest version.

    //  You can use the DbSet<T>.AddOrUpdate() helper extension method 
    //  to avoid creating duplicate seed data.
    var p1 = new Person() {FirstName = "John", LastName = "Smith", PersonId = 1};
    var p2 = new Person() {FirstName = "Jane", LastName = "Doe", PersonId = 2};
    context.People.AddOrUpdate(p1);
    context.People.AddOrUpdate(p2);
}
{% endhighlight %}

As noted in the comments the ` DbSet<T>.AddOrUpdate()` will insert new data or update existing items with any changes.

`PersonId`, which is an identity column, can be ignored or set explicitly.

The seed method can be run via `update-database` in the PMC even without a pending migraiton.