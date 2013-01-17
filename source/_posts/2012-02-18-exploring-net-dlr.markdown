---
layout: post
title: "The DLR Isn't Just For Dynamic Languages"
date: 2012-02-18 21:20
comments: true
categories:
- dynamic
- .NET
- C#
- DLR
---

### Introduction

After attending CodeMash this past January, I became very interested in the .NET DLR and what it has to offer with respect to making coding activities more natural and hassle-free. For those of you that are not familiar with the .NET DLR (dynamic language runtime), I highly encourage you to take a look at it. Essentially, the DLR brings dynamic language support to the .NET programming environment.  This has allowed for dynamic languages, such as Python or Ruby, to execute on the .NET runtime. Running Python and Ruby on .NET is pretty awesome. However, this might not be useful for a lot .NET developers.

It turns out that the DLR isn't just for dynamic languages. The DLR has some very powerful features that the everyday programmer can take advantage of. Let's imagine for a moment that you have a need for a dynamic object (and don't worry about the details of what a dynamic object is, or why you need one...hopefully this will become more clear
by the end of this discussion). Luckily for us, Microsoft has provided some tools to make dynamic object creation pretty easy. Here are a few examples of these tools:

*  ExpandoObject - a dynamic object that allows members to be added and removed at runtime
*  DynamicObject - a base class you can inherit from in order to implement your own dynamic object
*  IDynamicMetaObjectProvider - an interface that represents a dynamic object that has its operations bound at runtime

#### Exploring the DLR with the ExpandoObject

The ExpandoObject is dynamic object that Microsoft has implemented for
us, that we can start using immediately. Here's a quick example of the
ExpandoObject in action:

{% codeblock lang:csharp %}

// create a new dynamic "person" object
dynamic person = new ExpandoObject();

// creates a new property "FirstName" and sets it's value to "Bill"
person.FirstName = "Bill";

// prints "Bill" to the console
Console.WriteLine(person.FirstName);

// throws an exception because the "LastName" property does not yet exist
Console.WriteLine(person.LastName);

{% endcodeblock %}

As you can see from the above code sample, using the ExpandoObject was
pretty easy. Let's talk briefly about how this works. The first thing to
take note of is the "dynamic" keyword. "dynamic" is a static type, that
bypasses the static type checking performed by the compiler.  

How the "dynamic" keyword works:

* The "dynamic" keyword tells the C\# compiler to bypass the static checking that it normally performs
* This means that when you compile your code, it will not check whether a method or property exists on the object
* You are basically telling the compiler "trust me, this property or method exists"
* Because the compiler is not performing static checking, it will not be able to determine whether a method or property exists until runtime (this is referred to as late-binding and is accomplished through dynamic dispatch)
* Because the compiler is not performing static checking, you can call a method or property that does not exist, and you will not see an error until this method or property is invoked at runtime
* There is a performance penalty because the compiler is not able perform any optimizations, and has to use dynamic dispatch (this is not the full story however, the compiler also emits call site caching which is beyond the scope of this article)

As you can see, the story of using the "dynamic" keyword is not simple.
A lot of things are taking place behind the scenes that you might not
have been aware of.

Now that we've covered how the "dynamic" keyword works, we can explore the ExpandoObject in more detail. In the code example, we instantiate a new ExpandoObject as the type "dynamic". This allows us to call any
method or property on the object without the compiler generating errors. You may be asking, how does the ExpandoObject get or set a member when no implementation has been defined? The ExpandoObject is essentially a
dictionary, where the keys are member names (ex. "FirstName"), and the value of the keys are set to the members' values (ex. "Bill"). For example, consider this code:

{% codeblock lang:csharp %}

// create a new dynamic "person" object
dynamic person = new ExpandoObject();

// creates a new property "FirstName" and sets it's value to "Bill"
person.FirstName = "Bill";

// prints "Bill" to the console
Console.WriteLine(person.FirstName);

{% endcodeblock %}

When the member "FirstName" is set to the string "Bill" on the person
object, the following takes place:

* The DLR calls a method on the ExpandoObject, passing in the name of the property as the string "FirstName"
* It also passes the value of the property, the string "Bill"
* Because the ExpandoObject is an IDictionary\<string, object\>, it will use the property name "FirstName" as the key to save the value "Bill"

#### Exploring the DLR with DynamicObject

The ExpandoObject is a pretty powerful right out of the box. Still, there may be situations where the ExpandoObject won't provide the features that you need. At this point, you'll have to write your own dynamic object implementation. A possible option would be to inherit from the DynamicObject class. This is a little more involved than using an ExpandoObject, but it's still pretty straightforward. The DynamicObject class still takes care a lot of the low-level DLR stuff so that you can easily implement your own dynamic object. Let's walk through the details of implementing our own dynamic object by creating our own implementation of an ExpandoObject.

{% codeblock lang:csharp %}

public class AnotherExpandoObject : DynamicObject
{
  private static readonly IDictionary<string, object>Dictionary =
    new Dictionary<string, object>();

  public override bool TryGetMember(GetMemberBinder binder, out object result)
  {
    return Dictionary.TryGetValue(binder.Name, out result);
  }

  public override bool TrySetMember(SetMemberBinder binder, object value)
  {
    if (Dictionary.ContainsKey(binder.Name))
    {
      Dictionary[binder.Name] = value;
    }
    else
    {
      Dictionary.Add(binder.Name, value);
    }
    return true;
  }
}

{% endcodeblock %}

This is a pretty basic implementation of an dynamic object that has some of the same behavior as the built-in ExpandoObject class. Let's change the code from the first code sample, to the following code:

{% codeblock lang:csharp %}

// create a new dynamic "person" object
dynamic person = new AnotherExpandoObject();

// creates a new property "FirstName" and sets it's value to "Bill"
person.FirstName = "Bill";

// prints "Bill" to the console
Console.WriteLine(person.FirstName);

{% endcodeblock %}

Notice that the only change was that we are now using AnotherExpandoObject instead of ExpandoObject. This particular code sample will run and behave in the same way as it did with the ExpandoObject.

#### Exploring the DLR with IDynamicMetaObjectProvider

A third option for creating your own dynamic object is to use the IDynamicMetaObjectProvider interface. The details of using this method, are beyond the scope of this article. I am hoping to explore this more myself, and to talk about it in a future post.

#### Where to go next?

So, why would you want to use the DLR? One particular scenario I've been exploring is with consuming RESTful services that utilize JSON and XML. I believe that the DLR is a really compelling choice for creating libraries and frameworks that help discover and consume RESTful web services. Imagine there was a library that allowed you to write the following type of code (against any RESTful API):

{% codeblock lang:csharp %}

// create a new rest client to the specified base url
dynamic restClient = new RestClient("http://url.to.some.restful.api.com");

// build up the resource url dynamically, and execute an http get
// against the url http://url.to.some.restful.api.com/albums/photos/
var photos = restClient.albums.photos(Method.Get);

// read the returned json collection in a very natural way
foreach (var photo in photos)
{
  Console.WriteLine(photo.name);
  Console.WriteLine(photo.date);
  Console.WriteLine(photo.albumName);
}

{% endcodeblock %}

There are a lot of concepts in the above example (the .NET DLR, convention over configuration, fluent interfaces, etc.) Some work has already been with respect to a JSON implementation. Check out the following code on [github](https://github.com/bahrens/Dyno.Net) to get a preview. I'm hoping to cover this in more detail in my next post.

Comments
--------

Ben Ahrens

I was working with ASP.NET MVC 4 the other day and stumbled across the following class....System.Json.JsonObject. So, as it turns out Microsoft has already created the same class I was suggesting in my post. Most likely I will not continue creating the JsonObject class (aka Dyno.Net nuget package), unless it's purely for educational purposes. I would hate to reinvent the wheel. However, it still may be worthwhile to create a dynamic rest client.
