---
layout: post
title:  "Entitity Framework: How to change the connection string of a 'database first' model at runtime?"
date:   2017-03-11 12:09:06 +0100
comments: true
tags: c# 
---
When defining a database first model with Entity Framework in ASP.net, you are forced
to define the database connection string in `Web.config` like this:

```xml
  <connectionStrings>
    <add name="MyConnection" connectionString="Data Source=(local);Database=EfExample;Integrated Security=True" providerName="System.Data.SqlClient" />
  </connectionStrings>
```

The Entity Framework will delegate all database calls to the model via this connection, as the generated context
class gets instantiated with this connection string name:

```csharp
public partial class MyEntitites
{
  public MyEntities(): base("name=MyConnection")
  {

  }
}
```

To modify this behavior, subclassing and overriding the constructor is not feasable, because the parent class 
is not providing a constructor which allows to pass a connection string name.
As mentioned, the above got generated. Changes to the class would be overwritten on new updates to the datamodel.
Therefore it is not recommended to modify generated content, as this would result in an unmaintainable nightmare.
Fortunately the generated class is marked with the `partial` modifier. Just add another 'part' to this class in
a separate file which provides the desperately needed constructor.

The final result would look like this:
```csharp
//In a seperate file, e.g. ConfigurableEntities.cs
public partial class MyEntitites
{
  public MyEntities(string dbName): base($"name={dbName}")
  {

  }
}
```

Add the additional connectionString to `Web.config`.
```xml
  <connectionStrings>
    <add name="MyConnection" connectionString="Data Source=(local);Database=ExampleDB;Integrated Security=True" providerName="System.Data.SqlClient" />
    <add name="MyAlternativeConnection" connectionString="Data Source=(local);Database=AlternativeExampleDB;Integrated Security=True" providerName="System.Data.SqlClient" />
  </connectionStrings>
```

Use the new constructor in your code:
```csharp
public MyClass {
  private MyEntitites db = new MyEntities("MyAlternativeConnection");
}
```

Calls to this context instance will lead to your new connection string.