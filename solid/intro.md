# Advantages of SOLID Principles in C#
As a developer, while you are designing and developing any software applications, then you need to consider the following points.

- Maintainability: 
As we know, nowadays maintaining software is very important for organizations. Day by day the business may grow for the organization and you may need to enhance the software with new changes. So you need to design the software in such a way that it should accept future changes without any problem.

- Testability:
Test-Driven Development (TDD) is one of the most important key aspects nowadays when you need to design and develop a large-scale application. We need to design the application in such a way that we should test each individual functionalities. 

- Flexibility and Extensibility:
Nowadays flexibility and extensibility both are very much required for enterprise applications. So we should design the application in such a way that it should be flexible so that it can be adapted to work in different ways and extensible so that we can add new features easily with minimum modifications.

- Parallel Development:
The Parallel Development of an application is one of the most important key aspects. As we know it is not possible to have the entire development team will work on the same module at the same time. So we need to design the software in such a way that different teams can work on different modules.

# SOLID Acronym
S stands for the `Single Responsibility Principle` which is also known as SRP.
- So we need to design the software in such a way that everything in a class or module should be related to a single responsibility. That does not mean your class should contain only one method or property, you can have multiple members (methods or properties) as long as they are related to a single responsibility or functionality. So, with the help of SRP, the classes become smaller and cleaner and thus easier to maintain.


O stands for `the Open-Closed Principle` which is also known as OSP.
- The Open for extension means we need to design the software modules/classes in such a way that the new responsibilities or functionalities should be added easily when new requirements come. On the other hand, Closed for modification means, we should not modify the class/module until we find some bugs.
```csharp
class AreaCalculator
{
    // ...

    public function sum()
    {
        foreach ($this->shapes as $shape) {
            if (is_a($shape, 'ShapeInterface')) {
                $area[] = $shape->area();
                continue;
            }

            throw new AreaCalculatorInvalidShapeException();
        }

        return array_sum($area);
    }
}
```
```csharp
interface ShapeInterface
{
    public function area();
}
class Square implements ShapeInterface
{
    // ...
}
class Circle implements ShapeInterface
{
    // ...
}
```
L stands for the `Liskov Substitution` Principle which is also known as LSP.
-  This principle states that, if S is a subtype of T, then objects of type T should be replaced with the objects of type S. In other words, we can say that objects in an application should be replaceable with the instances of their subtypes without modifying the correctness of that application. 
```csharp
$areas = new AreaCalculator($shapes);
$volumes = new VolumeCalculator($solidShapes);

$output = new SumCalculatorOutputter($areas);
$output2 = new SumCalculatorOutputter($volumes);

class VolumeCalculator extends AreaCalculator
{
    public function construct($shapes = [])
    {
        parent::construct($shapes);
    }

    public function sum()
    {
        // logic to calculate the volumes and then return an array of output
        return [$summedData];
    }
}
```

I stand for the Interface `Segregation Principle` which is also known as ISP.
```csharp
namespace SOLID_PRINCIPLES.ISP
{
    public interface IPrinterTasks
    {
        void Print(string PrintContent);
        void Scan(string ScanContent);
        void Fax(string FaxContent);
        void PrintDuplex(string PrintDuplexContent);
    }
}

interface ShapeInterface
{
    public function area();
}

interface ThreeDimensionalShapeInterface
{
    public function volume();
}

class Cuboid implements ShapeInterface, ThreeDimensionalShapeInterface
{
    public function area()
    {
        // calculate the surface area of the cuboid
    }

    public function volume()
    {
        // calculate the volume of the cuboid
    }
}
```


D stands for `Dependency Inversion Principle` which is also known as DIP.
- The most important point that you need to remember while developing real-time applications, always to try to keep the High-level module and Low-level module as loosely coupled as possible.

When a class knows about the design and implementation of another class, it raises the risk that if we do any changes to one class will break the other class. So we must keep these high-level and low-level modules/classes loosely coupled as much as possible. To do that, we need to make both of them dependent on abstractions instead of knowing each other
```csharp
class MySQLConnection implements DBConnectionInterface
{
    public function connect()
    {
        // handle the database connection
        return 'Database connection';
    }
}

class PasswordReminder
{
    private $dbConnection;

    public function __construct(DBConnectionInterface $dbConnection)
    {
        $this->dbConnection = $dbConnection;
    }
}
```