## Functional programming for the lazy C# programmer
##### <span style="font-family:Helvetica Neue; font-weight:bold">The power of functions today!</span>

---

## Functions

A function transforms a value of some type into another value

```haskell
length : String -> Int
toUpper : String -> String
```
```csharp
int Length(string str);
string ToUpper(string str);
```

+++

## Functions

A function should transform, not modify

Modifying input parameters means surprises for the caller

+++ 

## Functions

A function should always return something

* A void method does not transform its input
* Probably uses side-effects
* Can not be chained

**Avoid using `void` in function signature!**

+++

### Pure functions

A function is pure if it the result of the function only depends on its inputs:

```csharp
int Length(string str) => str.Length;
string ToUpper(string str) => str.ToUpper();
```

+++ 

### Non-pure functions

Code using some global state:

```csharp
string FormatLogMessage(string message) => 
    String.Format("{0}: {1}", DateTime.Now, message);
```

+++

### Non-pure functions

Code accessing local variables:

```csharp
string IsEmailTheSame(string email) =>
    String.Equals(this.Email, email);
```

+++

### Non-pure functions

Code accessing a database:

```csharp
User GetUser(string id) =>
    repository.ById(id);
```

+++

### Why does it matter if it's pure?

Given the same inputs a pure function will always return the same output:

```csharp
Length("abcde"); // 5
Length("abcde"); // 5
Length("abcde"); // 5
```

+++ 

### Why does it matter if it's pure?

Non-pure functions make no such promises:

```csharp
FormatLogMessage("Hello World!"); // 2017-06-21 13:48:22: Hello, World!
FormatLogMessage("Hello World!"); // 2017-06-21 13:48:23: Hello, World!
FormatLogMessage("Hello World!"); // 2017-06-21 13:48:25: Hello, World!

obj.IsEmailTheSame("santa.clause@christmas.com"); // true
obj.Email = "kris.kringle@christmas.com";
obj.IsEmailTheSame("santa.clause@christmas.com"); // false

GetUser("1234"); // User { Id = "1234", Name = "Santa" };
GetUser("1234"); // User { Id = "1234", Name = "Kris" };
```

+++

### Why does it matter if it's pure?

Pure functions

* limit the amount of state we need to remember
* contain no surprises
* are easy to test

+++

### Making it pure

Non-pure functions can be made pure by passing global state as parameters instead:

```csharp
string FormatLogMessage(DateTime date, string message) => 
    String.Format("{0}: {1}", date, message);

string IsEmailTheSame(string currentEmail, string newEmail) =>
    String.Equals(currentEmail, newEmail);
```

+++

### Making it pure

What about database access?

```csharp
User GetUser(string id) =>
    repository.ById(id);
```

+++

### What about objects?

When compiled, all instance methods always take a hidden parameter called `this`!

The dot-operator is a "convenience".

Extension methods reflect this fact:

```csharp
string DoubleLength(this string self) => self.Length * 2;
```

---

## Lambdas in C&#35;

* `Func<T1..TN, TResult>`
* `Action<T1..TN>`
* Delegate types

+++

### `Func<T1..TN, TResult>`

* Support max 8 inputs (`Func<T1, T2, T3, T4, T5, T6, T7, T8, TResult>`)
* Last generic type is the return type

+++

### `Func<T1..TN, TResult>`

```csharp
Func<string, int> toInt;

toInt = str => Convert.ToInt32(str);
toInt = new Func<string, int>(Convert.ToInt32);
toInt = Convert.ToInt32;

toInt("123"); // 123
```
+++

### `Action<T1..TN>`

*  Pretty much the same as `Func` but...
*  Has `void` return type!

+++

### `Action<T1..TN>`

Sometimes useful to make C# syntax look nice:

```csharp
Options Configure(Action<Options> configuration)
{
    return Configure(opts => {
        configuration(opts);

        return opts;
    });
}

Options Configure(Func<Options, Options> configuration)
{
    return configuration(new Options());
}

Configure(opts => opts.Timeout = TimeSpan.FromSeconds(30));
```

+++

### Delegate types

`Func` syntax can look overwhelming:

```csharp
Func<string, User> users = null;
```

Maintaining the signature for more than a couple of functions can be hell.

+++

### Delegate types

Defining delegates lets us get a nice type signature:

```csharp
public delegate User UserRepository(string id);

public static Changes Handle(UserRepository repository, 
    RegisterUser user);

```

To modify all usages, just modify the delegate type!

+++

### Delegate types

Type conversion kind of sucks:

```csharp
Func<string, User> users1 = id => new User(id);
UserRepository user2 = id => new User(id);

UserRepository users3 = users1; // Compiler error
UserRepository users3 = users1.Invoke; // Works!

Func<string, User> users3 = users2; // Compiler error!
Func<string, User> users3 = users2.Invoke; // Works
```

+++

### Behind the scenes

All lambdas or delegates are actually compiled to a class.

```csharp
var captureMe = 3;

Func<int, int ,int> sum = (lhs, rhs) => lhs + rhs + captureMe;
```

+++

### Behind the scenes

```csharp
public class FuncDelegate982398202 // Random name
{
    public int captureMe; // Captured variables

    public int Invoke(int lhs, int rhs) // Arguments
    {
        return lhs + rhs; // Lambda body
    }
}

FuncDelegate982398202 d = new FuncDelegate982398202();

d.captureMe = captureMe;

Func<int, int ,int> sum = d.Invoke;
```

---

## The fun in functions

* Composition
* Higher order functions
* Dealing with many things
* Dealing with absence

+++


### Higher order functions

A function which:

* accepts other functions as parameters
* returns a new function
* or both

+++

### Higher order functions

```csharp
public Task<IActionResult> ValidateHash<TRequest>(string hash, Func<Task<IActionResult>> handler)
{
    if (HashIsValid(hash))
        return handler();

    return Task.FromResult(Unauthorized());
}
```

+++

### Higher order functions

```csharp
Func<DateTime, string> CustomDateFormatter(string format)
{
    return dateTime => dateTime.ToString(format);
}

Func<DateTime, string> CustomDateFormatter(string format) => dateTime => dateTime.ToString(format)
```

+++

### Higher order functions

```csharp
public Func<User, string> IfActive(Func<User, string> format) =>
    user => user.Active ? format(user) : "Not active";
```

+++

### Composition

Functions can be composed to create new functions:

```csharp
public static Func<TIn1, TReturn> AndThen<TIn1, TIn2, TReturn>(this Func<TIn1, TIn2> self, Func<TIn2, TReturn> next) =>
    arg => next(self(arg));
```

```haskell
andThen f g x = g(f(x))
```

+++

### Composition

```csharp
Func<string, string> trim => str => str.Trim();
Func<string, string> toUpper => str => str.ToUpper();

Func<string, string> trimToUpper = trim.AndThen(toUpper);

trimToUpper("  abcd  "); // "abcd"
```

+++

### Currying

Turn a function with many parameters into many higher-order functions with only one parameter each

+++

### Currying

```csharp
Func<UserRepository, RegisterUser, User> Handle =>
    (repository, command) => repository(command.Id)
        .Map(user => /* Business logic */);

Func<UserRepository, Func<RegisterUser, User>> Handle =>
    repository => command => repository(command.Id)
        .Map(user => /* Business logic */);
```


+++

### Currying

```csharp
UserRepository repository = id => Some(new User(id));
var commandHandler = Handle(repository);

// Our command handler can no be called with same repository
// repeatedly
commandHandler(new RegisterUser { Id = "0123" });
commandHandler(new RegisterUser { Id = "1234" });
```

+++

### Filtering many things the C# way

```csharp
IEnumerable<User> GetActiveUsers(IEnumerable<User> users)
{
    var result = new List<User>();

    foreach (var user in users)
    {
        if (user.Active)
        {
            result.Add(user);
        }
    }

    return result;
}
```

+++

### Transforming many things the C# way

```csharp
IEnumerable<string> GetEmails(IEnumerable<User> users)
{
    var result = new List<string>();

    foreach (var user in users)
    {
        result.Add(user.Email);
    }

    return result;
}
```

+++

### Aggregating many things the C# way

```csharp
int GetTotalMessagesSent(IEnumerable<User> users)
{
    var result = 0;

    foreach (var user in users)
    {
        result += user.MessagesSent;
    }

    return result;
}
```

+++

### Dealing with many things the C# way

* The examples all look the same
* A lot of duplicated logic
* Most of the code is boilerplate

+++

### Filtering many things with functions

```csharp
IEnumerable<User> Filter(IEnumerable<User> u, Func<User, bool> predicate)
{
    foreach (var user in u)
    {
        if (predicate(user))
        {
            yield return user;
        }
    }
}

var activeUsers = Filter(users, user => user.Active);
```

+++

### Transforming many things with functions

```csharp
IEnumerable<User> Map(IEnumerable<User> u, Func<User, string> transformation)
{
    foreach (var user in u)
    {
        yield return transformation(user);
    }
}

var userEmails = Map(users, user => user.Email);
```

+++

### Aggregating many things the functional way

```csharp
int Fold(IEnumerable<User> u, Func<int, User, int> aggregator)
{
    var result = 0;

    foreach (var user in u)
    {
        result = aggregator(result, user);
    }

    return result;
}

var totalMessagesSent = Fold(users, (total, user) => total + user.MessagesSent);
```

+++

### Dealing with many things the functional way

Generic implementations:

```csharp
IEnumerable<T> Filter<T>(this IEnumerable<T> self, Func<T, bool> predicate);
IEnumerable<R> Map<T, R>(this IEnumerable<T> self, Func<T, R> mapper);
TAcc Fold<T, TAcc>(this IEnumerable<T> self, TAcc initialValue, Func<TAcc, T, TAcc> aggregator);
```

+++

### Dealing with many things the functional way

*  Filter/Map/Fold are standard functions
*  Available in most languages with support for lambdas (incl. JavaScript)
*  Fold is somtimes called Reduce (and comes in two flavors: left and right fold)

+++

### Dealing with many things the LINQ way

* `Filter` -> `Where`
* `Map` -> `Select`
* `Fold` (a.k.a. `Reduce`) -> `Aggregate`

+++

### Dealing with many things

Filter, map and fold can be chained:

```csharp
var to = users
    .Where(u => u.IsActive)
    .Select(u => u.Email)
    .Aggregate("", (to, email) => to + email + ";");
```

> LINQ is actually lazy, so it will only iterate over the
> items once and not until it has to (e.g. `ToList` or `Aggregate` is called)

+++

### Dealing with absence of things

How do you design a API which may or may not return a value?

* Need to communicate expectations on implementations
* Need to communicate expectations to callers

+++

### Example interface

```csharp
interface UserRepository
{
    User ById(string id);
}
```

* What is the implementer expected to return when there is no user?
* What will the caller expect the return value to be when there is no user?

+++

### Using null for abscence

```csharp
class SomeUserRepository : IUserRepository
{
    public User ById(string id)
    {
        if (id == "1")
        {
            return new User(id);
        }
        else 
        {
            return null;
        }
    }
}
```

+++

### Using null for abscence

```csharp
var user = userRepository.ById("2");

Console.WriteLine(user.Name);
```

```
NullReferenceException: Object reference not set to an instance of an object
```

+++

### Using null for abscence

* As an implementor:
    * Should I return null? 
    * Will my callers do proper null-checking?
* As a caller: 
    * Do I need to null check? 
    * Does the implementation return null?

+++

### Using null for abscence

Seriously, who null-checks everything?

+++

### Using null for absence

Rules for using null:

1. Never, ever use `null`.
2. If you have to use `null`, see rule 1.
3. If your reeeally have to, make sure it is a comparison.
4. If you still re-he-he-heally have to, you better be doing serialization

> PS: Using `undefined` instead is just as bad, if not worse
> because then you are using JavaScript. DS.

+++

### A better way?

Lists could model absence:

```csharp
interface IUserRepository
{
    IEnumerable<User> ById(string id);
}
```

+++

### A better way?

```csharp
class SomeUserRepository : IUserRepository
{
    public IEnumerable<User> ById(string id)
    {
        if (id == "1")
        {
            return new [] { new User(id) };
        }
        else {
            return Enumerable.Empty<User>();
        }
    }
}
```

+++

### A better way?

```csharp
var user = userRepository.ById("2");

Console.WriteLine(user.Name); // Won't compile

// Look ma', no NullReferenceExceptions
var name = user
    .Select(u => u.Name)
    .Select(u => u.ToUpper())
    .SingleOrDefault()
    ?? "No such user"; // <- Unless we forget this

Console.WriteLine(name);
```

+++

### A better way?

Benefits:

* Compile-time safe
* No need for null-checks
* Implementers know how to return no user
* Callers know they need to deal with no user

+++

### A better way?

Downsides:

* Implementers can return 2, 5 or 107 users
* The `FirstOrDefault`/`SingleOrDefault` gets us right back into `null` land

+++

### A better way!

We need a special case of a list, allowing 0 or 1 instance!

+++

### A better way!

```csharp
public class Maybe<T>
{
    Maybe<T> Some(T value) => new Maybe<T>(value);
    Maybe<T> None() => new Maybe<T>();

    Maybe<R> Map(Func<T, R> mapper);
    Maybe<R> Bind<Func<T, Maybe<R>> binder);
    T Or(Func<T> defaultValue);
}
```

+++

### A better way!

Lists could model absence:

```csharp
interface IUserRepository
{
    Maybe<User> ById(string id);
}
```

+++

### A better way!

```csharp
class SomeUserRepository : IUserRepository
{
    public Maybe<User> ById(string id)
    {
        if (id == "1")
        {
            return Maybe<User>.Some(new User(id));
        }
        else 
        {
            return Maybe<User>.None();
        }
    }
}
```

+++

### A better way!

```csharp
var user = userRepository.ById("2");

Console.WriteLine(user.Name); // Won't compile

// Look ma', no NullReferenceExceptions
var name = user
    .Map(u => u.Name) 
    .Map(name => name.ToUpper())
    .Or(() => "No such user");

Console.WriteLine(name);
```

+++

### A better way!

How come lists and our Maybe-type are so similar?

They are part of a category of types called `Functors`. 

+++

### Chaining things with Maybe

Sometimes we want to call another function returning a `Maybe`.

```csharp
Maybe<char> FirstLetter(string input) =>
    input.Length > 0 ? Maybe<char>.Some(input[0]) : Maybe<char>.None;

// Won't compile, return type is actually Maybe<Maybe<char>>
Maybe<char> user.Map(u => u.Name)
    .Map(name => name.ToUpper())
    .Map(FirstLetter);
```

+++

### Chaining things with Maybe

The `Bind`-function:

```csharp
Maybe<R> Bind<R>(Func<T, Maybe<R>> binder) =>
    hasValue ? binder(value) : Maybe<R>.None();

Maybe<R> Map<T>(Func<T, R> mapper) =>
    hasValue ? Maybe<R>.Some(mapper(value)) : Maybe<R>.None();
``` 

+++

### Binding is like a railroad

```csharp
using static Maybe<int>; // Expose Some and None.

var maybeInt = Some(0);

maybeInt // Some 0
    .Bind(v => Some(v + 1)) // Some 1
    .Bind(v => Some(v + 1)) // Some 2
    .Bind(v => None())      // None
    .Bind(v => Some(v + 1)) // None
    .Bind(v => Some(v + 1)) // None
``` 

+++

### What about lists?

They can do it too. In LINQ, `Bind` is called `SelectMany`:

```csharp
var list = new List<int>()
{
    1, 2, 3
};

list
    .Select(count => Enumerable.Range(0, count))
    .ToList(); // [[0], [0, 1], [0, 1, 2]]

list
    .SelectMany(count => Enumerable.Range(0, count))
    .ToList(); // [0, 0, 1, 0, 1, 2]
```

---

## Simpler code with functions

By using what we have learned, we can do more while typing less!

+++

### Configurable behaviors

Implementing a backoff algorithm for failure recovery.

```csharp
interface IBackOff
{
    TimeSpan Calculate(int attempt);
}

class ConstantBackOff : IBackOff
{ 
    private TimeSpan backoff;

    public ConstantBackOff(TimeSpan backoff)
    {
        this.backoff = backoff;
    }

    public TimeSpan Calculate(int attempt)
    {
        return backoff;
    }
}

class ExponentialBackOff : IBackOff { }
class ExponentialBackOffWithRandomVariance : IBackOff { }

```

+++

### Configurable behaviors

![Such code. Many sigh!](assets/simpsons-scream.jpg)

+++

### Configurable behaviors

```csharp
delegate TimeSpan BackOff(int attempt);

class BackOffs
{
    public static BackOff Constant(TimeSpan value) =>
        _ => value;

    public static BackOff Exponential(TimeSpan start) =>
        attempt => /* Calculations! */;

    public static BackOff RandomVariance(Random random, BackOff otherFunction) =>
        otherFunction.AndThen(timeSpan => /* add some variance */);

}
```

+++

### Cross-cutting concerns

```csharp
var commandHandler = 
    Handle<RegisterUser>(c => 
        Users.Handle(Database.Users.ById(connectionString), c)
    )
    .With(Authorization.InRole(Roles.Administrator))
    .With(Logging.Command)
    .AndThen(Logging.Changes);

commandHandler(new RegisterUser 
{
    UserId = "chunky_santa86"
});
```

+++

### Mock-free testing

```csharp
[Fact]
public void GivenAPreCondition_WhenDoingCoolStuff_ShouldBeMuchWow()
{
    var result = Users.Handle(_ => new User("1234"), new RegisterUser
    {
        UserId = "1234"
    });

    Assert.IsType<UserAlreadyExists>(result.ErrorOrDefault());
}
```

+++

### But what about code organization?

* Classes are often used to group code together

    * Repositories contains our database logic
    * Our application service contains all business logic
    * Etc.

* Makes it easier to find similar functionality!

+++

### But what about code organization?

No problem:

```csharp
public static class Users
{
    public static User ById(string id) => 
        Execute(Queries.UsersById(id));

    public static User Save(User user) => 
        Execute(Queries.SaveUser(user));
}

UserRepository userRepository = Users.ById;
```

+++

### But what about code organization?

With dependency injection:

```csharp
container.Register<UserRepository>(Users.ById);
```