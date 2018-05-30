---
layout: single
title:  "Entity Framework 6 Refresher: Initial Setup"
date:   2018-05-22 20:30:00 -0500
categories: entity-framework
comments: true
tags: entity-framework
---
It has been a long time since I used Entity Framework on anything other than an brownfield project where most work consisted of adding or removing the occasional field.  This series of posts is intented as a refresher as I relearn some concepts.


1. [Initial Setup]({{ site.baseurl }}{% post_url 2018-05-23-entity-framework-refresher %})
2. [Seed Data]({{ site.baseurl }}{% post_url 2018-05-24-entity-framework-seed-data %})
3. [Mapping Files]({{ site.baseurl }}{% post_url 2018-05-25-entity-framework-refresher-mapping-files %})
4. [One-to-many mapping]({{ site.baseurl }}{% post_url 2018-05-26-entity-framework-refresher-one-to-many %})
5. [Many-to-many mapping]({{ site.baseurl }}{% post_url 2018-05-27-entity-framework-refresher-many-to-many %})
6. [Many-to-many self mapping]({{ site.baseurl }}{% post_url 2018-05-28-entity-framework-refresher-many-to-many-self %})
7. [One-to-one mapping]({{ site.baseurl }}{% post_url 2018-05-29-entity-framework-refresher-one-to-one %})


Initial setup.

In a new .Net Framwork class library, add a reference to Microsoft.EntityFramework via the NuGet package manager.

Next add a basic class 

{% highlight csharp %}
public class Person
{
    public int PersonId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
}
{% endhighlight %}

Then add a DbContext.

{% highlight csharp %}
public class LearnMappingsContext : DbContext
{
    public DbSet<Person> People { get; set; }
}
{% endhighlight %}


Next add a connection string to the app.config file that was created when you added the NuGet package.

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <configSections>
    <!-- For more information on Entity Framework configuration, visit http://go.microsoft.com/fwlink/?LinkID=237468 -->
    <section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
  </configSections>
  <connectionStrings>
    <add name="LearnMappings" connectionString="Server=.\SQLEXPRESS;Database=LearnMappings;Trusted_Connection=True;" providerName="System.Data.EntityClient" />
  </connectionStrings>
  <entityFramework>
    <defaultConnectionFactory type="System.Data.Entity.Infrastructure.SqlConnectionFactory, EntityFramework" />
    <providers>
      <provider invariantName="System.Data.SqlClient" type="System.Data.Entity.SqlServer.SqlProviderServices, EntityFramework.SqlServer" />
    </providers>
  </entityFramework>
</configuration>
{% endhighlight %}

We are now ready to set up migrations. The first step is to enable them.  In the Package Manager Console (PMC from now on - this can be found under Tools &#124; NuGet Package Manager &#124; Package Manager Console) run the command to add migrations:

`enable-migrations`

This will create a `/Migrations` folder with a file called `configurations.cs`.

Next, create the migration for the person class (this does not need to be named "InitialMigration")

`add-migration "InitialMigration"`

This will generate a migration script to add the `Person` class.

Finally, run `update-database` in the PMC to create the table in your database.


