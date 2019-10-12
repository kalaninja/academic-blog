+++
tags = ["c#", "mongodb"]
categories = ["DB"]
title = "Reducing the size of MongoDB document in c#"
date = 2017-04-15T14:17:00+03:00
draft = false
+++

Storing data in MongoDB with the official C# driver sooner or later you might run into the following exception:
```
System.IO.FileFormatException: Size 22327168 is larger than MaxDocumentSize 16777216.
```
This is happening because of the limitation of a single document size that exists in MongoDB by design.

> The maximum BSON document size is 16 megabytes. The maximum document size helps ensure that a single document cannot use excessive amount of RAM or, during transmission, excessive amount of bandwidth.

Though there is a JIRA ticket called [Increase max document size to at least 64mb](https://jira.mongodb.org/browse/SERVER-5923), it doesn’t seem likely to be done in the near future. Geert Bosch, senior software engineer at MongoDB, wrote the following comment:

> In order to support much larger documents, such as the 64Mb documents suggested, assumptions such as that we can easily allocate copies of documents for modification will no longer hold. Transactions could grow extremely large (just think of a 64Mb array of elements to index). We would need to significantly throttle the number of concurrent operations, or raise the memory requirements for mongod.

But, if you find yourself stuck in such a situation, don’t get frustrated – there is a solution.
<!--more-->

## Explore the document

MongoDB uses JSON documents in order to store records, but stores them in a binary-encoded format called BSON. Take a look at your document. It might look somewhat similar to this:
```js
{
  "Property1": true,
  "Property2": 123,
  "Property3": "Hello World",
  "Property4": null,
  "Property5": null,
  "Property6": [
    1,
    2,
    3
  ],
  "Property7": []
}
```
With an object-oriented representation like .net objects usually store their name and the `null` literal for each null property. Also you may notice empty arrays. Another thing that may be tuned is the property name mapping. By default, the names of the fields in the generated JSON document are equal to those of the corresponding properties. So, the solution is to get rid of everything that is possible and shorten the names.

## The solution

### Ignoring null values

To ignore the null values you have at least 3 options:

*   decorating properties with BsonIgnoreIfNull attribute
*   using fluent api
*   register a global convention

The first option lets you mark properties individually:
```cs
public class Class1
{
    public bool Property1 { get; set; }
    public int Property2 { get; set; }
	[BsonIgnoreIfNull]
    public string Property3 { get; set; }
	[BsonIgnoreIfNull]
    public string Property4 { get; set; }
	[BsonIgnoreIfNull]
    public string Property5 { get; set; }
	[BsonIgnoreIfNull]
    public List<int> Property6 { get; set; }
	[BsonIgnoreIfNull]
    public List<int> Property7 { get; set; }
}
```
But it is better to keep your business entities clean, so consider the second option.
```cs
BsonClassMap.RegisterClassMap<Class1>(x =>
{
     x.AutoMap();
     x.GetMemberMap(m => m.Property3).SetIgnoreIfNull(true);
     x.GetMemberMap(m => m.Property4).SetIgnoreIfNull(true);
     x.GetMemberMap(m => m.Property5).SetIgnoreIfNull(true);
     x.GetMemberMap(m => m.Property6).SetIgnoreIfNull(true);
     x.GetMemberMap(m => m.Property7).SetIgnoreIfNull(true);
});
```
But still it can add a lot of code, so you might want to introduce a global policy.
```cs
ConventionRegistry.Register(
	"Ignore null values",
	new ConventionPack
	{
		new IgnoreIfNullConvention(true)
	},
	t => t == typeof(Class1));
```
The last predicate ensures the policy only applies to `Class1` class. Also make sure you are registering the new convention pack early enough. If the class has already been mapped it won’t be updated.

### Ignoring empty collections

It is not complicated to do with the `SetShouldSerializeMethod`. The specified predicate is triggered to determine whether the member should be serialized. Simply check the number of items inside the collection before saving it. Here is the code
```cs
BsonClassMap.RegisterClassMap<Class1>(x =>
{
     x.AutoMap();
     x.GetMemberMap(m => m.Property7).SetShouldSerializeMethod(x => ((Class1)x).Property7.Count > 0);
});
```

### Shortening names

The last and probably the most efficient way is to make each class property name shorter. Of course it affects your data readability, that is why it is not the first offered solution, but in my case it reduced the document size by two times. As usual you can use an attribute or a fluent approach. The attribute is called `BsonElement.`
```cs
public class Class1
{
	[BsonElement("p1")]
    public bool Property1 { get; set; }
	[BsonElement("p2")]
    public int Property2 { get; set; }
	[BsonElement("p3")]
    public string Property3 { get; set; }	
}
```
The following code can be used with the fluent api
```cs
BsonClassMap.RegisterClassMap<Class1>(x =>
{
     x.AutoMap();
	 x.GetMemberMap(m => m.Property4).SetElementName("p4");
	 x.GetMemberMap(m => m.Property5).SetElementName("p5");
	 x.GetMemberMap(m => m.Property6).SetElementName("p6");
     x.GetMemberMap(m => m.Property7).SetElementName("p7");
});
```

## Result
After applying this solution you might be able to get a document that fits into the MongoDB database. Compared to the original document the result seems to save a lot of space. Consider the resulting JSON
```js
{
  "p1": true,
  "p2": 123,
  "p3": "Hello World",
  "p6": [
    1,
    2,
    3
  ]
}
```
I think that the difference in size is obvious. Using all three techniques allowed me to store a document that seemed too large in the beginning.