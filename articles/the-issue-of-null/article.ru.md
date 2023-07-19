# Давайте поговорим о null

Каждая такая статья начинается с одной и той же цитаты вне зависимости от мнения по вопросу. Чтож, не буду нарушать традицию. В 2009 году, [Тони Хоар](https://en.wikipedia.org/wiki/Tony_Hoare), заявил что изобретение null-ссылок было его ["ошбикой на мллиард долларов"](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/). "*Это привело нас к нисчесляемым ошибкам, уязвимостям и крашам, которые за последние 40 лет наверняка стоили порядка миллиарда долларов*". И правда, сейчас тяжело найти программиста, который не сказал бы что null создает больше проблем чем вызывает, особенно если этот программист пишет на высокоуровневых языках (Java/C#/etc..). На момент 2023 года, избегать ``null`` в коде уже кажется "хорошей практикой" или одним из ключевых навыков разработчика. Почему? Давайте разбираться.

**Дисклеймер:** Я буду использовать Java для примера, однако большинство кода и проблем применимы к другим платформам и языкам, в которых есть ``null``/``nil``. Все упомянутые либы/фреймворки можно загуглить по приципу "#{ИмяЛибыИлиКласса} C# alternative", однако автор не уверен в их существовании.

**Дисклеймер #2:** Статья изначально писалась на английском языке в мой блог, поэтому все ссылки будут на англоязычные ресурсы.

## Исследуем проблемы

Сам по себе, ``null`` - уникальное значение для любого типа (кроме примитивных) и может быть установлен в любой момент рантайма. По сути, это проявление  [динамической типизации](https://en.wikipedia.org/wiki/Type_system#Dynamic_type_checking_and_runtime_type_information), но так вышло что это единственное "динамическое исключение" в статически типизированных языках. Однако даже малого исключения хватает чтобы привзнести в язык полный спектр вытекающих проблем.

### Вы можете получить не то что вы ожидаете

Статически типизированный код, обычно опирается на ожидания. Когда все методы/функции вынуждены явно определять тип данных, который они возвращают, тогда и вы, как следствие, уверены в том что получите, и пишите код пологаясь на это. Когда вы видите метод `User getCurrentUser()`, вы думаете что вы получите что-то с типом ``User``, а не с типом `Car`, `Kittie` или каким-либо еще. Только экземпляр `User`. Или `null`... И если сейчас вы скажите что это очевидно, то я скажу что вы врете и мне и себе, и сейчас я попробую это доказать. Давайте посмотрим на следующий код:

```java
public class UserUtils {

    public static String getFullUserName(User user) {
        return user.name() + " " + user.surname() + " " + user.lastname();
    }
}
```

Выглядит довольно просто, да? Но что если `user` будет `null`? Или что `user.surname()` вернет `null`? Вы никогда не знаете ничего о происхождении экземпляра `User`, который получаете, как и не знаете о том какие его поля ``null`` а какие нет. Вы также не знаете как другой разработчик когда-то использует ваш метод, опираясь на его название или javadoc и будет ли он знать что может полчить NPE? Ну, давайте предположим что это просто плохой код, и нам надо его переписать с учетом нашей проблемы:

```java
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

return ""; // Или что?????
```

Теперь безопасно? Наверное да, но мы проучили 15 непонятных строк вместо одной, в которой была простая и прямая логика. Кстати даже такой код может верунть неожиданный результат вроде строки из двух пробелов.

Видите коментарий га последней строке? Я написал его, потому что как разработчк метода я не совсем понимаю что от меня ожидают когда дают мне `null` вместо `User`. **И дело не во мне**, потому что мой метод предназначен для `User`, а не для `Car`, `Kittie` или `null`. В том то и дело - когда мы пишем статически типзировнный код, мы явно говорим что нам нужно и с чем мы будем работать.

### Немного философии

Идяя объектно-ориентированного програмиирования основана на нашем материальном мире. Мы смотрим в окно, мы видим машины, мы видим дороги по которым они едут. Несколько машин, у которых несколько колес. Вот и идея - класс Wheel и четыре его экземпляра в классе Car. Прямо и явно. Нам не нужно знать из чего сделаны колеса в нашей машине (обычно), потому что за это отвечают производители колес и различные инспекции со своими стандартами. Нам не нужно знать насколько долго делается колесо для нашей машины, потому что производитель сам это улучшит, если это возможно и нужно. И даже если вам нужны какие-то другие, особенный колеса, вы можете сами стать производителем, чтобы взять эту ответственность на себя. В любом случае, сейчас у меня сломалось одно из колес и я хочу его заменить, и вот моя объектно-ориентированная история:

Я иду в магазин, покупаю колесо, затем иду в гараж, снимаю сломанное колесо, устанавливаю новое, и вот когда я закручиваю на нем болты - **БУМ!** вселенная схлопнулась по неизвестным причинам, но если я начну разбираться, то узнаю что весь свой путь я проделал держа в руках ``null`` а не колесо, потому что в магазине колес закончились колеса.

Конечно, было бы здорово если бы магазин выкинул `WheelsOutOfStockException`. Даже учитывая то что мы пожертвовали целую вселенную, мы хотябы смогли бы ее по-быстрее отдебажить и перезапустить, а еще уведомить ее жителей хоть чем-то кроме бессмысленного NPE.

### Практика - критерий истины

Не убедил? Тогда давайте вспомним о том, сколько багов в вашей практике уже повылазило по причинам выше. Еще, кстати, давайте вспомним что эти баги из тех самых надоедливых, непонятных, трудно воспроизводимых и неожиданных. А еще давайте вспомним что все эти ошибки были допущены просто из-за невнимательности и их можно было бы **полностью** избежать, следуя довольно простым рекомендациям. И давайте не забывать об этом пока мы будем смотреть на статистику.

Вот немного данных о проценте `NullPointerException`ов среди обычных ``Exception``ов.

**Stackoverflow** (10 Июля 2023)

| Поисковой запрос                      | Число   | Процент |
|:------------------------------------- |:------- |:------- |
| [java] Exception                      | 589,896 | 100.0%  |
| [java] NullPointerException           | 54,781  | 9.3%    |
| [java] IllegalStateException          | 10,887  | 1.8%    |
| [java] ClassCastException             | 10,158  | 1.7%    |
| [java] ArrayIndexOutOfBoundsException | 5,234   | 0.9%    |

**GitHub** (10 Июля 2023)

| Поисковой запрос                             | Число | Процент |
|:-------------------------------------------- |:----- |:------- |
| language:Java Exception                      | 902k  | 100.0%  |
| language:Java NullPointerException           | 207k  | 22.9%   |
| language:Java IllegalStateException          | 64k   | 7.1%    |
| language:Java ClassCastException             | 81.2k | 9.0%    |
| language:Java ArrayIndexOutOfBoundsException | 43.4k | 4.8%    |

Также, вот немного данных о конкретных продуктах, взятых из их багтрекеров 10 июля 2023:

*Все ссылки на источники в конце статьи*

| Продукт          | Все эксепшены | NPE   | процент NPE |
|:---------------- | -------------:| -----:| -----------:|
| IntelliJ IDEA    | 28,411        | 4,027 | 14.2%       |
| Minecraft        | 13,069        | 4,118 | 31.5%       |
| Eclipse Platform | 10,570        | 4,484 | 42.4%       |
| Jenkins          | 10,348        | 3,181 | 30.7%       |
| GraalVM          | 1,306         | 207   | 15.8%       |
| Unity3D (C#)     | 2,576         | 1,433 | 55.6%       |

## Как и когда можно избежать null

First of all, Java (and other languages) is **just designed with this mistake**. Language designers can't just remove it, even if they would, at least because of backward compatibility. Anything, even programming languages becoming a legacy at some time, and it has to support all the code was written before.

But you can protect your new code, even if it uses old libraries/frameworks. Here are a few cases when you would use null, and how to avoid it:

### Кейс: Не могу что-то вернуть из-за ошибки

Sometimes you may try to get a thing, but you get an unexpected exception. That thing might be a database connection, a file on the local disk, resource, static library, or any other **external** dependency that is out of your responsibility scope. These cases are usual, and even if Java forces you to ``catch`` exceptions, it doesn't mean you have do handle them.

There are two possible solutions: forward exception or wrap it into runtime one. Choice of solution is up to you, the both ways are conventioned, but they're still different. Let's look at both:

```java
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

The first way with forwarding is more appliable to critical parts of code, where you **guess that your method might cause an error** and you want to notify another developer (or yourself) about it. The second way is more confident and silent, you may use it when everything should be fine, but **maybe someone changed something** (external code, environment vars, configs, etc..) that broke my code, so it's not goes that well anymore. These cases are unusual and break the whole thing, so there’s no necessity to handle all of them explicitly.

Well, you could actually combine two of these ways by using a javadoc:

```java
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

With that way, you don't force people to write ugly try-catch blocks, but also point to them what exactly was wrong if they got an exception from your method. Description might be more detailed than mine, but it still partially removes an implicity of the second method.

### Кейс: Оно просто не существует

But what if a thing just not exists yet or anymore? What if you are definitely sure that at some state of your object, it will not have a field? It might be a ``record`` or a [POJO](https://en.wikipedia.org/wiki/Plain_old_Java_object) with multiple nullable fields, so it's just inconvenient to wrap all the code with checks?

There's a way in default Java API to handle null values - ``Objects.requireNonNull()`` method.

```java
import java.util.Objects;

public class Application {

    public Database getDatabase() {
        return Objects.requireNonNull(db); // Will throw an NPE here if db is null
    }
}
```

Usually this check goes on the ``getDatabase()`` invoker side, because it may confuse other developers who may face an NPE this code throw. Although, it's definitely better than just ignore possible nullability, because that NPE is thrown at the exact position of stack which needed for debugging (In example of wheel shop - we throw an exception when we buy it, before actual usage). Anyway, there’s one extra thing you should definitely have an attention on - the **has-methods**.

Has-method is a method that returns ``boolean`` value, indicating an existence of something.

```java
public class Application {

    public Database getDatabase() {
        return Objects.requireNonNull(db);
    }

    public boolean hasDatabase() {
        return db != null;
    }
}
```

Here, an ``Application`` instance owner knows that you have some method ``boolean hasDatabase()``, which obviously indicates about a "database" existence, so it might not exist. Also, an owner could write a more readable code for checking it, which directly improves a code quality:

```java
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

### Кейс: Это может не существовать (наличие опционально)

Sometimes we using ``null`` when we couldn't return a thing, but it's not that necessary for execution to throw an exception and drop the entire stack of calls. In this case, the ``Optional`` class appears:

```java
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

If you're not familiar with this one, it might cause questions related to actual need of that, because, at first sight, it might look like [another](https://softwareengineering.stackexchange.com/a/202482) abstraction. First, we need to find out what impact an ``Optional`` does besides obvious ``isPresent()`` which could be replaced by has-method.

#### Сравниваем Optional и has-методы

Let's take a look on obvious and simple case of getting a user name:

```java
// Bad
final User user = app.getCurrentUser();
return user == null ? "Unknown" : user.getName();
// Better
return app.hasCurrentUser() ? 
        app.getCurrentUser().getName() : "Unknown"
// Same?...
return app.getCurrentUser().map(user -> user.getName())
        .orElse("Unknown");
```

In such common cases, we can find at least two advantages of an ``Optional``:

- If the return type is wrapped in ``Optional`` then it would have an explicit marker of possible inexistence (comparing to javadoc or has-method implicity)

- ``Optional`` provides inline interface for handling an inexistence, so you don't have to wrap everything in ternary operator.

Although there's a huge downside when you're so confident that user you get is not null, you have to write something like:

```java
final User userImSureExists = getCurrentUser().get(); // IDE Warning
```

And that is really stupid. Some people say "how you ever be sure if you retrieving an **OPTIONAL** which is semantically something that you couldn't be sure in", but sometimes it's really a case. An example is a big state-control logic that was semantically separated by multiple methods to just not make a single bulky one, and all of them operating with multiple optional values. Let's say the number of these is enough to make method ugly by listing them as parameters, so it's easier to perform ``getCurrentUser()`` twice (especially if it's a simple getter), instead of do it once, and keep the value to another method, which will be invoked next. So the thing is you already has retrieved an optional, and checked its existence, and when you retrieve it a second time in the same call stack, it's obviously the same, so you have to do extra ugly ``.get()`` and suffer an IDE warning.

That's definitely a downside, and it's not a single one. I don't want to list all of them, at least because they are trivial, but if you are interested, I recommend checking [this article](https://blogs.oracle.com/javamagazine/post/optional-class-null-pointer-drawbacks).

There's also an extra text at the end of an article about cases where Optional might be a great solution.

### Кейс: Другой метод возвращает мне null но проверка делает код уродливым и запутанным

Before this one, the other cases I mentioned were only for the "root code" where we get an error. Sometimes we using libraries that already delivered with uncovered nullability and it's too bulky to wrap its entire interface to another null-safe adapter. Well, the ``requireNonNull()`` method sounds like solution, because it's a shortest one, but it just throws NPE every time, and the code starting looks like ton of assertions, whose might be not necessary though.

These cases are usual, and it's more recommended to use an ``Optional`` as functional interface. The biggest reason for that - decent readability improvement. Let's see an example:

```java
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

This implementation of ``getFullUserName()`` still feels a little overcomplex, but keep in mind that it's not only performing checks, it also provides default replacements for nullable fields, so its functionality is extended. Also, notice that ``Objects.requireNonNullElse()`` method might be better sometimes. But what if we want to just throw NPE's, the code might be a way more simple:

```java
public static String getFullUserName(User user) {
    return  requireNonNull(user.name()) + " " + 
            requireNonNull(user.surname()) + " " + 
            requireNonNull(user.lastname());
}
```

Notice that all solutions above are appliable for any Java program since the 1.8 version. Obviously, not only default SDK can provide you with tools for nullability excluding. Most of the Java programmers use popular frameworks and libraries which contain some extras for our problem.

### Другие решения

#### Контракты

In 2023, static code analysis is a common thing. It is already integrated in most popular IDEs whose perform scanning at the same time as you writing the code. It may give you some useful tips that help to avoid most usual problems. Modern static analysis tools also may work with contracts.

Contract - is an explicit declaration of part or an entire behaviour of some logical code unit, which might be a class, method, field, etc... The contract by itself is described in javadoc or annotation of unit. The example of contract is ``@NonNull`` or ``@Nullable`` annotations. If some method or property is marked with that, then the static analyser will leave a warning on its usage when it violates the contract. Also, it might warn a programmer that performs operations on ``@Nullable`` objects, which helps to decently decrease chances of unexpected brakeages at runtime. Here are a few examples of how to use these annotations:

```java
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

Both annotations mentioned are present in multiple frameworks and libs and all of them made for the purpose described above. You may find them in [Spring](https://www.baeldung.com/spring-null-safety-annotations), [Android SDK](https://developer.android.com/reference/androidx/annotation/NonNull), and [many other sources](https://checkerframework.org/manual/#nullness-related-work).

#### Lombok

The [Project Lombok](https://projectlombok.org) is a sort of extension for Java language, that brings a lot of QoL improvements. It deserves a separate mention since its [``@NonNull``](https://projectlombok.org/features/NonNull) annotation is not only a contract but also a decent preprocessor. It adds some highly configurable ``null`` checking you would write manually. Basically, it just removes bulkiness of:

```java
public static String getFullUserName(User user) {
    if (user == null) {
        throw new NullPointerException("User couldn't be null!")
    }

    // Blah-blah...
}
```

By just adding an annotation to a ``user`` parameter

```java
public static String getFullUserName(@NonNull User user) {
    // Blah-blah...
}
```

## Почему null вообще существует?

Unfortunately, even if we realize that nullability causes such critical issues, we can't just give it up. Nullablity - isn't just a feature or [pattern](https://en.wikipedia.org/wiki/Null_object_pattern), it's more like one of the root concepts that carries an entire language and paradigm design that was invented during the years. That's why it is so painful to exclude ``null`` from your code.

Contracts, abstractions, force-checks - are workarounds that might be even worse solution sometimes. Even the ``null`` keyword by itself usually not causing problems, but solving them, because it's obviously was mad for a good purpose. Sad, but over the time we just figured that the more we use it - than bigger the chance of new bugs appear.

Nullability - isn't a straight-forward thing, it's really relative and undefined. Nobody can exactly say what is ``null`` in high-level languages. It just has no clear definition, because every time it wanted to be explained, people talk about references and memory, which moves the context under the hood. Even the language by its own syntax hiding these low-level things from us:

```java
final Integer a = 1; // A reference to 1
final int     b = 1; // Not a reference, but 1
final var     c = 1; // Not an Integer, but int

assert a != null; // Yeah, it's a ref
assert 1 != null; // Compiler error
```

*Expression `Integer a = 1` not means `new Integer(1)`, but more similar to `Integer.valueOf(1)`, because it wanted to not initiate new reference*

It's better if we don't have ``null``, but the ``null`` is a brick in the language wall. If you remove it - sooner or later, the whole thing falls. You may hide it by putting some concrete or wallpaper over, but remember that it still be behind your workarounds, new languages with default non-null-immutable values, designs and patterns, it's will be there, and one day you will have to deal with it.

It's better if we don't have `null`, but since it's here, **we have to use it.**

### Null в низкоуровневых языках: C

Here's a fun fact that definitely surprises you if you never used low-level languages - the C definition of NULL is:

```c
#define NULL 0
```

Yes, ``NULL`` is ``0``. It's an integer. And it's actually makes sense, because pointers (references) in C are actually integers. See, the low-level languages allow you to directly manage memory, so any pointer, in short words, is an address of a RAM cell. And in C, you can have a pointer to the primitive, which you would nullify in the future.

```c
/** Refering to example above on Java */
void foo() {
    int *a;
    int b = 1;
    a = &b;
    *a = 2;
    assert(b == 2); // true
}
```

So here we see an actual (semantically correct) reference to something, and we can access an actual data by this reference. We can retrieve and put the data by that, and we use ``NULL`` or a ``0``, to say that there's no data and you can't put anything there. The ``NULL`` is a conventioned address that points to nothing, and you may use it to point to nothing.

In addition to that, C doesn't have a boolean type, but true and false are defined as following the constants:

```c
#define TRUE  1
#define FALSE 0
```

Zero is also a ``FALSE`` in the C world which adds some simplicity to null-checks

```c
char* get_user_name_by_id(int id) {
    user_iterable_t *user = get_all_users();
    if (!user) return NULL;

    while (user->next) {
        if (user->id == id) goto user_found;
        user = user->next;
    }
    return NULL;

    user_found:
    void *p = strcpy(user->name)
    user_iterable_free(user);
    return p;
}
```

See, the ``getUserNameById()`` function is still bulky, but pretty readable for such low-level language that is so close to assembler and machine instructions in meanings.

Speaking of any low-level language, the necessity of nullability is more dependent on performance and optimization questions. Having an ``Optional``, or immutability, or any other "NPE avoiding mechanism" will slightly increase consumption of memory and processing resources which are semantically redurant in the context of machine execution. If you have a GC in your environment or even an entire VM, these optimizations are worthless.

### Как совеременные языки помогают решить проблему

In the more modern languages than Java, we may find some decent improvements when we work with nulls. These improvements might be convenient ways to avoid nullability or cases when we get NPEs.

#### Kotlin

In [Kotlin](https://kotlinlang.org/), everything is not null by default. Any nullable variable is marked by a specific type suffix - ``?``. Also there's a quick check operator ``!!`` which is equal to ``Object.requireNonNull()`` in Java.

```kotlin
var nullableVar: Int? = null
nullableVar = requestSomeValue()
println(nullableVar) // prints "null"
println(nullableVar!!) // throws an NPE here

var nonNullVar: Int = null // Compiler error
```

Also, there's a safe invocation operator ``?.`` which is equivalent to ``.`` but will not perform a call if the object is null;

```kotlin
class Printer {
    fun print() = println("Im printing")
}

fun main() {
    var p: Printer? = Printer()
    p?.print() // prints "Im printing"
    p = null
       p?.print() // does nothing
}
```

Well, Kotlin is known as a "Sugar language" so [there are](https://kotlinlang.org/docs/null-safety.html) even more ways to avoid nullability and its issues.

#### Golnag

The [Go](https://go.dev/) is a low-level (relatively) programming language, which was designed as improvement and modern alternative of the C language. Common types, including structures, strings and arrays, couldn't be ``nil``. This keyword is made for pointers (like in C), interfaces, functions, slices, maps and channels (all of these are types). 

Also, Go has OOP methods, which are semantically functions (similar to Kotlin extensions), so they couldn't cause a panic when invoked on a ``nil``.

```go
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
    fmt.Println(user.GetFullName()) // "[Unknown user]", not a panic
}
```

That's should be a pretty useful feature in object-oriented architecture designs, because it moves nil-handling responsibilities from object to the method of that, which could check is ``this`` a ``nil`` or not. That makes null value more clear, because with this way the null is actually typed, so when we say the User is null, this user still has a full name. Comparing to Java, it's not guarantees that ``this`` is not null, but improves semantic correctness, which decreases breakage chances.

#### Rust

The [Rust](https://www.rust-lang.org/) goes kinda hardcore at this point - it just has no nulls. The solution is similar to java - an optional data structure, but it's a generified [enum](https://doc.rust-lang.org/std/option/enum.Option.html).

```rust
pub enum Option<T> {
    None,
    Some(T),
}
```

That enum has multiple methods similar to Java's ``Optional``.

In the most common cases, like [SQL result mappings](https://docs.rs/postgres/latest/postgres/) or [JSON mappings](https://github.com/serde-rs/json), it's common to use an optionality technique (use the ``Option`` enum or create a new with "null" value). But Rust also has an [unsafe part](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html) of it, which allowing to use nulls:

```rust
use std::ptr;

let p: *const i32 = ptr::null();
assert!(p.is_null());
```

As you can see, methods here are works just the same as in Golang, so it's definitely bringing some modernity to avoid null reference panics. Although, I should mention that Rust community are [hardly force](https://www.reddit.com/r/rust/comments/epzukc/actix_web_repository_cleared_by_author_who_says/) people to not use an ``unsafe`` so it's more correct to say that optionality is an only way in Rust.

#### Ruby

The ``nil`` in the Ruby language is not a value but an object of the ``NilClass`` type. It's a pretty good decision considering its dynamic typing system. The huge benefit of that is an ability to perceive it as actual value and convert to another type. Ruby classes have multiple methods named ``to_i()`` or ``to_s()``, for example, and these methods converting values to another that corresponding them (int and string for our case).

```ruby
def sum(a, b)
  a.to_i() + b.to_i()
end

something = nil
anything  = "2"
print sum(something, anything) # Will print 2
                               # because nil was converted to 0
```

The ``NilClass`` source file is also available in the [language core package](https://github.com/rubinius/rubinius/blob/master/core/nil.rb), so a ``nil`` becomes an explicit thing.

Besides that, a `nil` and `false` are only things that "falsy" in Ruby, so null-checks are the same as in C or a JavaScript:

```ruby
array = [1,2,3,4,5]
if array[999] && array[999].size
  # Will go there only if array has 999th element and it has size
end
```

Ruby also implements the safe invocation feature, which works the same as in Kotlin, but its operator is ``&.`` instead.

```ruby
something = nil
print(something&.value, "text") # Will print text
print(something.value, "text") # NoMethodError

# Short version of previous if
if array[999]&.size
  # Will go there only if array has 999th element and it has size
end
```

## Заключение

I really hope that this article will help someone to understand the null and its issues. The best solution for the null issue - is being noticed about that. Some people getting [too impressed](https://www.yegor256.com/2014/05/13/why-null-is-bad.html) when they realise the issue, and starting to cover all the code with optionality and exceptions. The good programmer knows the language and then he knows how to code well and how to ``null`` well.

Pay attention to ``null``s, they're more than you think they are.

## Дополнительно: Еще немного об Optional

As I already mentioned - an Optional provides really good functional interface which could change your way to work with it. One of those ways - is to understand an Optional as a collection with maximum possible size of one. Comparing to a ``List``, for instance, which might contain one or more or zero elements, all its methods might be applied to an Optional, if it had those. So technically, ``Optional`` is a container.

When you think that way, calling a ``get()`` is starting to sound like absurd, like ``get(0)`` from a collection with unknown size. When you do this:

```java
if (!list.isEmpty()) doSometh(list.get(0));
```

it's semantically the same as this:

```java
if (optional.isPresent()) doSomth(optional.get());
```

And you switching context back, from collection to a **single** element. So the thing is to avoid using ``get()`` and ``isPresent()`` methods, because they turn you back to an inconvenient imperative way of working with the concept of optionality. 

Why is that inconvenient? Here's the case: we have a function that returns a user name by id, considering that all the context goes under the hood and we have a method that returns users list from DB. Let's look on default implementation:

```java
public static Optional<String> /*or String*/ getUserNameById(Integer id) {
    if (id != null) {
        final List<User> list = getListFromDB();
        if (list != null) {
            for (User u : list) {
                if (Objects.equals(u.id(), id))
                    return ofNullable(u.name()); // or u.name()
            }
        }
    }

    return Optional.empty(); // or null
}
```

Personally, I never write that deep-nesting code, so for me, it's better to write it like:

```java
public static Optional<User> getUserNameById(Integer id) {
    if (id == null) return Optional.empty();

    final List<User> list = getListFromDB();
    if (list == null) return Optional.empty();

    for (User u : list) {
        if (Objects.equals(u.id(), id))
            return ofNullable(u.name());
    }

    return Optional.empty();
}
```

In my personal opinion, it's more a readable imperative code. Anyway, what if we will avoid all the imperativeness and create hardcore functional code?

```java
public static Optional<String> getUserNameById(Integer id) {
    return ofNullable(id).flatMap(v -> ofNullable(getListFromDB())
            .flatMap(list -> list.stream().filter(u -> 
                     Objects.equals(u.id(), id))
            .findAny().map(User::name)));
}
```

See, there are opportunities to chain multiple optional operations that depend on each previous, perfectly combined with streams and its functions. It looks like Monad ([if I actually understand what is this though](https://stackoverflow.com/questions/70453016/why-are-monads-hard-to-explain)), to have an ability to exclude inexistence from the operation you do (or a branch of operation).

## Источники и другие ссылки

### Данные о багах в проектах

| Project          | Source                                                       | Search request for all                                     | Search request for NPEs                                                          |
| ---------------- | ------------------------------------------------------------ | ---------------------------------------------------------- | -------------------------------------------------------------------------------- |
| IntelliJ IDEA    | [JetBrains YT](https://youtrack.jetbrains.com/issues/idea)   | Exception                                                  | NullPointerException                                                             |
| Minecraft        | [Mojang Jira](https://bugs.mojang.com)                       | summary ~ "\*Exception\*" OR description ~ "\*Exception\*" | summary ~ "\*NullPointerException\*" OR description ~ "\*NullPointerException\*" |
| Eclipse Platform | [Eclipse Bugzilla](https://bugs.eclipse.org)                 | Product: Platform Content: NullPointerException            | Content: Exception Product: Platform                                             |
| Jenkins          | [Jenkins Jira](https://issues.jenkins.io)                    | text ~ "NullPointerException"                              | text ~ "Exception"                                                               |
| GraalVM          | [Graal GitHub](https://github.com/oracle/graal/issues)       | https://github.com/oracle/graal/issues?q=Exception         | https://github.com/oracle/graal/issues?q=NullPointerException                    |
| Unity 3D         | [Unity issues](https://unity3d.com/search?refinement=issues) | https://unity3d.com/search?refinement=issues&gq=exception  | https://unity3d.com/search?refinement=issues&gq=nullreferenceexception           |

### Мнения людей

[Джон Скит](https://github.com/jskeet) (Разработчик, Писатель) [ref](https://en.wikipedia.org/wiki/Special:BookSources?isbn=978-1617294532)

> The consistent use of a single type to represent a possibly missing value enables the language to make our lives easier, and library authors have an idiomatic way of representing it in their API surface, too.

> Если правильно использовать возможность отсутсвия значения вне зависимости от типа, то жизнь станет проще, и авторы библиотек тоже будут иметь один единый способ для этого.

[Андерс Хейлсберг](https://en.wikipedia.org/wiki/Anders_Hejlsberg) (Ведущий архитектор по разработке C# в Microsoft): [ref](https://www2.computerworld.com.au/article/261958/a-z_programming_languages_c_/?pp=3)

> 50% of the bugs that people run into today, coding with C# in our platform, and the same is true of Java for that matter, are probably null reference exceptions. If we had had a stronger type system that would allow you to say that ‘this parameter may never be null, and you compiler please check that at every call, by doing static analysis of the code’. Then we could have stamped out classes of bugs.

> 50% всех багов с которыми мы сегодня сталкиваемся в С# или даже в Java, наверное исключения вызванные null-ссылками. Если бы у нас была более строгая типизация и мы бы могли явно сказать что "вон та штука не может быть null, а ты, компилятор, проследи за этим во время статического анализа", то мы бы избавились от целой категории багов.

[Роберт Мартин](https://en.wikipedia.org/wiki/Robert_C._Martin) (Инженер ПО, автор [Agile](https://en.wikipedia.org/wiki/Agile_Manifesto "Agile Manifesto")): [ref](https://en.wikipedia.org/wiki/Special:BookSources/0-13-597444-5)

> Most of us have also been burned by forgetting to test for null. Common though the idiom may be, it is ugly and error prone.

> Многие из нас обжигались из-за того что забывали проверять что-то на null. Пусть даже null используют все, он все равно уродливый и вызывает ошибки. 

[Мартин Фаулер](https://en.wikipedia.org/wiki/Martin_Fowler_(software_engineer)) (Инженер ПО, писатель): [ref](https://martinfowler.com/eaaCatalog/specialCase.html)

> Nulls are awkward things in object-oriented programs because
> they defeat polymorphism.

> Null не вписывается в объектно-ориентированные программы, потому что он ломает полиморфизм.

[Дуг Ли](https://en.wikipedia.org/wiki/Doug_Lea) (Профессор области computer science): [ref](https://github.com/google/guava/wiki/UsingAndAvoidingNullExplained)

> Null sucks.