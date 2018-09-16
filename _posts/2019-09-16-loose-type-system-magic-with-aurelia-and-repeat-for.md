---
layout: single
title:  "Loose Type System Magic With Aurelia's Repeat-For"
date:   2018-09-15 19:18:00 -0500
categories: javascript
comments: true
tags: javascript, aurelia
---

Coming from a background with string static type checking, I am often very happy with the loseness of JavaScript.  Working with this TypeScript interface, and Aurelia controller and related HTML snippet

{% highlight typescript %}
export class ITicketSummary {
  id: number;
  title: string;
  dateOpened: Date;
}
{% endhighlight %}

{% highlight html %}
<div repeat.for="t of tickets">
  <user-autocomplete 
    context.bind="{priority: priority}" 
    on-user-selected.call="userSelected(user, context)">
  </user-autocomplete>
</div>
{% endhighlight %}

{% highlight typescript %}
export class MyController {
  public summaryItems: ITicketSummary[]

  public disableUserSelectForTickt(ticket: ITicketSummary) {
    //PROBLEM:  How do I find the auto complete associated with this ticket
  }
}
{% endhighlight %}

As noted in the **PROBLEM** comment, I needed to access the autocomplete associated with a particular `ITicketSummary`.  


I needed to access the autocomplete for a particular `ticket` inside of a loop.  My first thought was that would need to add an array of `user-autocomplete` controls and then look them up based on some kind of index related to the array of `ITicketSummary`.   My second thought was "ugh, that's kind of gross", so I posed the question to a `Aurelia` discussion channel and was quickly answered with a *much* more elegant solution.

> you can bind view-model.ref="" to your list of items. and just bind the view model to a property on your items

To which my immediate reaction was 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/UWotM8.jpg" alt="U WOT M8?">

In doing this, Aurelia's binding engine adds a new field to the `ITicketSummary` called `autoComplete`

{% highlight html %}
<div repeat.for="t of tickets">
  <user-autocomplete 
    view-model.ref="t.autoComplete"
    context.bind="{priority: priority}" 
    on-user-selected.call="userSelected(user, context)">
  </user-autocomplete>
</div>
{% endhighlight %}

{% highlight typescript %}
export class MyController {
  public summaryItems: ITicketSummary[]

  public disableUserSelectForTickt(ticket) {
    //TADA!
    ticket.autoComplete.disable();
  }
}
{% endhighlight %}

I did have to cheat a little by removing the `ticket` type from the method signature, but I could have just as easily cast to any or added a property to my interface.  Lesson reinforced:  If you want your javascript objects to have a method or property at runtime that they do not have at design time just add it.