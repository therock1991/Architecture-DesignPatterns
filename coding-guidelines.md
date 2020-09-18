# 1. Coding Guidelines

- This document to help write clean code steps and supported with StyleCop and TSLint library
- **[Back-end]** StyleCop helps indicate style and consistency rule violations such as documentation, layout, naming, ordering, readability, spacing, and so forth
- **[Front-end]** TSLint

# 2. Installations

- **C#** : Install nuget package StyleCop.Analyzers
- **JS** : TSLint

### 2.1 TSLint

### 2.2 StyleCop

- Install nuget package StyleCop.Analyzers
- **Settings/.editorconfig** - helps maintain consistent coding styles and control more settings in the project.

### 2.3 Examples

```csharp
# editorconfig.org

[*.cs]
charset = utf-8-bom
insert_final_newline = true
indent_style = space
indent_size = 4

# Sort using and Import directives with System.* appearing first
dotnet_sort_system_directives_first = true

# Always use "this." and "Me." when applicable; let StyleCop Analyzers provide the warning and fix
dotnet_style_qualification_for_field = true:none
dotnet_style_qualification_for_property = true:none
dotnet_style_qualification_for_method = true:none
dotnet_style_qualification_for_event = true:none

# SA1200: Using directives should be placed correctly
dotnet_diagnostic.SA1200.severity = none

# SA1600: Elements should be documented
dotnet_diagnostic.SA1600.severity = none

```

- More details : <https://github.com/DotNetAnalyzers/StyleCopAnalyzers/blob/master/.editorconfig>

# 3. Highlights of Coding Standards

This section gives a highlight of some rules with both one that will possibly meet in daily coding life

### 3.1 Class Definition Order (Mandatory)

In the following order, from less restricted scope (public) to more restrictive (private)

- Field members (for example, member variables, const, etc.)
- Member functions
  - Constructors
  - Finalizer
  - Methods (Properties, Events, Operations, Overridables and Static)
  - Private nested types

### 3.2 Naming (Mandatory)

#### **DO** use plural form for namespaces

#### **DO** use PascalCasing for all public member, type, and namespace names consisting of multiple words

```csharp
PropertyDescriptor HtmlTag IOStream
```

#### **DO** use **camelCasing** for **variable and parameter names.**

```csharp
propertyDescriptor htmlTag ioStream
```

#### **DO** start with **underscore** for **private fields.**

```csharp
private readonly Guid _appKey = Guid.NewGuid();
```

#### **DO** start static **readonly** field and **constant** names with capitalized case

- private **static** readonly IEntityAccessor EntityAccessor = null;
- private **const** string MetadataName = "MetadataName";

#### **DO** use Async suffix in the asynchronous method names to notice people how to use it properly

```csharp
public async Task<string> LoadContentAsync() { ... }
```

#### **DO** use pronounceable names

**Bad:**

```csharp
var sTravelDate;
var cTime;
```

**Good:**

```csharp
var StartTravellingDate;
var CreatedTime;
```

#### **AVOID** using bad names - the name should reflect what it does and give context

**Bad:**

```csharp
int d;
```

**Good:**

```csharp
int modifiedDate;
```

#### **AVOID** misleading names - the name to reflect what it is used for

**Bad:**

```csharp
var dataFromDb = db.GetData().ToList();
```

**Good:**

```csharp
var employees = db.GetEmployees().ToList();
```

### 3.3 Variables (Mandatory)

#### **DO** use searchable names

**Bad:**

```csharp
Task.Delay(86400000);
```

**Good:**

```csharp
const MILLISECONDS_IN_A_DAY = 86400000;
Task.Delay(MILLISECONDS_IN_A_DAY);
```

#### **DO** use the same vocabulary for the same type of variable

**Bad:**

```csharp
GetUserInfo();
GetClientData();
GetCustomerRecord();
```

**Good:**

```csharp
GetUser();
```

#### **DO** use default arguments instead of short circuiting or conditionals

**Bad:**

```csharp
public void CreateMicrobrewery(string name = null)
{
    var breweryName = name ?? "Hipster Brew Co.";
    // ...
}
```

**Good:**

```csharp
public void CreateMicrobrewery(string breweryName = "Hipster Brew Co.")
{
    // ...
}
```

#### **DO NOT** add unneeded context

If your class/object name tells you something, don't repeat that in your variable name

**Bad:**

```csharp
public class Car
{
    public string CarModel { get; set; }
    public string CarColor { get; set; }
}
```

**Good:**

```csharp
public class Car
{
    public string Model { get; set; }
    public string Color { get; set; }
}
```

#### **AVOID** nesting too deeply and should return early

Too many if else statements can make code hard to follow

**Bad**

```csharp
public bool IsShopOpen(string day)
{
    if (!string.IsNullOrEmpty(day))
    {
        day = day.ToLower();
        if (day == "friday")
        {
            return true;
        }
        else if (day == "saturday")
        {
            return true;
        }
        else if (day == "sunday")
        {
            return true;
        }
        else
        {
            return false;
        }
    }
    else
    {
        return false;
    }
}
```

**Good**

```csharp
public bool IsShopOpen(string day)
{
    if (string.IsNullOrEmpty(day))
    {
        return false;
    }

    var openingDays = new[] { "friday", "saturday", "sunday" };
    return openingDays.Any(d => d == day.ToLower());
}
```

#### **AVOID** magic string - Should use enum, constant instead

**Bad:**

```csharp
if (userRole == "Admin")
{
    // logic in here
}
```

**Good:**

```csharp
public enum UserRole{
    Admin = 1,
    Guest = 2,
}
if (userRole == UserRole.Admin)
{
    // logic in here
}
```

#### **AVOID** mental mapping

**Bad:**

```csharp
var l = new[] { "Texas", "LA", "Washington DC" };

for (var i = 0; i < l.Count(); i++)
{
    var li = l[i];
    DoStuff();
    // ...

    //li for what?
    Dispatch(li);
}
```

**Good:**

```csharp
var locations = new[] { "Texas", "LA", "Washington DC"  };

foreach (var location in locations)
{
    DoStuff();
    // ...

    Dispatch(location);
}
```

### 3.4 Functions (Mandatory)

#### **DO** use function arguments (2 or fewer ideally)

Limiting the amount of function parameters is incredibly important because it makes testing your function easier

Having more than three leads to a combinatorial explosion where you have to test tons of different cases with each separate argument

Zero arguments is the ideal case. One or two arguments is ok, and three should be avoided

**Bad:**

```csharp
// On old browsers, each iteration with uncached `list.length` would be costly
// because of `list.length` recomputation. In modern browsers, this is optimized.
public void CreateMenu(string title, string body, string buttonText, bool cancellable)
{
    // ...
}
```

**Good:**

```csharp
public class MenuConfig
{
    public string Title { get; set; }
    public string Body { get; set; }
    public string ButtonText { get; set; }
    public bool Cancellable { get; set; }
}

var config = new MenuConfig
{
    Title = "Foo",
    Body = "Bar",
    ButtonText = "Baz",
    Cancellable = true
};

public void CreateMenu(MenuConfig config)
{
    // ...
}
```

#### **DO** make functions should do one thing

This is by far the most important rule in software engineering

When functions do more than one thing, they are harder to compose, test, and reason about

When you can isolate a function to just one action, they can be refactored easily and your code will read much cleaner

**Bad:**

```csharp
public void SendEmailToListOfClients(string[] clients)
{
    foreach (var client in clients)
    {
        var clientRecord = db.Find(client);
        if (clientRecord.IsActive())
        {
            Email(client);
        }
    }
}
```

**Good:**

```csharp
public void SendEmailToListOfClients(string[] clients)
{
    var activeClients = GetActiveClients(clients);
    Email(activeClients);
}

public List<Client> GetActiveClients(string[] clients)
{
    return db.Find(clients).Where(s => s.Status == "Active");
}
```

#### **DO** make functions should only be one level of abstraction

When you have more than one level of abstraction your function is usually doing too much

Splitting up functions leads to reusability and easier testing

**Bad:**

```csharp
public string ParseBetterJSAlternative(string code)
{
    var regexes = [
        // ...
    ];

    var statements = explode(" ", code);
    var tokens = new string[] {};
    foreach (var regex in regexes)
    {
        foreach (var statement in statements)
        {
            // ...
        }
    }

    var ast = new string[] {};
    foreach (var token in tokens)
    {
        // lex...
    }

    foreach (var node in ast)
    {
        // parse...
    }
}
```

**Good:**

```csharp
class Tokenizer
{
    public string Tokenize(string code)
    {
        // ...
        return tokens;
    }
}

class Lexer
{
    public string Lexify(string[] tokens)
    {
        // ...
        return ast;
    }
}

class BetterJSAlternative
{
    private string _tokenizer;
    private string _lexer;

    public BetterJSAlternative(Tokenizer tokenizer, Lexer lexer)
    {
        _tokenizer = tokenizer;
        _lexer = lexer;
    }

    public string Parse(string code)
    {
        var tokens = _tokenizer.Tokenize(code);
        var ast = _lexer.Lexify(tokens);
        foreach (var node in ast)
        {
            // parse...
        }
    }
}
```

#### **DO** make function callers and callees should be close

If a function calls another, keep those functions vertically close in the source file

Ideally, keep the caller right above the callee. We tend to read code from top-to-bottom, like a newspaper. Because of this, make your code read that way

**Bad:**

```csharp
class PerformanceReview
{
    private readonly Employee _employee;

    private IEnumerable<PeerReviews> GetPeerReviews()
    {
       var peers = LookupPeers();
        // ...
    }

    public PerfReviewData PerfReview()
    {
        GetPeerReviews();
        GetManagerReview();
        GetSelfReview();
    }

    public ManagerData GetManagerReview()
    {
        var manager = LookupManager();
    }

    public EmployeeData GetSelfReview()
    {
        // ...
    }
}

var  review = new PerformanceReview(employee);
review.PerfReview();
```

**Good:**

```csharp
class PerformanceReview
{
    private readonly Employee _employee;

    public PerfReviewData PerfReview()
    {
        GetPeerReviews();
        GetManagerReview();
        GetSelfReview();
    }

    private IEnumerable<PeerReviews> GetPeerReviews()
    {
        var peers = LookupPeers();
        // ...
    }

    private IEnumerable<PeersData> LookupPeers()
    {
        // ...
    }

    private ManagerData GetManagerReview()
    {
       var manager = LookupManager();
    }

    private ManagerData LookupManager()
    {
        // ...
    }

    private EmployeeData GetSelfReview()
    {
        // ...
    }
}

var review = new PerformanceReview(employee);
review.PerfReview();
```

#### **DO** encapsulate conditionals

**Bad:**

```csharp
if (article.state == "published")
{
    // ...
}
```

**Good:**

```csharp
if (article.IsPublished())
{
    // ...
}
```

#### **DO** remove dead code

Dead code is just as bad as duplicate code. There's no reason to keep it in your codebase. If it's not being called, get rid of it! It will still be safe in your version history if you still need it

**Bad:**

```csharp
public void OldRequestModule(string url)
{
    // ...
}

public void NewRequestModule(string url)
{
    // ...
}

var request = NewRequestModule(requestUrl);
```

**Good:**

```csharp
public void RequestModule(string url)
{
    // ...
}

var request = RequestModule(requestUrl);
```

#### **DO NOT** over-optimize

In Javascript, Modern browsers do a lot of optimization under-the-hood at runtime

A lot of times, if you are optimizing then you are just wasting your time

**Bad:**

```javascript
// On old browsers, each iteration with uncached `list.length` would be costly
// because of `list.length` recomputation. In modern browsers, this is optimized.
for (let i = 0, len = list.length; i < len; i++) {
  // ...
}
```

**Good:**

```javascript
for (let i = 0; i < list.length; i++) {
  // ...
}
```

#### **DO NOT** write to global functions

Polluting globals is a bad practice in many languages because you could clash with another library and the user of your API would be none-the-wiser until they get an exception in production

**Bad:**

```csharp
public string[] Config()
{
    return  [
        "foo" => "bar",
    ]
}
```

**Good:**

```javascript
class Configuration
{
    private string[] _configuration = [];

    public Configuration(string[] configuration)
    {
        _configuration = configuration;
    }

    public string[] Get(string key)
    {
        return (_configuration[key]!= null) ? _configuration[key] : null;
    }
}
// Load configuration and create instance of Configuration class
// And now you must use instance of Configuration in your application.
var configuration = new Configuration(new string[] {
    "foo" => "bar",
});
```

#### **AVOID** side effects (part 1)

A side effect could be writing to a file, modifying some global variable

The main point is to avoid common pitfalls like sharing state between objects without any structure.

**Bad:**

```csharp
// Global variable referenced by following function.
// If we had another function that used this name, now it'd be an array and it could break it.
var name = 'Ryan McDermott';

public string SplitIntoFirstAndLastName()
{
return name.Split(" ");
}

SplitIntoFirstAndLastName();

Console.PrintLine(name); // ['Ryan', 'McDermott'];
```

**Good:**

```csharp
public string SplitIntoFirstAndLastName(string name)
{
    return name.Split(" ");
}

var name = 'Ryan McDermott';
var splittedName = SplitIntoFirstAndLastName(name);

Console.PrintLine(name); // 'Ryan McDermott';
Console.PrintLine(splittedName); // ['Ryan', 'McDermott'];
```

#### **AVOID** side effects (part 2)

In JavaScript, primitives are passed by value and objects/arrays are passed by reference

In the case of objects and arrays, by adding an item to cart array to purchase, then any other function that uses that cart array will be affected by this addition

A great solution would be for the addItemToCart to always clone the cart, edit it, and return the clone. This ensures that no other functions that are holding onto a reference of the shopping cart will be affected by any changes.

**Bad:**

```javascript
const addItemToCart = (cart, item) => {
  cart.push({ item, date: Date.now() });
};
```

**Good:**

```javascript
const addItemToCart = (cart, item) => {
  return [...cart, { item, date: Date.now() }];
};
```

#### **AVOID** negative conditionals

**Bad:**

```csharp
public bool IsOutOfStock(string id)
{
    // ...
}

if (!IsOutOfStock(id))
{
    // ...
}
```

**Good:**

```csharp
public bool IsOutOfStock(string node)
{
    // ...
}

if (IsOutOfStock(node))
{
    // ...
}
```

#### **AVOID** conditionals

You can use polymorphism to achieve the same task in many cases

**Bad:**

```csharp
public class Employee
{
  public int GetSalary(EmployeeType type)
  {
     var salary = 0;

     switch (type)
     {
        case EmployeeType.Apprentice:
           salary = 10000;
           break;

        case EmployeeType.Developer:
           salary = 20000;
           break;

        case EmployeeType.Manager:
           salary = 30000;
           break;
     }

     return salary;
  }
}
public enum EmployeeType
{
    Manager = 0,
    Developer,
    Apprentice
}
static void Main(string[] args)
{
     var employee = new Employee();

     Console.WriteLine("Apprentice salary = " + employee.GetSalary(EmployeeType.Apprentice));
     Console.WriteLine("Developer salary = " + employee.GetSalary(EmployeeType.Developer));
     Console.WriteLine("Manager salary = " + employee.GetSalary(EmployeeType.Manager));
}
```

**Good:**

```csharp
public interface IEmployee
{
  int GetSalary();
}

public class Apprentice : IEmployee
{
    public int GetSalary()
    {
        return 10000;
    }
}

public class Developer : IEmployee
{
    public int GetSalary()
    {
        return 20000;
    }
}

public class Manager : IEmployee
{
    public int GetSalary()
    {
        return 30000;
    }
}

static void Main(string[] args)
{
    Console.WriteLine("Apprentice salary = " + new Apprentice().GetSalary());
    Console.WriteLine("Developer salary = " + new Developer().GetSalary());
    Console.WriteLine("Manager salary = " + new Manager().GetSalary());
}
```

#### **AVOID** type-checking (part 1)

**Bad:**

```csharp
public Path TravelToTexas(object vehicle)
{
    if (vehicle.GetType() == typeof(Bicycle))
    {
        (vehicle as Bicycle).PeddleTo(new Location("texas"));
    }
    else if (vehicle.GetType() == typeof(Car))
    {
        (vehicle as Car).DriveTo(new Location("texas"));
    }
}
```

**Good:**

```csharp
public abstract class Vehicle
{
   public abstract void Move(Location location);
}

class Bicycle: Vehicle
{
  public override void Move(Location location)
  {
    // ...
  }
}

class Car: Vehicle
{
  public override void Move(Location location)
  {
    // ...
  }
}

public Path TravelToTexas(Vehicle vehicle)
{
    vehicle.Move(new Location("texas"));
}
```

#### **AVOID** flags in method parameters

**Bad:**

```csharp
public void CreateFile(string name, bool temp = false)
{
    if (temp)
    {
        Touch("./temp/" + name);
    }
    else
    {
        Touch(name);
    }
}
```

**Good:**

```csharp
public void CreateFile(string name)
{
    Touch(name);
}

public void CreateTempFile(string name)
{
    Touch("./temp/"  + name);
}
```

### 3.5 Objects and Data Structures (Mandatory)

#### **DO** use getters and setters

**Bad:**

```csharp
class BankAccount
{
    public double Balance = 1000;
}

var bankAccount = new BankAccount();

bankAccount.Balance -= 100;
```

**Good:**

```csharp
class BankAccount
{
    private double _balance = 0.0D;

    pubic double Balance {
        get {
            return _balance;
        }
    }

    public BankAccount(balance = 1000)
    {
      _balance = balance;
    }

    public void WithdrawBalance(int amount)
    {
        if (amount > _balance)
        {
            throw new Exception('Amount greater than available balance.');
        }

        _balance -= amount;
    }

    public void DepositBalance(int amount)
    {
        _balance += amount;
    }
}

var bankAccount = new BankAccount();

bankAccount.WithdrawBalance(price);

// Get balance
balance = bankAccount.Balance;
```

#### **DO** make objects have private/protected members

**Bad:**

```csharp
class Employee
{
    public string Name { get; set; }

    public Employee(name)
    {
        Name = name;
    }
}

var employee = new Employee('John Doe');
Console.WriteLine(employee.Name) // Employee name: John Doe
```

**Good:**

```csharp
class Employee
{
    public string Name { get; }

    public Employee(string name)
    {
        Name = name;
    }
}

var employee = new Employee('John Doe');
Console.WriteLine(employee.GetName());// Employee name: John Doe
```

- **DO** prefer composition over inheritance

  As stated famously in Design Patterns by the Gang of Four, you should prefer composition over inheritance where you can..

  You might be wondering then, "when should I use inheritance?" It depends on your problem at hand, but this is a decent list of when inheritance makes more sense than composition:

  - Your inheritance represents an "is-a" relationship and not a "has-a" relationship (Human->Animal vs. User->UserDetails).
  - You can reuse code from the base classes (Humans can move like all animals).
  - You want to make global changes to derived classes by changing a base class. (Change the caloric expenditure of all animals when they move).

    **Bad:**

    ```csharp
    class Employee
    {
        private string Name { get; set; }
        private string Email { get; set; }

        public Employee(string name, string email)
        {
            Name = name;
            Email = email;
        }

        // ...
    }

    // Bad because Employees "have" tax data.
    // EmployeeTaxData is not a type of Employee

    class EmployeeTaxData : Employee
    {
        private string Name { get; }
        private string Email { get; }

        public EmployeeTaxData(string name, string email, string ssn, string salary)
        {
            // ...
        }

        // ...
    }
    ```

    **Good:**

    ```csharp
    class EmployeeTaxData
    {
        public string Ssn { get; }
        public string Salary { get; }

        public EmployeeTaxData(string ssn, string salary)
        {
            Ssn = ssn;
            Salary = salary;
        }

        // ...
    }

    class Employee
    {
        public string Name { get; }
        public string Email { get; }
        public EmployeeTaxData TaxData { get; }

        public Employee(string name, string email)
        {
            Name = name;
            Email = email;
        }

        public void SetTax(string ssn, double salary)
        {
            TaxData = new EmployeeTaxData(ssn, salary);
        }

        // ...
    }
    ```

### 3.6 SOLID (Mandatory)

#### **Single Responsibility Principle (SRP)**

As stated in Clean Code, "There should never be more than one reason for a class to change". It's tempting to jam-pack a class with a lot of functionality, like when you can only take one suitcase on your flight.

The issue with this is that your class won't be conceptually cohesive and it will give it many reasons to change. Minimizing the amount of times you need to change a class is important.

It's important because if too much functionality is in one class and you modify a piece of it, it can be difficult to understand how that will affect other dependent modules in your codebase.

**Bad:**

```csharp
class UserSettings
{
    private User User;

    public UserSettings(User user)
    {
        User = user;
    }

    public void ChangeSettings(Settings settings)
    {
        if (verifyCredentials())
        {
            // ...
        }
    }

    private bool VerifyCredentials()
    {
        // ...
    }
}
```

**Good:**

```csharp
class UserAuth
{
    private User User;

    public UserAuth(User user)
    {
        User = user;
    }

    public bool VerifyCredentials()
    {
        // ...
    }
}

class UserSettings
{
    private User User;
    private UserAuth Auth;

    public UserSettings(User user)
    {
        User = user;
        Auth = new UserAuth(user);
    }

    public void ChangeSettings(Settings settings)
    {
        if (Auth.VerifyCredentials())
        {
            // ...
        }
    }
}
```

#### **Liskov Substitution Principle (LSP)**

The best explanation for this is if you have a parent class and a child class, then the base class and child class can be used interchangeably without getting incorrect results.

This might still be confusing, so let's take a look at the classic Square-Rectangle example.

Mathematically, a square is a rectangle, but if you model it using the "is-a" relationship via inheritance, you quickly get into trouble.

**Bad:**

```csharp
class Rectangle
{
    protected double Width = 0;
    protected double Height = 0;

    public Drawable Render(double area)
    {
        // ...
    }

    public void SetWidth(double width)
    {
        Width = width;
    }

    public void SetHeight(double height)
    {
        Height = height;
    }

    public double GetArea()
    {
        return Width * Height;
    }
}

class Square : Rectangle
{
    public double SetWidth(double width)
    {
        Width = Height = width;
    }

    public double SetHeight(double height)
    {
        Width = Height = height;
    }
}

Drawable RenderLargeRectangles(Rectangle rectangles)
{
    foreach (rectangle in rectangles)
    {
        rectangle.SetWidth(4);
        rectangle.SetHeight(5);
        var area = rectangle.GetArea(); // BAD: Will return 25 for Square. Should be 20.
        rectangle.Render(area);
    }
}

var rectangles = new[] { new Rectangle(), new Rectangle(), new Square() };
RenderLargeRectangles(rectangles);
```

**Good:**

```csharp
abstract class ShapeBase
{
    protected double Width = 0;
    protected double Height = 0;

    abstract public double GetArea();

    public Drawable Render(double area)
    {
        // ...
    }
}

class Rectangle : ShapeBase
{
    public void SetWidth(double width)
    {
        Width = width;
    }

    public void SetHeight(double height)
    {
        Height = height;
    }

    public double GetArea()
    {
        return Width * Height;
    }
}

class Square : ShapeBase
{
    private double Length = 0;

    public double SetLength(double length)
    {
        Length = length;
    }

    public double GetArea()
    {
        return Math.Pow(Length, 2);
    }
}

Drawable RenderLargeRectangles(Rectangle rectangles)
{
    foreach (rectangle in rectangles)
    {
        if (rectangle is Square)
        {
            rectangle.SetLength(5);
        }
        else if (rectangle is Rectangle)
        {
            rectangle.SetWidth(4);
            rectangle.SetHeight(5);
        }

        var area = rectangle.GetArea();
        rectangle.Render(area);
    }
}

var shapes = new[] { new Rectangle(), new Rectangle(), new Square() };
RenderLargeRectangles(shapes);
```

#### **Interface Segregation Principle (ISP)**

ISP states that "Clients should not be forced to depend upon interfaces that they do not use."

A good example to look at that demonstrates this principle is for classes that require large settings objects.

Not requiring clients to setup huge amounts of options is beneficial, because most of the time they won't need all of the settings. Making them optional helps prevent having a "fat interface".

**Bad:**

```csharp
public interface IEmployee
{
    void Work();
    void Eat();
}

public class Human : IEmployee
{
    public void Work()
    {
        // ....working
    }

    public void Eat()
    {
        // ...... eating in lunch break
    }
}

public class Robot : IEmployee
{
    public void Work()
    {
        //.... working much more
    }

    public void Eat()
    {
        //.... robot can't eat, but it must implement this method
    }
}
```

**Good:**

```csharp
public interface IWorkable
{
    void Work();
}

public interface IFeedable
{
    void Eat();
}

public interface IEmployee : IFeedable, IWorkable
{
}

public class Human : IEmployee
{
    public void Work()
    {
        // ....working
    }

    public void Eat()
    {
        //.... eating in lunch break
    }
}

// robot can only work
public class Robot : IWorkable
{
    public void Work()
    {
        // ....working
    }
}
```

#### **Dependency Inversion Principle (DIP)**

This principle states two essential things:

      * High-level modules should not depend on low-level modules. Both should depend on abstractions.
      * Abstractions should not depend upon details. Details should depend on abstractions.

This can be hard to understand at first, but if you've worked with .NET/.NET Core framework, you've seen an implementation of this principle in the form of Dependency Injection (DI).

**Bad:**

```csharp
public abstract class EmployeeBase
{
    protected virtual void Work()
    {
        // ....working
    }
}

public class Human : EmployeeBase
{
    public override void Work()
    {
        //.... working much more
    }
}

public class Robot : EmployeeBase
{
    public override void Work()
    {
        //.... working much, much more
    }
}

public class Manager
{
    private readonly Robot _robot;
    private readonly Human _human;

    public Manager(Robot robot, Human human)
    {
        _robot = robot;
        _human = human;
    }

    public void Manage()
    {
        _robot.Work();
        _human.Work();
    }
}
```

**Good:**

```csharp
public interface IEmployee
{
    void Work();
}

public class Human : IEmployee
{
    public void Work()
    {
        // ....working
    }
}

public class Robot : IEmployee
{
    public void Work()
    {
        //.... working much more
    }
}

public class Manager
{
    private readonly IEnumerable<IEmployee> _employees;

    public Manager(IEnumerable<IEmployee> employees)
    {
        _employees = employees;
    }

    public void Manage()
    {
        foreach (var employee in _employees)
        {
            _employee.Work();
        }
    }
}
```

#### **Don’t repeat yourself (DRY)**

Do your absolute best to avoid duplicate code. Duplicate code is bad because it means that there's more than one place to alter something if you need to change some logic.

Imagine if you run a restaurant and you keep track of your inventory: all your tomatoes, onions, garlic, spices, etc. If you have multiple lists that you keep this on, then all have to be updated when you serve a dish with tomatoes in them. If you only have one list, there's only one place to update!

Oftentimes you have duplicate code because you have two or more slightly different things, that share a lot in common, but their differences force you to have two or more separate functions that do much of the same things. Removing duplicate code means creating an abstraction that can handle this set of different things with just one function/module/class.

Getting the abstraction right is critical, that's why you should follow the SOLID principles laid out in the Classes section. Bad abstractions can be worse than duplicate code, so be careful! Having said this, if you can make a good abstraction, do it! Don't repeat yourself, otherwise you'll find yourself updating multiple places anytime you want to change one thing.

**Bad:**

```csharp
public List<EmployeeData> ShowDeveloperList(Developers developers)
{
    foreach (var developers in developer)
    {
        var expectedSalary = developer.CalculateExpectedSalary();
        var experience = developer.GetExperience();
        var githubLink = developer.GetGithubLink();
        var data = new[] {
            expectedSalary,
            experience,
            githubLink
        };

        Render(data);
    }
}

public List<ManagerData> ShowManagerList(Manager managers)
{
    foreach (var manager in managers)
    {
        var expectedSalary = manager.CalculateExpectedSalary();
        var experience = manager.GetExperience();
        var githubLink = manager.GetGithubLink();
        var data =
        new[] {
            expectedSalary,
            experience,
            githubLink
        };

        render(data);
    }
}
```

**Good:**

```csharp
public List<EmployeeData> ShowList(Employee employees)
{
    foreach (var employee in employees)
    {
        var expectedSalary = employees.CalculateExpectedSalary();
        var experience = employees.GetExperience();
        var githubLink = employees.GetGithubLink();
        var data =
        new[] {
            expectedSalary,
            experience,
            githubLink
        };

        render(data);
    }
}
```

### 3.8 Concurrency

#### **Use Async/Await (C#)**

The async/await is the best for IO bound tasks (networking communication, database communication, http request, etc.) but it is not good to apply on computational bound tasks (traverse on the huge list, render a hugge image, etc.). Because it will release the holding thread to the thread pool and CPU/cores available will not involve to process those tasks. Therefore, we should avoid using Async/Await for computional bound tasks.

For dealing with computational bound tasks, prefer to use Task.Factory.CreateNew with TaskCreationOptions is LongRunning. It will start a new background thread to process a heavy computational bound task without release it back to the thread pool until the task being completed.

> [Cheat Sheet](https://gist.github.com/jonlabelle/841146854b23b305b50fa5542f84b20c) and
> [Best practices](https://docs.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming)

- **Async/Await are even cleaner than Promises (ES6,Javascript)**

  Promises are a very clean alternative to callbacks, but ES2017/ES8 brings async and await which offer an even cleaner solution.

  All you need is a function that is prefixed in an async keyword, and then you can write your logic imperatively without a then chain of functions.

  Use this if you can take advantage of ES2017/ES8 features today!

  **Bad:**

  ```javascript
  import { get } from "request-promise";
  import { writeFile } from "fs-extra";

  get("https://en.wikipedia.org/wiki/Robert_Cecil_Martin")
    .then(body => {
      return writeFile("article.html", body);
    })
    .then(() => {
      console.log("File written");
    })
    .catch(err => {
      console.error(err);
    });
  ```

  **Good:**

  ```javascript
  import { get } from "request-promise";
  import { writeFile } from "fs-extra";

  async function getCleanCodeArticle() {
    try {
      const body = await get(
        "https://en.wikipedia.org/wiki/Robert_Cecil_Martin"
      );
      await writeFile("article.html", body);
      console.log("File written");
    } catch (err) {
      console.error(err);
    }
  }

  getCleanCodeArticle();
  ```

### 3.9 Error Handling

#### **DO** use multiple catch block instead of if conditions

If you need to take action according to type of the exception, you better use multiple catch block for exception handling.

**Bad:**

```csharp
try
{
    // Do something..
}
catch (Exception ex)
{

    if (ex is TaskCanceledException)
    {
        // Take action for TaskCanceledException
    }
    else if (ex is TaskSchedulerException)
    {
        // Take action for TaskSchedulerException
    }
}
```

**Good:**

```csharp
try
{
    // Do something..
}
catch (TaskCanceledException ex)
{
    // Take action for TaskCanceledException
}
catch (TaskSchedulerException ex)
{
    // Take action for TaskSchedulerException
}
```

#### **DO** keep exception stack trace when rethrowing exceptions

C# allows the exception to be rethrown in a catch block using the throw keyword. It is a bad practice to throw a caught exception using throw e;. This statement resets the stack trace.

Instead use throw;. This will keep the stack trace and provide a deeper insight about the exception. Another option is to use a custom exception.

Simply instantiate a new exception and set its inner exception property to the caught exception with throw new CustomException("some info", e);. Adding information to an exception is a good practice as it will help with debugging. However, if the objective is to log an exception then use throw; to pass the buck to the caller.

**Bad:**

```csharp
try
{
    FunctionThatMightThrow();
}
catch (Exception ex)
{
    logger.LogInfo(ex);
    throw ex;
}
```

**Good:**

```csharp
try
{
    FunctionThatMightThrow();
}
catch (Exception error)
{
    logger.LogInfo(error);
    throw;
}

// or
try
{
    FunctionThatMightThrow();
}
catch (Exception error)
{
    logger.LogInfo(error);
    throw new CustomException(error);
}
```

#### **DO NOT** use 'throw ex' in catch block

If you need to re-throw an exception after catching it, use just 'throw' By using this, you will save the stack trace. But in the bad option below, you will lost the stack trace.

**Bad:**

```csharp
try
{
    // Do something..
}
catch (Exception ex)
{
    // Any action something like roll-back or logging etc.
    throw ex;
}
```

**Good:**

```csharp
try
{
    // Do something..
}
catch (Exception ex)
{
    // Any action something like roll-back or logging etc.
    throw;
}
```

#### **DO NOT** ignore caught errors

Doing nothing with a caught error doesn't give you the ability to ever fix or react to said error.

Throwing the error isn't much better as often times it can get lost in a sea of things printed to the console.

If you wrap any bit of code in a try/catch it means you think an error may occur there and therefore you should have a plan, or create a code path, for when it occurs.

**Bad:**

```csharp
try
{
    FunctionThatMightThrow();
}
catch (Exception ex)
{
    // silent exception
}
```

**Good:**

```csharp
try
{
    FunctionThatMightThrow();
}
catch (Exception error)
{
    NotifyUserOfError(error);

    // Another option
    ReportErrorToService(error);
}
```

### 3.8 Comments

#### **DO** only comment things that have business logic complexity

Comments are an apology, not a requirement. Good code mostly documents itself.

**Bad:**

```csharp
public int HashIt(string data)
{
    // The hash
    var hash = 0;

    // Length of string
    var length = data.length;

    // Loop through every character in data
    for (var i = 0; i < length; i++)
    {
        // Get character code.
        const char = data.charCodeAt(i);
        // Make the hash
        hash = ((hash << 5) - hash) + char;
        // Convert to 32-bit integer
        hash &= hash;
    }
}
```

**Better but still Bad:**

```csharp
public int HashIt(string data)
{
    var hash = 0;
    var length = data.length;
    for (var i = 0; i < length; i++)
    {
        const char = data.charCodeAt(i);
        hash = ((hash << 5) - hash) + char;

        // Convert to 32-bit integer
        hash &= hash;
    }
}
```

**Good:**

```csharp
public int Hash(string data)
{
    var hash = 0;
    var length = data.length;

    for (var i = 0; i < length; i++)
    {
        var character = data[i];
        // use of djb2 hash algorithm as it has a good compromise
        // between speed and low collision with a very simple implementation
        hash = ((hash << 5) - hash) + character;

        hash = ConvertTo32BitInt(hash);
    }
    return hash;
}

private int ConvertTo32BitInt(int value)
{
    return value & value;
}
```

#### **DO NOT** leave commented out code in your codebase

Version control exists for a reason. Leave old code in your history.

**Bad:**

```csharp
doStuff();
// doOtherStuff();
// doSomeMoreStuff();
// doSoMuchStuff();
```

**Good:**

```csharp
doStuff();
```

#### **DO NOT** have journal comments

Remember, use version control! There's no need for dead code, commented code, and especially journal comments. Use git log to get history!

**Bad:**

```csharp
/**
* 2018-12-20: Removed monads, didn't understand them (RM)
* 2017-10-01: Improved using special monads (JP)
* 2016-02-03: Removed type-checking (LI)
* 2015-03-14: Added combine with type-checking (JR)
*/
public int Combine(int a,int b)
{
    return a + b;
}
```

**Good:**

```csharp
public int Combine(int a,int b)
{
    return a + b;
}
```

#### **Avoid** positional markers

They usually just add noise. Let the functions and variable names along with the proper indentation and formatting give the visual structure to your code.

**Bad:**

```csharp
////////////////////////////////////////////////////////////////////////////////
// Scope Model Instantiation
////////////////////////////////////////////////////////////////////////////////
var model = new[]
{
    menu: 'foo',
    nav: 'bar'
};

////////////////////////////////////////////////////////////////////////////////
// Action setup
////////////////////////////////////////////////////////////////////////////////
void Actions()
{
    // ...
};
```

**Good:**

```csharp
var model = new[]
{
    menu: 'foo',
    nav: 'bar'
};

void Actions()
{
    // ...
};
```

### 3.9 Performace Consideration

- **DO** use `sealed` for private classes if they are not to be inherited.
- **DO** add `readonly` to fields if they do not tend to be changed.
- **DO** use `static` methods if it is not instance relevant.
- **DO** use `RegexOptions.Compiled` for readonly Regex.

# 4. Cheetsheets

1. [Csharp coding standards](https://dotnet.github.io/docfx/guideline/csharp_coding_standards.html)
2. [Clean code typescript](https://github.com/labs42io/clean-code-typescript)
3. [Clean code dotnet](https://github.com/thangchung/clean-code-dotnet)
4. [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/) - Cheatsheet was created to provide a concise collection of high value information on specific application security topics
5. [Clean Code](https://github.com/thangchung/clean-code-dotnet/blob/master/cheetsheets/Clean-Code-V2.4.pdf) - The summary of Clean Code: A Handbook of Agile Software Craftsmanship book
6. [Clean Architecture](https://github.com/thangchung/clean-code-dotnet/blob/master/cheetsheets/Clean-Architecture-V1.0.pdf) - The summary of Clean Architecture: A Craftsman's Guide to Software Structure and Design book
7. [Modern JavaScript Cheatsheet](https://github.com/mbeaudru/modern-js-cheatsheet) - Cheatsheet for the JavaScript knowledge you will frequently encounter in modern projects

# 5. Tools

1. [StyleCop](https://github.com/DotNetAnalyzers/StyleCopAnalyzers)
2. [TSLint](https://github.com/Glavin001/tslint-clean-code)
