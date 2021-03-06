---
layout: single
title:  "Dependency Injection"
date:   2020-01-09 06:30:00 -0500
categories: Practices
comments: true
tags: dependency injection
---
> Q:  What is dependency injection?

Dependency injection is providing the dependencies to a class instead of having that class be responsible for creating it's own instances

    public class Service 
    {
       private DependencyOne _dependency1;
       private DependencyTwo _dependency2;
       
       public Service() 
       {
    	_dependency1 = new DependencyOne(new Logger(), new DatabaseClass())
    	_dependency2 = new DependencyTwo();
       }
    }
	
With Dependency Injection this becomes

	public class Service 
	{
	   private DependencyOne _dependency1;
	   private DependencyTwo _dependency2;
	   
	   public Service(DependencyOne dependency1, Dependency2 dependency2) 
	   {
		_dependency1 = dependency1;
		_dependency2 = dependency2;
	   }
	}
	
We can see the first benefit immediately:  If  the constructor of `DependencyOne` changes to require additional child dependencies, we will not need to change our `Service` class like we would in the first example.

>Q:  OK, but it looks like all I did was bump my "new" statement up a level.  Now instead of `new Service()`, I need to create `new Service(new DependencyOne(), new DependencyTwo()`

At some point you are going to have what's known as a "composition root" where you need to make the decisions of how new instances are created.  To help with this, we can use dependency injection frameworks such as Lamar, StructureMap or AutoFac.  These frameworks are known as "containers" and they are responsible for creating the "dependency graph" (there are a lot of terms associated with this concept, any they will start to make sense as you get more comfortable with it).

>Q:  Why do I want to do this?

There is a design guideline known as Dependency Inversion that states that 

	High-level modules should not depend on low-level modules. Both should depend on abstractions.
	Abstractions should not depend on details. Details should depend on abstractions.
	
This is part of a fairly well know set of principles known as [SOLID]([https://en.wikipedia.org/wiki/SOLID](https://en.wikipedia.org/wiki/SOLID)) and Dependency Injection is a way of implementing Dependency Inversion.

>Q:  That's very vague.

Consider our second example above.  We are using Dependency Injection but not Dependency Inversion.  Dependency Inversion would look like this

	public class Service 
	{
	   private IDependencyOne _dependency1;
	   private IDependencyTwo _dependency2;
	   
	   public Service(IDependencyOne dependency1, IDependencyTwo dependency2) 
	   {
		_dependency1 = dependency1;
		_dependency2 = dependency2;
	   }
	}

Where we are now passing in Interfaces instead of implementations.  We have "inverted" the dependencies and our service class now depends on abstractions (the Interfaces) instead of concrete implementations.  The `Service` class is now only very loosely coupled to it's dependencies.  The creator of the `Service` class is now responsible for providing those dependencies.

> Q: That kind of makes sense, but what are the benefits?

The primary benefit is loose coupling.  It is much cleaner to depend on an `IWeatherService` that defines what a weather service should do than on a concrete implementation like  `SimpleWeatherService` or `WeatherChannelWeatherService`.  
When we only care about the abstraction, we can easily swap in any implementation of `IWeatherService` without having to make any changes to the class consuming this.

Additionally, there are some other benefits such as testable code.   Dependency Inversion allows you to break apart complex problems into small isolated units that can be tested individually.
	
You also end up with a substantial reduction in "new".  When using a container, object initialization is taken care of for you.  Without a container, If you had `SimpleWeatherService(Logger)` and you added `SimpleWeatherService(Logger, Location)` you would need to update every usage.  When the container takes care of this for you, you won't find yourself updating multiple files across your application.

>Q: When would I want to use this kind of approach vs sticking to what I've been doing?

Generally, you'd switch to this approach when you start feeling any type of discomfort from managing dependencies.  If you have a small application, there's hardly every a good case for adding additional complexity, but if that application grows and you want to take advantage of the benefits listed above it might be time to switch.

Additionally, if you are doing any kind of test driven development you will probably start with this approach.

> Q: You mentioned something about frameworks earlier.  Do I need one?  And how do I pick one?

Good news: You don't need one

Bad news:  You should probably use one.

You can definitely do all of this manually, but given the sheer variety of things that an IoC container  can do and the number of things they simplify it's probably better to use something that's been tested and used in multiple differing production environments by other people.  For example - how should lifecycles be managed?  Do you want a single instances per HTTP request?  Or a new instance for every class that needs one?  Or do you just want a single instance to be created for the entire application?

As for how to pick a container:  Microsoft provides one with .net core that will work with web or console applications.  It's a solid choice, but it does lack some of the more advanced features that other containers provide.  However, you may never need these features so it's probably better to start simple.  The "wiring up" of these containers is generally done during application start up, so switching one out for another should usually be a relatively quick task.

> Q:  There is a lot of jargon and acronyms.

There is, and a lot of the [terms](https://stackoverflow.com/a/6551303/190592) being used around this concept are interchangeable 

>Q:  This sounds interesting.  Can you show me some examples?

[Yes.](https://github.com/joelowrance/DIDemo)

