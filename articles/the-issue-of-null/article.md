# Nullability: What causes, how appeard and what to do with it
In 2009 [Tony Hoare](https://en.wikipedia.org/wiki/Tony_Hoare) has declared that invention of nullability was his
["Billion dollar mistake"](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/).
"This has led to innumerable errors, vulnerabilities, and system crashes, which have probably caused a billion dollars
of pain and damage in the last forty years" he said. It's hard to find an opinion which could disagree with that, 
especially if their primary language is high-level (Java/C#/etc...). At the moment of 2023 it's more like a good
practice or even convention to avoid ``null``s in your code. Why? Let's find it out.

**Disclaimer:**
I will use Java as an example language, but most of the text and code are aplliable to any other language that have 
a ``null`` keyword. If I mention some class or library/framework, you may google "${ClassLibName} C# alternative" and I
promise you find something similiar.

## Issues of ``null``
The ``null`` by itself is a unique value for any type (exluding primitives) and could be assigned at any time. 
Techinacally, that is manifistation of the 
["dynamic typing system"](https://en.wikipedia.org/wiki/Type_system#Dynamic_type_checking_and_runtime_type_information)
but for most languages it's more like an only dynamic exception of the static typing system. Although, it still
bringing all the dynamic typing downsides to the language and here they are.

### You may get not what you expect
The staticly typed code usually rely on expectations. When all the methods and functions explicitly defines a type of
what they returns, and you know that compiler will not let them return anything else, you also sure about it and you
writing your code by expecting it. When there's a method ``User getCurrentUser()`` it makes you belive that you will
get a ``User``, not a ``Car``, ``Kittie``, or anything else. Only ``User``. Or ``null``... And if you say that it's an
obvious thing to you, then you're a huge lier and i'll try to prove it. Look at the following code:
```Java
public class UserUtils {

    public static String getFullUserName(User user) {
        return user.name() + " " + user.surname() + " " + user.lastname();
    }
}
```

Looks pretty simple right? But what if ``user`` is ``null``? Or what if ``user.surname()`` will return ``null``? You
never know the origin of the ``User`` object you receive, and how it was built, which its fields were assigned and
which not. You also couldn't be sure how other programmer will use your method and will he know that it might produce
an NPE? Well, probably that was just a bad code, so we have to rewrite it to something like:
```Java
if (user != null) {
    final var builder = new StringBuilder();
    if (user.name() != null) {
        builder.append(user.name() + " ")
    }
    if (user.surname() != null) {
        builder.append(user.surname() + " ")
    }
    if (user.lastname() != null) {
        builder.append(user.lastname())
    }

    return builder.toString();
}

return ""; // Or what?????
```

Now it safe? Yeah, but we got 15 lines instead of one line of simple and direct logic. Also the code below will produce
unexpected results like just two spaces and that's it.

You see the comment on the last line? I left it because I don't really understand what should I do if I got ``null``
insteaed of a ``User`` instance. **And it's not my fault**, because I writing my method for a ``User``, not for a
``Car``, ``Kittie`` or ``null``. That's the point - you define what you expect and work with it.

### Theoretical corectness
Object-oriented programming idea is based on our material world. We look in the window, we see cars riding streets.
Multiple cars has multiple weels. Here an idea - class Wheel and four instances of that in the class Car. Straight and
explicit. We don't need to know what that Wheels made of (usually), since it's a responsibility of a wheel factory and
government standarts. We don't need to know how long it takes to create a wheel, the wheel factory might improve it if
it's possible. But if you really need some special wheels, you can open your own wheel factory and take that
reposnsibility on yourself. Anyway now I want to replace a broken wheel in my car so here's my object-oriented story
about that:

I going to shop, buying a whell, then going to my garage, removing a broken wheel, putting the new one I bought and
when I tighten the bolts on it - **BOOM** the universe collapsing and I don't even know why, because all that path
I maid, I didn't even knew that I carrying a ``null`` instead of a wheel, because a wheel shop is out of stock.

Imagine if a wheel shop just said ``WheelsOutOfStockException``. Even if the universe sacrifised we might had a sraight
reason of that, which we could handle in future.

### Practice - is the criterion of truth TODO
Ignoring this problem was already caused decent amount of bugs. These bugs are among of most annoying that occurs
unexpectedly, hard to reproduce and takes long to investigate them. Also an important thing about these bugs - if
programmer follows a couple of simple recomendations - these bugs will **never** appear. Keep it in mind, when you see
the stats.

Here's some data about how much ``Exception``s were ever occured and how much of them are ``NullPointerException``s.

**Stackoverflow** (10 July 2023)
| Search query                          | Amount  | Precent |
|:--------------------------------------|:--------|:--------|
| [java] Exception                      | 589,896 | 100.0%  |
| [java] NullPointerException           | 54,781  | 9.3%    |
| [java] IllegalStateException          | 10,887  | 1.8%    |
| [java] ClassCastException             | 10,158  | 1.7%    |
| [java] ArrayIndexOutOfBoundsException | 5,234   | 0.9%    |

**GitHub** (10 July 2023)
| Search query                                 | Amount | Precent |
|:---------------------------------------------|:-------|:--------|
| language:Java Exception                      | 902k   | 100.0%  |
| language:Java NullPointerException           | 207k   | 22.9%   |
| language:Java IllegalStateException          | 64k    | 7.1%    |
| language:Java ClassCastException             | 81.2k  | 9.0%    |
| language:Java ArrayIndexOutOfBoundsException | 43.4k  | 4.8%    |

Also, there's some data from exact products bugtrackers, collected at 10 July 2023

*All there references are at the end of an article.*

| The software     | All Exceptions | NPE's | NPE's precent |
|:-----------------|---------------:|------:|--------------:|
| IntelliJ IDEA    | 28,411         | 4,027 | 14.2%         |
| Minecraft        | 13,069         | 4,118 | 31.5%         |
| Eclipse Platform | 10,570         | 4,484 | 42.4%         |
| Jenkins          | 10,348         | 3,181 | 30.7%         |
| GraalVM          | 1,306          | 207   | 15.8%         |
| Unity3D (C#)     | 2,576          | 1,433 | 55.6%         |

## How and when to avoid ``null``s
First of all, Java (and other languages) is just designed with this mistake. Language desginers can't just remove it,
even if they would, atleast because of backward compatibility. Anythying, even programming languages becoming a legacy 
at some time, and it have to support all the code was written before.

But you can protect your new code, even if it use old libraries/frameworks. Here's few cases when you would use null,
and how to avoid it:

### Case: Can't provide it because of error
Sometimes you may try to get a thing, but you get an unexpceted exception. That thing might be a database connnection,
file on local disk, resource, static library, or any other **external** dependedncy that is out of your responsibility
scope. These cases are usual, and even if Java forces you to ``catch`` exceptions, it doesn't means you have do handle
them.

There are two possible solutions: forward exception or wrap it into runtime one. Choose of solution is up to you, the 
both ways are conventioned, but they're still different. Let's look on both
```Java
public class Application {
    private Database db = null;

    public Database initDbConnection() throws DbIntializationException, DbBadCredentialsException {
        this.db = new Database(); // Both exceptions are thrown from there
        return db;
    }

    // OR:

    public Database initDbConnection() {
        try {
            this.db = new Database();
        } catch (DbIntializationException || DbBadCredentialsException e) {
            throw new RuntimeException("Unable to init DB connection", e);
        }

        return db;
    }
}
```

The first way with forwarding is more appliable to critical parts of code, where you **guess that your method might
cause an error** and you want to notify another developer (or yourself) about it. The second way is more confident and
silent, you may use it when everything should be fine, but **maybe someone changed something** (external code, 
environment vars, configs, etc..) that broke my code, so it's not goes that well anymore. These cases are unsual and
breaks the whole thing, so there's no necceserrity to explicitly handle all of them.

Well, you could actually combine two these ways by using a javadoc:
```Java
/**
 * Initializes a database connection
 * @throws RuntimeException when connection couldn't be initialized or wrong credentials
 */
public Database initDbConnection() {
    try {
        this.db = new Database();
    } catch (DbIntializationException || DbBadCredentialsException e) {
        throw new RuntimeException("Unable to init DB connection", e);
    }

    return db;
}
```

With that way, you don't force people to write ugly try-catch blocks, but also point them what excatly were wrong if
they got an exception from your method. Description might be more detailed then mine, but it still partially removes an
implicity of the second method.

### Case: It's just not exists
But what if thing just not exists yet or anymore? What if you definetly sure that at some state of your object, it will
not have a field? It might be a ``record`` or a [POJO](https://en.wikipedia.org/wiki/Plain_old_Java_object) with
multiple nullable fields, so it's just inconvenient to wrap all the code with checks?

There's a way in default Java API to handle null values - ``Objects.requireNonNull()`` method.
```Java
import java.util.Objects;

public class Application {
    
    public Database getDatabase() {
        return Objects.requireNonNull(db); // Will throw an NPE here if db is null
    }
}
```

Usually this check goes on the "``getDatabase()``" invoker side, because it may confuse other developers who may face
that NPE this code throrws. Although, it's definately better then just ignore possible nullability, because that NPE is
thrown at the exact position of stack which needed for debugging (In example of weel shop - we throw an exception when
we buy it, before actual usage). Anyways there's one extra thing you should definatly have an attention on - the
**has-methods**.

Has-method is a method that returns ``boolean`` value indicating an existance of something.
```Java
public class Application {
    
    public Database getDatabase() {
        return Objects.requireNonNull(db);
    }

    public boolean hasDatabase() {
        return db != null;
    }
}
```

Here, an ``Application`` instance owner knows that you have some method ``boolean hasDatabase()``, which obviously
indicates about a "database" existance, so it might not exist. Also, an owner could write more readable code for
checking it, that directly improves a code quality:
```Java
public class ApplicationManager {
    private final Set<Application> apps;

    public void executeQueryForAll(Query query) {
        /*  This code isn't that clear:

        apps.stream().filter(app -> app.getDatabase() != null).forEach(
            app -> app.getDatabase().execute(query));

            This one is worse:

        apps.stream().forEach(app -> {
            if (app.getDatabase() != null) {
                app.getDatabase().execute(query);
            }
        });
        */
        apps.stream().filter(Application::hasDatabase).forEach(
            app -> app.getDatabase().execute(query));
    }
}
```

Also, there's an ``Optional`` class exists, but it will be the next one

### Case: It's not neccesarry to exists (Existance is optional)
Sometimes we using ``null`` when we couldn't return a thing, but it's not that neccesarry for execution to throw an 
exception and drop the whole stack of calls. In this case, the ``Optional`` class appears
```Java
import java.util.Optional;

public class Application {

    /**
     * @return User if it exists in the database
     * @see #registerUser();
     */
    public Optional<User> getCurrentUser() {
        // Selecting user from database with this application id
        final User user = db
                .select(User.class)
                .where(Criteria.eq(UserFields.appId(), getId()));

        return Optional.ofNullable(user);
    }
}
```

If you're not familiar with this one, it's a beautiful replacement for previous solution, but not everywhere. 
> The short description of what that ``Optional`` does (may skip if already know)
>> ``Optional`` object may hold value or nothing and you could easly check that. It also provides convinient function
>> interfaces like ``orElse(/*AnotherInstance*/)`` or ``ifPresent(i -> {/*action*/})``.



#### Comparing ``Optional`` and has-methods

### Case: Another method returns me a ``null`` and checking it is bulky and wacky
Before this one, the other cases I mentioned was only for the "root code" where we get an error. Sometimes we using 
libararies that already delivered with uncovered nullability and it's too bulky to wrap its entire interface to another
null-safe adapter. Well, the ``requireNonNull()`` method sounds like solution, because it's a shortest one, but it just
throws NPE every time, and the code starting looks like ton of assertions, that might be not neccessary though.

These cases are usual, and it's more recomended to use an ``Optional`` as functional interface. The biggest reason of
that - decent readability improvement. Let's see an example
```Java
import static java.util.Optional.ofNullable;

public class UserUtils {

    public static String getFullUserName(User user) {
        return ofNullable(user).map(u -> 
                ofNullable(u.name()    ).orElse("[No name]") + " " + 
                ofNullable(u.surname() ).orElse("[No surname]") + " " + 
                ofNullable(u.lastname()).orElse("[No lastname]"))
            .orElse("[Unknown user]");
    }
}
```

This implementation of ``getFullUserName()`` still feels a little overcomplex, but keep in mind that it's not only
performing a checks, it also provides default replacements for nullable fileds, so its functionality is extended. Also,
notice that ``Objects.requireNonNullElse()`` method might be better somtimes. But what if we want to just throw NPE's, 
the code might be a way more simple:
```Java
public static String getFullUserName(User user) {
    return  requireNonNull(user.name()) + " " + 
            requireNonNull(user.surname()) + " " + 
            requireNonNull(user.lastname());
}
```

Notice that all solutions above are appliable for any Java program since the 1.8 version. Obviously, not only default
SDK can provide you tools for nullability excluding. Most of the Java programmers use popular frameworks and libraries
which contains some extras for our problem.

### Another solutions
#### Contracts
In 2023, static code analysis is a common thing. It already integrated in popular IDE's which performs scanning at the
same time you writing the code. It may give you some useful tips that helps to avoid most usual problems. Modern static
anylisis tools also may work with contracts.

Contract - is an explicit declaration of part or an entire behaviour of some logical code unit, which might be a class,
method, field, etc.. . The contract by itself is described in javadoc or annotation of unit. The example of contract is 
``@NonNull`` or ``@Nullable`` annotations. If some method or propety is marked with that, then the static analyser will
leave a warning on its usage when it violates the contract. Also it might warn a programmer that perform operaions on 
``@Nullable`` objects, which helps to decently decrease chances of unexpected brekeages at runtime. Here's few examples
of how to use these annotations:
```Java
public class UserUtils {

    // Exlicitly saying that this method handles nullability
    public static String getFullUserNameWithDefaults(@Nullable User user) {
        return ofNullable(user).map(u -> 
                ofNullable(u.name()    ).orElse("[No name]") + " " + 
                ofNullable(u.surname() ).orElse("[No surname]") + " " + 
                ofNullable(u.lastname()).orElse("[No lastname]"))
            .orElse("[Unknown user]");
    }

    public static String getFullUserName(@NotNull User user) {
        return user.name() + " " + user.surname() + " " + user.lastname();
    }

    public static void aLittleTest() {
        final User user = null;
        final String fullName = getFullUserName(user); // IDE shows warning on this line even if getFullUserName() has
                                                       // no sources
    }
}
```

Both annotation mentioned are present in multiple frameworks and libs and all made for porupse described above. You may
find them in [Spring](https://www.baeldung.com/spring-null-safety-annotations), 
[Android SDK](https://developer.android.com/reference/androidx/annotation/NonNull), and 
[many other sources](https://checkerframework.org/manual/#nullness-related-work)

#### Lombok
The [Project Lombok](https://projectlombok.org) is a sort of extension for Java language, that brings a lot of QoL 
improvements. It deserves a separate mention since its [``@NonNull``](https://projectlombok.org/features/NonNull)
annotation is not only a contract, but also a decent preprocessor. It adds some highly configurable ``null`` checkings.
Basically it just removes bulkiness of
```Java
public static String getFullUserName(User user) {
    if (user == null) {
        throw new NullPointerException("User couldn't be null!")
    }

    // Blah-blah...
}
```

By just adding an annotation to a ``user`` parameter

```Java
public static String getFullUserName(@NonNull User user) {
    // Blah-blah...
}
```

## Why null still exists?
Unfortunetly, even if we realize that ``null`` existance causes such critical issues we can't just give it up. 
Nullablity - isn't just a feature or [pattern](https://en.wikipedia.org/wiki/Null_object_pattern), it's more like one
of the root concepts that carries entire language and paradigm design that was inventioned during the years. That's why
it so painful to exclude ``null`` from your code.

Contracts, optionality abstractions, force-checks - are workarounds that might be even worse solution sometimes. Even
the ``null`` by itself usually solve problems, not causing them. Then more we use it - then bigger the chance of new
issues

Nullability - isn't straight-forward thing, it's super relative and undefined. Nobody can exactly say what is ``null``
in high-level languages. It just have no non-abstract definition, beacuse every time it wanted to be explained, people
talk about references and memory, which moves the context under the hood. Even the languge by its own syntax hiding
these low-level things from us:
```Java
final Integer a = 1; // A reference to 1
final int     b = 1; // Not a reference, but 1
assert a != null // Yeah, it's a ref
assert 1 != null // Compiler error
```

### ``null`` in low-level languages: C
If you didn't know, here's fun fact that definetly surprise you if you never used low-level languages - the C 
definition of NULL is:
```C
#define NULL 0
```

Yes, ``NULL == 0``, it's an integer. An it's actually makes sense, because pointers (references) in C are actually
integers! See, the low-level languages allows you to directly manage mememory, so any pointer (in short words) is a RAM
address. And in C, you can have a pointer to primivitive, which you would nullify in future.
```C
/** Refering to example above on Java */
void foo() {
    int *a;
    int b = 1;
    a = &b;
    *a = 2;
    assert(b == 2); // true
}
```

So here we see an actual (semantically correct) reference to something, and we can access an actual data by this
reference. We can retrieve and put the data by that, and we use ``NULL`` or a ``0``, to say that there's no data and
you can't put anythyng here. The ``NULL`` is conventioned address that points to nothing, and you may use it to point
to nothing.

### How modern languages handle nullability
In the more modern languages than Java or C#, we may find some decent improvements when we work with nulls.

#### Kotlin
In [Kotlin](https://kotlinlang.org/), everything is not null by default. Any nullable variable is marked by specific
type suffix - ``?``. Also there's a quick check operator ``!!`` which is equal to ``Object.requireNonNull()`` in Java.
```Kotlin
var nullableVar: Int? = null
nullableVar = requestSomeValue()
println(nullableVar!!) // throws an NPE here
```

#### Golnag
The [Go](https://go.dev/) is a low-level (relatively) programming language, which was designed as improved verison of
the C language, but more modern. Common types, inculding structures, strings and arrays couldn't be ``nil``. This
keyword is made for pointers (like in C), interfaces, functions, slices, maps and channels. 

Also, Go has methods which are semantically functions, so they couldn't cause a panic when invoked
```Go
type User struct {
    Name string
    Surname string
    Lastname string
}

func (this User) GetFullName() string {
    if (this == nil) {
        return "[Unknown user]"
    }

    return fmt.Sprintf("%v %v %v", this.Name, this.Surname, this.Lastname)
}

func main() {
    var user User = nil
    fmt.Println(user.GetFullName()) // [Unknown user]
}
```

That's will be a pretty useful feature in object-oriented architecture designs, because it moves nil-handling 
responsibilities from object to the method of that, which could check is ``this`` is ``nil``. That makes null value
more clear, because with this way the null is actually typed, so when we say the User is null, this user still has a
full name. Comparing to Java, it's not guarantees that ``this`` is not null, but improves semantic corectness at some
point.

## Data sources and useful links
### Data about projects

youtrack.jetbrains.com/issues/idea:
Exception: 28,411
NullPointerException: 4,027

https://bugs.mojang.com:
summary ~ "*Exception*" OR description ~ "*Exception*": 13069
summary ~ "*NullPointerException*" OR description ~ "*NullPointerException*": 4118

https://bugs.eclipse.org:
Product: Platform Content: NullPointerException : 4484
Content: Exception Product: Platform : 10570

https://issues.jenkins.io:
text ~ "NullPointerException": 3181
text ~ "Exception": 10348

https://github.com/oracle/graal/issues?q=Exception 1306
https://github.com/oracle/graal/issues?q=NullPointerException 207

https://unity3d.com/search?refinement=issues&gq=exception 2,576
https://unity3d.com/search?refinement=issues&gq=nullreferenceexception 1,433

### Opinions of people
[Anders Hejlsberg](https://en.wikipedia.org/wiki/Anders_Hejlsberg) (Microsoft Lead Architect of the C# language):
[ref](https://www2.computerworld.com.au/article/261958/a-z_programming_languages_c_/?pp=3)
>50% of the bugs that people run into today, coding with C# in our platform, and the same is true of Java for that
>matter, are probably null reference exceptions. If we had had a stronger type system that would allow you to say that
>‘this parameter may never be null, and you compiler please check that at every call, by doing static analysis of the 
>code’. Then we could have stamped out classes of bugs.