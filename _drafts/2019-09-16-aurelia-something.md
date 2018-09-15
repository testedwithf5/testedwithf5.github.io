---
layout: single
title:  "Logging Gulp Errors"
date:   2018-09-15 19:18:00 -0500
categories: javascript
comments: true
tags: javascript, aurelia
---

if I have a custom element in a repeat.for is it possible to somehow reference this in the parent page? So for example, I have an items list that is a non-deterministic length

           <div repeat.for="i of items">
              <user-autocomplete context.bind="{priority: priority}" on-user-selected.call="userSelected(user, context)">
              </user-autocomplete>
            </div>
How would I go about accessing userAutoComplete(someIndex) ?


Tim Fish @timfish 11:59
@joelowrance you can bind view-model.ref="" to your list of items.
and just bind the view model to a property on your items
So you'd have <user-autocomplete view-model.ref="i.vm" ...
