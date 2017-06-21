## Functional design patterns for the lazy C# programmer
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
public static string DoubleLength(this string self) => self.Length * 2;
```

---

## Lambdas in C#

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