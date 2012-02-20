Yet another persistence library for Android. This library works as a SQLite wrapper and allows you to easily create,
query and work with schemas based on objects.

###Maven integration

In order to use this library from you Android project using maven your pom should look like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project ...>
    ...

    <dependencies>
        <dependency>
            <groupId>com.codeslap</groupId>
            <artifactId>persistence</artifactId>
            <version>[0.9.)</version>
            <scope>compile</scope>
        </dependency>
        ...
    </dependencies>

    <repositories>
        <repository>
            <id>codeslap</id>
            <url>http://casidiablo.github.com/codeslap-maven/repository/</url>
        </repository>
    </repositories>
</project>
```

###Normal integration

Refer to the downloads section to get a JAR to import to your project.

###How it works?

Create a class that extends `android.app.Application` like this:

```java
public class App extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        SqlPersistence database = PersistenceConfig.getDatabase("database_name.db", 1);
        database.match(Foo.class, Bar.class);
    }
}
```

And add this to your manifest:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="your.package.name"
          ...>
    <application ...
                 android:name="your.package.name.App">
     ...
```

Here `Foo` and `Bar` are [POJO][1]s that you will use through your app. Persistence library will automatically create
sqlite tables for those classes, which will allow you to insert, query, update and delete data in a easy way:

In order to interact with the database, you must use an implementation of the [SqlAdapter][2] interface. There are two
implementations; you can get an instance of those implemantion this way:

```java
// 1. Standard implementation...
SqlAdapter adapter = Persistence.getSqliteAdapter(context);
// use the adapter and then...
adapter.close();

// 2. Quick implementation
Persistence.quick(context).someMethod();
```

Difference between standard and quick implementation is that quick adapter can be used only once and it will run
clean-up tasks after it is executed. Use quick implementation when you just want to do a simple thing (e.g. inserting
bean); use standard implementation if you want to do more things (e.g. inserting an object, then updating another and
finally query some other data).

### Inserting/updating data

To insert a simple object to the database use the `store` method:

```java
// single insertion
Foo foo = new Foo();
// add data to your object foo.setExample(...);
Persistence.quick(context).store(foo);
```

 **Notice:** if you are inserting an object of type `Foo`, you must have already registered that class in the
 *`Application` class*.

If you want to store a collection of beans use the `storeCollection(list, listener)` method:

```java
List<Foo> foos = new ArrayList();
// foos.add(foo);
Persistence.quick(context).storeCollection(null, new ProgressListener() {
    @Override
    public void onProgressChange(int percentage) {
    }
});
```

This is much more efficient than implementing a loop manually since this will not insert items one-by-one but instead
will create a bulk insert statement. There is another version of this method called `storeUniqueCollection` which
basically inserts and updates objects that you pass in the list, and delete from the database those items that are not
included in the list.

When you insert an object whose primary key is not auto-increment, it will try to update it instead of inserting a new
one. In other cases use the `update` method:

```java
City sample = new City();
sample.setName("vogota");

City newCity = new City();
newCity.setName("Bogotá");

Persistence.quick(context).update(newCity, sample);
```

Notice that `update` method can also be used with raw SQL statements and Android wildcards.

### Querying data

You can query single objects or a collection of objects:

```java
// query a single item by example
City city = new City();
city.setName("Bogotá");
City bogota = Persistence.quick(context).findFirst(city);
```

You can also use raw SQL queries:

```java
City bogota = Persistence.quick(context).findFirst(City.class, "name LIKE 'Bogotá'", null);
// although it is recommended to use Android's wildcards:
City bogota = Persistence.quick(context).findFirst(City.class, "name LIKE ?", new String[]{"Bogotá"});
```

Use `findAll` to get a list of objects that matches some conditions:

```java
// Query all cities
List<City> cities = Persistence.quick(context).findAll(City.class);

// Query cities that match a sample
City sample = new City();
sample.setCountryCode("CO");
List<City> colombianCities = Persistence.quick(context).findAll(sample);

// You can set some constraints
Constraint constraint = new Constraint().limit(3).groupBy("column").orderBy("name");
List<City> someCities = Persistence.quick(context).findAll(sample, constraint);
```

### Deleting data

Just use the `delete` method:

```java
// this will truncate the table...
Persistence.quick(context).delete(City.class, null, null);

// this will delete the items that match the sample
City sample = new City();
sample.setCountryCode("CO");
Persistence.quick(context).delete(sample);
```

### Feedback

If you have any questions or suggestions do not hesitate to sending me an email about it (cristian@elhacker.net).
Keep in mind that this is project is in beta phase and I do not warranty it will work as expected.

  [1]: http://en.wikipedia.org/wiki/Plain_Old_Java_Object
  [2]: https://github.com/casidiablo/persistence/blob/master/src/main/java/com/codeslap/persistence/SqlAdapter.java