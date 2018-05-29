---
layout: single
title:  "Entity Framework 6 Refresher: Changing a nullable field to non-nullable"
date:   2018-05-30 09:30:00 -0500
categories: entity-framework
comments: true
---

You may not know this, but on rare occasions business rules and requirements change.  Let's consider a case where our `Person` class has a `Weight` and a `Modified` field.

{% highlight csharp %}
public class Person
{
    public int PersonId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public int? Weight { get; set; }
    public DateTimeOffset Modified { get; set; }
}

//migration
public override void Up()
{
    AddColumn("dbo.People", "Weight", c => c.Int());
    AddColumn("dbo.People", "Modified", c => c.DateTimeOffset(precision: 7));
}
{% endhighlight %}

After applying this migration, data in our database looks like this.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/ef-refresh-nullable-fields.png" alt="">

Let's imagine that everything runs smoothly for several days and then all of a sudden a project manager or a process manager or a product manager calls and makes a much bigger deal than he needs to about `Weight` and `Modified` being critical.  "No big deal" you think.  And off to work you go to make these fields non-nullable.

{% highlight csharp %}
public class PersonMap : EntityTypeConfiguration<Person>
{
    public PersonMap()
    {
        Property(x => x.Modified).IsRequired();
        Property(x => x.Weight).IsRequired();
    }
}

public override void Up()
{
    AlterColumn("dbo.People", "Weight", c => c.Int(nullable: false));
    AlterColumn("dbo.People", "Modified", c => c.DateTimeOffset(nullable: false, precision: 7));
}
{% endhighlight %}


In them PMC, you enter `update-database` and get a nice long error starting with `System.Data.SqlClient.SqlException (0x80131904): Cannot insert the value NULL into column 'Weight', table 'RefresherContext.dbo.People'; column does not allow nulls. UPDATE fails.`.   Oh.  If we run `update-database -script` we see the following.  The existing records all have null for `Weight` and `Modified` and this migration is explicitly saying that is not allowed.

{% highlight sql %}
DECLARE @var0 nvarchar(128)
SELECT @var0 = name
FROM sys.default_constraints
WHERE parent_object_id = object_id(N'dbo.People')
AND col_name(parent_object_id, parent_column_id) = 'Weight';
IF @var0 IS NOT NULL
    EXECUTE('ALTER TABLE [dbo].[People] DROP CONSTRAINT [' + @var0 + ']')
ALTER TABLE [dbo].[People] ALTER COLUMN [Weight] [int] NOT NULL
DECLARE @var1 nvarchar(128)
SELECT @var1 = name
FROM sys.default_constraints
WHERE parent_object_id = object_id(N'dbo.People')
AND col_name(parent_object_id, parent_column_id) = 'Modified';
{% endhighlight %}

No problem.  We can manually update the migration to update the existing values.

{% highlight csharp %}
public override void Up()
{
    Sql("UPDATE dbo.People SET Weight=0 WHERE Weight IS NULL");
    Sql("UPDATE dbo.People SET Modified=GetDate() WHERE Modified IS NULL");
    AlterColumn("dbo.People", "Weight", c => c.Int(nullable: false));
    AlterColumn("dbo.People", "Modified", c => c.DateTimeOffset(nullable: false, precision: 7));
}
{% endhighlight %}

The `Sql` method here gets us a script that looks something like this.

{% highlight sql %}
UPDATE dbo.People SET Weight=0 WHERE Weight IS NULL
UPDATE dbo.People SET Modified=GetDate() WHERE Modified IS NULL
DECLARE @var0 nvarchar(128)
SELECT @var0 = name
FROM sys.default_constraints
WHERE parent_object_id = object_id(N'dbo.People')
AND col_name(parent_object_id, parent_column_id) = 'Weight';
IF @var0 IS NOT NULL
    EXECUTE('ALTER TABLE [dbo].[People] DROP CONSTRAINT [' + @var0 + ']')
ALTER TABLE [dbo].[People] ALTER COLUMN [Weight] [int] NOT NULL
DECLARE @var1 nvarchar(128)
SELECT @var1 = name
FROM sys.default_constraints
WHERE parent_object_id = object_id(N'dbo.People')
AND col_name(parent_object_id, parent_column_id) = 'Modified';
{% endhighlight %}