Here is a complete, before-and-after code breakdown for all five **SOLID** principles and the bonus **DRY** principle using C# syntax (the logic applies universally to Java, TypeScript, or C++).

---

## 1. Single Responsibility Principle (SRP)

> **Definition:** A class should have only one reason to change.

### ❌ The Violation

The `Invoice` class handles its core data but also takes on the responsibility of sending emails and logging errors.

```csharp
public class Invoice
{
    public int InvoiceId { get; set; }
    public double Amount { get; set; }

    public void AddInvoice()
    {
        // Logic to add invoice
    }

    public void SendInvoiceEmail()
    {
        // Violation: Invoice shouldn't care about email protocols
        Console.WriteLine("Sending invoice via email...");
    }

    public void LogError(string error)
    {
        // Violation: Invoice shouldn't care about logging mechanisms
        Console.WriteLine($"Logging error: {error}");
    }
}

```

### The Solution

Split the extra responsibilities into their own dedicated classes.

```csharp
public class Invoice
{
    public int InvoiceId { get; set; }
    public double Amount { get; set; }

    public void AddInvoice() { /* Core Logic Only */ }
}

public class EmailService
{
    public void SendInvoiceEmail(Invoice invoice) { /* Email Logic */ }
}

public class LoggerService
{
    public void LogError(string error) { /* Logging Logic */ }
}

```

---

## 2. Open/Closed Principle (OCP)

> **Definition:** Open for extension, closed for modification.

### ❌ The Violation

If a new account type (like `Current`) is introduced tomorrow, we are forced to break into this class and modify it with a new `else if` block.

```csharp
public class BankAccount
{
    public string AccountType { get; set; }
    public double Balance { get; set; }

    public double CalculateInterest()
    {
        // Violation: Modifying this code block for every new account type
        if (AccountType == "Savings")
        {
            return Balance * 0.03;
        }
        else if (AccountType == "Fixed")
        {
            return Balance * 0.05;
        }
        return 0;
    }
}

```

### The Solution

Create an abstraction (interface) and extend it via new classes without modifying existing working code.

```csharp
public interface IAccount
{
    double CalculateInterest(double balance);
}

public class SavingsAccount : IAccount
{
    public double CalculateInterest(double balance) => balance * 0.03;
}

public class FixedAccount : IAccount
{
    public double CalculateInterest(double balance) => balance * 0.05;
}

// Extension: We easily add a Current Account without altering Savings or Fixed logic
public class CurrentAccount : IAccount
{
    public double CalculateInterest(double balance) => balance * 0.07;
}

```

---

## 3. Liskov Substitution Principle (LSP)

> **Definition:** Subclasses must be completely substitutable for their base classes without breaking the application.

### ❌ The Violation

The base class forces all employees to have a bonus. Because a contractual employee doesn't get a bonus, the developer overrides it to throw a runtime exception, breaking the app if swapped with the base class.

```csharp
public class Employee
{
    public virtual double CalculateSalary() => 100000;
    public virtual double CalculateBonus() => 10000;
}

public class PermanentEmployee : Employee
{
    public override double CalculateSalary() => 200000;
}

public class ContractualEmployee : Employee
{
    public override double CalculateSalary() => 150000;

    // Violation: This throws an unexpected error and crashes the app loop 
    public override double CalculateBonus()
    {
        throw new NotImplementedException("Contractual employees don't get bonuses!");
    }
}

```

### The Solution

Remove the bonus logic from the base class so that `Employee` only contains rules universally shared by *all* subtypes.

```csharp
public class Employee
{
    public virtual double CalculateSalary() => 100000;
}

// Separate interface specifically for positions that qualify for a bonus
public interface IBonusEligible
{
    double CalculateBonus();
}

public class PermanentEmployee : Employee, IBonusEligible
{
    public override double CalculateSalary() => 200000;
    public double CalculateBonus() => 10000; 
}

public class ContractualEmployee : Employee
{
    public override double CalculateSalary() => 150000;
    // No bonus method present, eliminating unexpected runtime crashes!
}

```

---

## 4. Interface Segregation Principle (ISP)

> **Definition:** Classes shouldn't be forced to depend on or implement interface methods they don't use.

### ❌ The Violation

A `Car` is forced to implement a dummy or empty `Fly()` method just to comply with the bloated interface.

```csharp
public interface IVehicle
{
    void Drive();
    void Fly();
}

public class Car : IVehicle
{
    public void Drive() => Console.WriteLine("Driving a car");
    
    // Violation: Car cannot fly!
    public void Fly() => throw new NotImplementedException(); 
}

```

### The Solution

Segregate the large interface into smaller, hyper-focused ones.

```csharp
public interface IDriveable
{
    void Drive();
}

public interface IFlyable
{
    void Fly();
}

// Car only uses what it needs
public class Car : IDriveable
{
    public void Drive() => Console.WriteLine("Driving a car");
}

// A futuristic flying car can implement both safely [00:10:27]
public class FlyingCar : IDriveable, IFlyable
{
    public void Drive() => Console.WriteLine("Driving on roads");
    public void Fly() => Console.WriteLine("Flying in the sky");
}

```

---

## 5. Dependency Inversion Principle (DIP)

> **Definition:** High-level modules should not depend on low-level modules. Both should depend on abstractions.

### ❌ The Violation

The high-level `DataAccessLayer` is tightly coupled directly to a concrete, low-level `FileLogger` class.

```csharp
public class FileLogger
{
    public void Log(string message) => Console.WriteLine($"Logged to file: {message}");
}

public class DataAccessLayer
{
    private FileLogger _logger = new FileLogger(); // Violation: Tight coupling

    public void AddCustomer()
    {
        // Database logic here
        _logger.Log("Customer added successfully.");
    }
}

```

### The Solution

Introduce an interface abstraction and inject it via the class constructor (Dependency Injection).

```csharp
public interface ILogger
{
    void Log(string message);
}

public class FileLogger : ILogger
{
    public void Log(string message) => Console.WriteLine($"Logged to file: {message}");
}

public class DataAccessLayer
{
    private readonly ILogger _logger;

    // High-level class now depends strictly on the abstraction (ILogger interface)
    public DataAccessLayer(ILogger logger)
    {
        _logger = logger;
    }

    public void AddCustomer()
    {
        _logger.Log("Customer added successfully.");
    }
}

```

---

## Bonus: DRY Principle (Don't Repeat Yourself)

> **Definition:** Avoid writing duplicate code or functionality multiple times.

### ❌ The Violation

The exact same tax validation calculation mathematical formula is repeated in completely separate parts of the app.

```csharp
public class OrderService
{
    public void ProcessOrder(double total)
    {
        // Duplicate Logic
        double tax = total * 0.18;
        Console.WriteLine($"Processing order tax: {tax}");
    }
}

public class QuoteService
{
    public void GenerateQuote(double total)
    {
        // Duplicate Logic
        double tax = total * 0.18;
        Console.WriteLine($"Estimated quote tax: {tax}");
    }
}

```

### The Solution

Extract the formula logic into a unified, reusable helper method or base structure.

```csharp
public static class TaxCalculator
{
    public static double CalculateStandardTax(double total) => total * 0.18;
}

public class OrderService
{
    public void ProcessOrder(double total)
    {
        double tax = TaxCalculator.CalculateStandardTax(total);
        Console.WriteLine($"Processing order tax: {tax}");
    }
}

public class QuoteService
{
    public void GenerateQuote(double total)
    {
        double tax = TaxCalculator.CalculateStandardTax(total);
        Console.WriteLine($"Estimated quote tax: {tax}");
    }
}

```