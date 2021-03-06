<!--- Copyright (C) 2009-2013 Typesafe Inc. <http://www.typesafe.com> -->
# Action composition

This chapter introduces several ways to define generic action functionality.

## Reminder about actions

Previously, we said that an action is a Java method that returns a `play.mvc.Result` value. Actually, Play manages internally actions as functions. Because Java doesn't yet support first class functions, an action provided by the Java API is an instance of `play.mvc.Action`:

```java
public abstract class Action {
    
  public abstract Promise<SimpleResult> call(Http.Context ctx);    
    
}
```

Play builds a root action for you that just calls the proper action method. This allows for more complicated action composition.

Notice that the `call` method returns `Promise<SimpleResult>`, this was introduced in [[version 2.2|Highlights22]] to improve handling different result type such as chunked, plain or async. 

## Composing actions

You can compose the code provided by the action method with another `play.mvc.Action`, using the `@With` annotation:

```java
@With(VerboseAction.class)
public static Result index() {
  return ok("It works!");
}
```

Here is the definition of the `VerboseAction`:

```java
public class VerboseAction extends Action.Simple {

  public Result call(Http.Context ctx) throws Throwable {
    Logger.info("Calling action for " + ctx);
    return delegate.call(ctx);
  }
}
```

At one point you need to delegate to the wrapped action using `delegate.call(...)`.

You also mix several actions by using custom action annotations:

```java
@Authenticated
@Cached(key="index.result")
public static Result index() {
  return ok("It works!");
}
```

> **Note:**  ```play.mvc.Security.Authenticated``` and ```play.cache.Cached``` annotations and the corresponding predefined Actions are shipped with Play. See the relevant API documentation for more information.


## Defining custom action annotations

You can also mark action composition with your own annotation, which must itself be annotated using `@With`:

```java
@With(VerboseAction.class)
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface Verbose {
   boolean value() default true;
}
```

You can then use your new annotation with an action method:

```java
@Verbose(false)
public static Result index() {
  return ok("It works!");
}
```

Your `Action` definition retrieves the annotation as configuration:

```java
public class VerboseAction extends Action<Verbose> {

  public Result call(Http.Context ctx) {
    if(configuration.value) {
      Logger.info("Calling action for " + ctx);  
    }
    return delegate.call(ctx);
  }
}
```

## Annotating controllers

You can also put any action composition annotation directly on the `Controller` class. In this case it will be applied to all action methods defined by this controller.

```java
@Authenticated
public Admin extends Controller {
    
  …
    
}
```

## Passing objects from action to controller

You can pass an object from an action to a controller by utilizing the context args map.

```java
public class SecurityAction extends Action<Verbose> {

  public Result call(Http.Context ctx) {
    ctx.args.put("user", User.findById(1234));
    return delegate.call(ctx);
  }
}
```

> **Next:** [[Content negotiation | JavaContentNegotiation]]
