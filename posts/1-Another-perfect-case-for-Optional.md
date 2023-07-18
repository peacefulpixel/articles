*This post is related to my article [The issue of null](https://ppixel.me/blog/2023/07/16/The-issue-of-null.html) ([mirror](https://github.com/peacefulpixel/articles/blob/main/articles/the-issue-of-null/article.md)).*

People still doubt that Java's ``Optional`` could be used effectively, and I understand them. Honestly, I was one of them, before I figured the power of ``.map()``&``.flatMap()`` methods. I want to describe a case that I got today, and also want **you**, reader, to try to associate that with your cases.

*All the code is simplified for understanding (I removed obvious logic like `try/catch` blocks and keywords like `private`)*

## What we had

A JavaFX application that has the following architecture for resource management:

```java
abstract class Resource<T> {
    T lazyVal = null;

    abstract String getRelativePath();
    abstract T build(URL url);
    
    public T get() {
        return askFor().orElseThrow();
    }

    public Optional<T> askFor() {

        if (lazyVal == null) {
            final URL url = Resource.class
                    .getResource("/com/example/" + getRelativePath());

            if (url != null)
                lazyVal = build(url);
        }

        return lazyVal == null ? Optional.empty() : Optional.of(lazyVal);
    }
}
```

And we got multiple implementations like ``CSSResource``, ``FXMLResource`` and ``ImageResource`` for instance. Each of those implements ``getRelativePath()`` and ``build(URL url)`` for the specific type ``T`` which is a ``javafx.scene.image.Image`` for ``ImageResource`` for example.

So the thing is that ``ImageResource`` produces an image with ``Image build(URL url)``, that's it.

## What we gotta do

JavaFX has a common issue with a system tray. In short words - it requires using an AWT like the Swing does. Honestly, I don't know why, because Im coding at Java 17 and using latest JFX, but the system tray is still not implemented. The team planning to use a native solution in the future, but now we need to have a prototype, which should be codded with nearest possible way - which is unfortunately an AWT.

To instantate a tray menu in AWT, we have to create a ``new TrayIcon(image)`` instance, where the image is ``Image``, but not a `javafx.scene.image.Image`. Yep, it's another image class, that is from AWT and they both obviously have no any compatibility 🙄

Now, our ``ImageResource`` should temporary be a resource for ``java.awt.Image`` and ``javafx.scene.image.Image`` both, to have a decent code which might be still even in release version.

## The result

Solution I see is to add ``askForAWTImage()`` method into a ``ImageResource``, but I really don't wanna copy resource retrieving logic, eventually because it contains classpath that I don't want to exclude in constant, because, by design, the only method should be dependent on it.

So we move actual resource retrieving logic to a separate method:

```java
protected URL getResource() {
    return Resource.class.getResource("/com/example/" + getRelativePath());
}
```

And here, the URL could be a ``null``, if a resource doesn't exist and also, the only dependent code on it yet, does the nullity check, so semantically that ``URL`` is optional. Well, I wrap it:

```java
protected Optional<URL> getResource() {
    return Optional.ofNullable(Resource.class.getResource(
            "/com/example/" + getRelativePath()));
}
```

Then I'm going to ``askFor()`` method and realize that it's so dumb, so I just use a single ``flatMap()`` to simplify it:

```java
public final Optional<T> askFor() {
    return Optional.ofNullable(lazyVal).or(() -> 
            getResource().map(url -> lazyVal = buildFromURL(url)));
}
```

Well, that's it. Just a reminder that the previous version was:

```java
public final Optional<T> askFor() {
    if (lazyVal == null) {
        final URL url = Resource.class.getResource("/com/example/" + getRelativePath());
        if (url != null)
            lazyVal = buildFromURL(url);
    }

    return lazyVal == null ? Optional.empty() : Optional.of(lazyVal);
}
```

## Conclusion

Comparing previous and current solutions, I can find the following benefits:

- Decreased nesting (+readability, +supportrability)

- Decreased code amount (+readability)

- Two simple methods instead of one combined (+flexibility)

My target was to improve flexibility for ``ImageResource`` implementation and, I guess, I found a most elegant way to reach it.


