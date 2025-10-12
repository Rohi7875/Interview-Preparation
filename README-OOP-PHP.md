# PHP OOP Interview Questions & Answers
### For 7+ Years Experienced PHP Developers

---

## Table of Contents
1. [Basic OOP Concepts](#basic-oop-concepts)
2. [Classes and Objects](#classes-and-objects)
3. [Inheritance](#inheritance)
4. [Polymorphism](#polymorphism)
5. [Encapsulation](#encapsulation)
6. [Abstraction](#abstraction)
7. [Interfaces](#interfaces)
8. [Traits](#traits)
9. [Magic Methods](#magic-methods)
10. [Design Patterns](#design-patterns)

---

## Basic OOP Concepts

### Q1. What is Object-Oriented Programming?
**Answer:** OOP is a programming paradigm based on the concept of "objects" which contain data (properties) and code (methods). It helps organize code in a modular, reusable, and maintainable way.

**Real-time Example:**
```php
// Think of a Car as an object
class Car {
    public $brand;
    public $color;
    
    public function __construct($brand, $color) {
        $this->brand = $brand;
        $this->color = $color;
    }
    
    public function drive() {
        return "The {$this->color} {$this->brand} is driving!";
    }
}

$myCar = new Car("Toyota", "Red");
echo $myCar->drive(); // Output: The Red Toyota is driving!
```

### Q2. What are the four pillars of OOP?
**Answer:** 
1. **Encapsulation** - Hiding internal details and showing only functionality
2. **Abstraction** - Hiding complex implementation details
3. **Inheritance** - Creating new classes from existing ones
4. **Polymorphism** - One interface, multiple implementations

### Q3. What is the difference between a class and an object?
**Answer:** 
- **Class** is a blueprint/template for creating objects
- **Object** is an instance of a class with actual values

**Real-time Example:**
```php
// Class is like a blueprint of a house
class House {
    public $rooms;
    public $color;
    
    public function __construct($rooms, $color) {
        $this->rooms = $rooms;
        $this->color = $color;
    }
}

// Objects are actual houses built from that blueprint
$house1 = new House(3, "White");  // First house object
$house2 = new House(5, "Blue");   // Second house object
```

### Q4. What is a constructor in PHP?
**Answer:** A constructor is a special method that is automatically called when an object is created. It's used to initialize object properties.

**Real-time Example:**
```php
class DatabaseConnection {
    private $host;
    private $username;
    private $password;
    private $connection;
    
    public function __construct($host, $username, $password) {
        $this->host = $host;
        $this->username = $username;
        $this->password = $password;
        $this->connect();
    }
    
    private function connect() {
        $this->connection = new PDO(
            "mysql:host={$this->host}", 
            $this->username, 
            $this->password
        );
        echo "Database connected successfully!";
    }
}

$db = new DatabaseConnection("localhost", "root", "password");
```

### Q5. What is a destructor in PHP?
**Answer:** A destructor is called when an object is destroyed or the script ends. It's used for cleanup operations like closing database connections or file handlers.

**Real-time Example:**
```php
class FileLogger {
    private $fileHandle;
    
    public function __construct($filename) {
        $this->fileHandle = fopen($filename, 'a');
        echo "Log file opened\n";
    }
    
    public function log($message) {
        fwrite($this->fileHandle, date('Y-m-d H:i:s') . " - $message\n");
    }
    
    public function __destruct() {
        if ($this->fileHandle) {
            fclose($this->fileHandle);
            echo "Log file closed\n";
        }
    }
}

$logger = new FileLogger("app.log");
$logger->log("Application started");
// Destructor automatically called at script end
```

---

## Classes and Objects

### Q6. What are access modifiers in PHP?
**Answer:** Access modifiers control the visibility of properties and methods:
- **public** - Accessible from anywhere
- **private** - Only accessible within the class
- **protected** - Accessible within the class and by inherited classes

**Real-time Example:**
```php
class BankAccount {
    public $accountHolder;      // Can be accessed anywhere
    private $balance;           // Only within this class
    protected $accountNumber;   // This class and child classes
    
    public function __construct($holder, $number, $initialBalance) {
        $this->accountHolder = $holder;
        $this->accountNumber = $number;
        $this->balance = $initialBalance;
    }
    
    public function deposit($amount) {
        $this->balance += $amount;
        return "Deposited: $amount. New balance: {$this->balance}";
    }
    
    public function getBalance() {
        return $this->balance;  // Public method to access private property
    }
}

$account = new BankAccount("John Doe", "ACC123", 1000);
echo $account->accountHolder;  // Works - public
// echo $account->balance;     // Error - private
echo $account->getBalance();   // Works - accessing via public method
```

### Q7. What is the difference between $this and self in PHP?
**Answer:**
- **$this** refers to the current object instance (used for non-static members)
- **self** refers to the current class (used for static members)

**Real-time Example:**
```php
class Counter {
    private $instanceCount = 0;      // Instance property
    private static $totalCount = 0;  // Static property
    
    public function increment() {
        $this->instanceCount++;      // Access instance property
        self::$totalCount++;         // Access static property
    }
    
    public function getInstanceCount() {
        return $this->instanceCount;
    }
    
    public static function getTotalCount() {
        return self::$totalCount;    // Can't use $this in static method
    }
}

$counter1 = new Counter();
$counter1->increment();
$counter1->increment();

$counter2 = new Counter();
$counter2->increment();

echo $counter1->getInstanceCount(); // 2
echo $counter2->getInstanceCount(); // 1
echo Counter::getTotalCount();      // 3 (total across all instances)
```

### Q8. What are static properties and methods?
**Answer:** Static members belong to the class itself rather than to any object instance. They can be accessed without creating an object.

**Real-time Example:**
```php
class Configuration {
    private static $config = [];
    
    public static function set($key, $value) {
        self::$config[$key] = $value;
    }
    
    public static function get($key) {
        return self::$config[$key] ?? null;
    }
    
    public static function all() {
        return self::$config;
    }
}

// No need to create object
Configuration::set('app_name', 'My Application');
Configuration::set('version', '1.0.0');
Configuration::set('environment', 'production');

echo Configuration::get('app_name');  // My Application
print_r(Configuration::all());
```

### Q9. What is method chaining in PHP?
**Answer:** Method chaining allows calling multiple methods on the same object in a single line by returning $this from each method.

**Real-time Example:**
```php
class QueryBuilder {
    private $query = "";
    
    public function select($columns) {
        $this->query = "SELECT $columns ";
        return $this;  // Return object for chaining
    }
    
    public function from($table) {
        $this->query .= "FROM $table ";
        return $this;
    }
    
    public function where($condition) {
        $this->query .= "WHERE $condition ";
        return $this;
    }
    
    public function orderBy($column) {
        $this->query .= "ORDER BY $column ";
        return $this;
    }
    
    public function getQuery() {
        return $this->query;
    }
}

$query = new QueryBuilder();
$sql = $query->select('*')
             ->from('users')
             ->where('active = 1')
             ->orderBy('created_at DESC')
             ->getQuery();

echo $sql;
// Output: SELECT * FROM users WHERE active = 1 ORDER BY created_at DESC
```

### Q10. What is the difference between include, require, include_once, and require_once?
**Answer:**
- **include** - Includes file, continues on error (warning)
- **require** - Includes file, stops on error (fatal error)
- **include_once** - Includes file only once, continues on error
- **require_once** - Includes file only once, stops on error

**Real-time Example:**
```php
// config.php
<?php
define('DB_HOST', 'localhost');
define('DB_USER', 'root');

// database.php
class Database {
    // Database connection logic
}

// app.php
require_once 'config.php';      // Must have, include only once
require_once 'database.php';    // Must have, include only once
include_once 'helpers.php';     // Optional, include only once

// Even if required multiple times, loaded only once
require_once 'config.php';  // Won't reload
```

---

## Inheritance

### Q11. What is inheritance in PHP?
**Answer:** Inheritance allows a class to inherit properties and methods from another class. The parent class is called the base/super class, and the child class is called the derived/sub class.

**Real-time Example:**
```php
// Parent class
class Employee {
    protected $name;
    protected $salary;
    
    public function __construct($name, $salary) {
        $this->name = $name;
        $this->salary = $salary;
    }
    
    public function getDetails() {
        return "Name: {$this->name}, Salary: {$this->salary}";
    }
    
    public function work() {
        return "{$this->name} is working...";
    }
}

// Child class inheriting from Employee
class Developer extends Employee {
    private $programmingLanguage;
    
    public function __construct($name, $salary, $language) {
        parent::__construct($name, $salary);  // Call parent constructor
        $this->programmingLanguage = $language;
    }
    
    public function code() {
        return "{$this->name} is coding in {$this->programmingLanguage}";
    }
    
    // Override parent method
    public function work() {
        return parent::work() . " and writing code!";
    }
}

$dev = new Developer("John", 75000, "PHP");
echo $dev->getDetails();  // Inherited method
echo $dev->code();        // Own method
echo $dev->work();        // Overridden method
```

### Q12. What is method overriding?
**Answer:** Method overriding occurs when a child class provides a specific implementation of a method that is already defined in its parent class.

**Real-time Example:**
```php
class PaymentGateway {
    public function processPayment($amount) {
        return "Processing payment of $$amount";
    }
    
    public function refund($amount) {
        return "Refunding $$amount";
    }
}

class PayPalGateway extends PaymentGateway {
    public function processPayment($amount) {
        // Override with PayPal specific implementation
        return "Processing PayPal payment of $$amount with fee calculation";
    }
}

class StripeGateway extends PaymentGateway {
    public function processPayment($amount) {
        // Override with Stripe specific implementation
        return "Processing Stripe payment of $$amount with token";
    }
}

$paypal = new PayPalGateway();
$stripe = new StripeGateway();

echo $paypal->processPayment(100);  // PayPal implementation
echo $stripe->processPayment(100);  // Stripe implementation
```

### Q13. What is the final keyword in PHP?
**Answer:** The final keyword prevents a class from being inherited or a method from being overridden.

**Real-time Example:**
```php
// Final class - cannot be extended
final class SecurityManager {
    public function encrypt($data) {
        return base64_encode($data);
    }
}

// This would cause error:
// class CustomSecurity extends SecurityManager {} // Error!

class User {
    // Final method - cannot be overridden
    final public function getId() {
        return $this->id;
    }
    
    public function getName() {
        return $this->name;
    }
}

class Admin extends User {
    // This would cause error:
    // public function getId() {} // Error!
    
    // This is allowed
    public function getName() {
        return "Admin: " . parent::getName();
    }
}
```

### Q14. What is the parent keyword?
**Answer:** The parent keyword is used to access methods or properties of the parent class from the child class.

**Real-time Example:**
```php
class Vehicle {
    protected $speed = 0;
    
    public function accelerate($increase) {
        $this->speed += $increase;
        return "Speed increased to {$this->speed} km/h";
    }
}

class SportsCar extends Vehicle {
    public function accelerate($increase) {
        // Call parent method first
        parent::accelerate($increase);
        
        // Add turbo boost
        $turboBoost = $increase * 0.5;
        $this->speed += $turboBoost;
        
        return "Sports car speed with turbo: {$this->speed} km/h";
    }
    
    public function getNormalSpeed($increase) {
        // Use parent's original method
        return parent::accelerate($increase);
    }
}

$car = new SportsCar();
echo $car->accelerate(50);      // With turbo boost
echo $car->getNormalSpeed(20);  // Without turbo boost
```

### Q15. What is multiple inheritance and does PHP support it?
**Answer:** Multiple inheritance means a class inheriting from multiple parent classes. PHP does not support multiple inheritance for classes, but it can be achieved using Interfaces and Traits.

**Real-time Example:**
```php
// PHP doesn't allow this:
// class Child extends Parent1, Parent2 {} // Error!

// Solution 1: Using Interfaces
interface Flyable {
    public function fly();
}

interface Swimmable {
    public function swim();
}

class Duck implements Flyable, Swimmable {
    public function fly() {
        return "Duck is flying";
    }
    
    public function swim() {
        return "Duck is swimming";
    }
}

// Solution 2: Using Traits
trait CanFly {
    public function fly() {
        return "Flying...";
    }
}

trait CanSwim {
    public function swim() {
        return "Swimming...";
    }
}

class SuperHero {
    use CanFly, CanSwim;
    
    public function showAbilities() {
        return $this->fly() . " and " . $this->swim();
    }
}

$hero = new SuperHero();
echo $hero->showAbilities();
```

---

## Polymorphism

### Q16. What is polymorphism in PHP?
**Answer:** Polymorphism means "many forms". It allows objects of different classes to be treated as objects of a common parent class. The same method name can have different implementations.

**Real-time Example:**
```php
abstract class Shape {
    abstract public function calculateArea();
    abstract public function calculatePerimeter();
}

class Circle extends Shape {
    private $radius;
    
    public function __construct($radius) {
        $this->radius = $radius;
    }
    
    public function calculateArea() {
        return pi() * $this->radius * $this->radius;
    }
    
    public function calculatePerimeter() {
        return 2 * pi() * $this->radius;
    }
}

class Rectangle extends Shape {
    private $width;
    private $height;
    
    public function __construct($width, $height) {
        $this->width = $width;
        $this->height = $height;
    }
    
    public function calculateArea() {
        return $this->width * $this->height;
    }
    
    public function calculatePerimeter() {
        return 2 * ($this->width + $this->height);
    }
}

class Triangle extends Shape {
    private $base;
    private $height;
    private $side1, $side2, $side3;
    
    public function __construct($base, $height, $side1, $side2, $side3) {
        $this->base = $base;
        $this->height = $height;
        $this->side1 = $side1;
        $this->side2 = $side2;
        $this->side3 = $side3;
    }
    
    public function calculateArea() {
        return 0.5 * $this->base * $this->height;
    }
    
    public function calculatePerimeter() {
        return $this->side1 + $this->side2 + $this->side3;
    }
}

// Polymorphism in action
function printShapeInfo(Shape $shape) {
    echo "Area: " . $shape->calculateArea() . "\n";
    echo "Perimeter: " . $shape->calculatePerimeter() . "\n";
}

$circle = new Circle(5);
$rectangle = new Rectangle(4, 6);
$triangle = new Triangle(3, 4, 3, 4, 5);

printShapeInfo($circle);     // Works with Circle
printShapeInfo($rectangle);  // Works with Rectangle
printShapeInfo($triangle);   // Works with Triangle
```

### Q17. What is the difference between compile-time and runtime polymorphism?
**Answer:**
- **Compile-time (Static) Polymorphism** - Method overloading (not directly supported in PHP)
- **Runtime (Dynamic) Polymorphism** - Method overriding (supported in PHP)

**Real-time Example:**
```php
// Runtime Polymorphism (Method Overriding)
class Notification {
    public function send($message) {
        return "Sending notification: $message";
    }
}

class EmailNotification extends Notification {
    public function send($message) {
        return "Sending email: $message to inbox";
    }
}

class SMSNotification extends Notification {
    public function send($message) {
        return "Sending SMS: $message to phone";
    }
}

class PushNotification extends Notification {
    public function send($message) {
        return "Sending push notification: $message to device";
    }
}

function sendNotification(Notification $notifier, $message) {
    // Runtime decision based on actual object type
    echo $notifier->send($message);
}

$email = new EmailNotification();
$sms = new SMSNotification();
$push = new PushNotification();

sendNotification($email, "Welcome!");  // Email implementation
sendNotification($sms, "Welcome!");    // SMS implementation
sendNotification($push, "Welcome!");   // Push implementation
```

### Q18. What is method overloading in PHP?
**Answer:** PHP doesn't support traditional method overloading like Java/C++. However, we can simulate it using magic methods `__call()` and `__callStatic()` or by using variable arguments.

**Real-time Example:**
```php
class Calculator {
    // Using variable arguments (PHP 5.6+)
    public function add(...$numbers) {
        return array_sum($numbers);
    }
    
    // Simulating method overloading with __call
    public function __call($method, $arguments) {
        if ($method === 'multiply') {
            $count = count($arguments);
            
            if ($count === 2) {
                return $arguments[0] * $arguments[1];
            } elseif ($count === 3) {
                return $arguments[0] * $arguments[1] * $arguments[2];
            }
        }
        
        throw new Exception("Method $method not found");
    }
}

$calc = new Calculator();

// add() works with different number of parameters
echo $calc->add(5, 10);           // 15
echo $calc->add(5, 10, 15);       // 30
echo $calc->add(5, 10, 15, 20);   // 50

// multiply() simulated overloading
echo $calc->multiply(5, 10);      // 50
echo $calc->multiply(5, 10, 2);   // 100
```

---

## Encapsulation

### Q19. What is encapsulation in PHP?
**Answer:** Encapsulation is the bundling of data (properties) and methods that operate on that data within a single unit (class), and restricting direct access to some of the object's components.

**Real-time Example:**
```php
class UserAccount {
    private $email;
    private $password;
    private $loginAttempts = 0;
    private $isLocked = false;
    
    public function __construct($email, $password) {
        $this->email = $email;
        $this->setPassword($password);
    }
    
    // Encapsulated password setting with validation
    public function setPassword($password) {
        if (strlen($password) < 8) {
            throw new Exception("Password must be at least 8 characters");
        }
        $this->password = password_hash($password, PASSWORD_BCRYPT);
    }
    
    // Encapsulated login logic
    public function login($email, $password) {
        if ($this->isLocked) {
            return "Account is locked. Contact support.";
        }
        
        if ($this->email === $email && password_verify($password, $this->password)) {
            $this->loginAttempts = 0;
            return "Login successful";
        }
        
        $this->loginAttempts++;
        
        if ($this->loginAttempts >= 3) {
            $this->isLocked = true;
            return "Too many failed attempts. Account locked.";
        }
        
        return "Invalid credentials. Attempts: {$this->loginAttempts}/3";
    }
    
    // Controlled access to email
    public function getEmail() {
        return $this->email;
    }
    
    // Can't directly access or modify password, attempts, or lock status
}

$user = new UserAccount("user@example.com", "securepass123");

// Can't do: $user->password = "123"; // Error - private property
// Can't do: $user->loginAttempts = 0; // Error - private property

echo $user->login("user@example.com", "wrongpass");  // Controlled access
echo $user->login("user@example.com", "wrongpass");
echo $user->login("user@example.com", "wrongpass");
echo $user->login("user@example.com", "securepass123"); // Account locked
```

### Q20. What are getters and setters?
**Answer:** Getters and setters are methods used to access and modify private properties, providing controlled access to encapsulated data.

**Real-time Example:**
```php
class Product {
    private $name;
    private $price;
    private $stock;
    private $discount = 0;
    
    public function __construct($name, $price, $stock) {
        $this->setName($name);
        $this->setPrice($price);
        $this->setStock($stock);
    }
    
    // Getter for name
    public function getName() {
        return $this->name;
    }
    
    // Setter for name with validation
    public function setName($name) {
        if (empty($name)) {
            throw new Exception("Product name cannot be empty");
        }
        $this->name = trim($name);
    }
    
    // Getter for price
    public function getPrice() {
        return $this->price;
    }
    
    // Setter for price with validation
    public function setPrice($price) {
        if ($price < 0) {
            throw new Exception("Price cannot be negative");
        }
        $this->price = $price;
    }
    
    // Getter for stock
    public function getStock() {
        return $this->stock;
    }
    
    // Setter for stock with validation
    public function setStock($stock) {
        if ($stock < 0) {
            throw new Exception("Stock cannot be negative");
        }
        $this->stock = $stock;
    }
    
    // Setter for discount with validation
    public function setDiscount($percentage) {
        if ($percentage < 0 || $percentage > 100) {
            throw new Exception("Discount must be between 0 and 100");
        }
        $this->discount = $percentage;
    }
    
    // Calculated getter
    public function getFinalPrice() {
        return $this->price - ($this->price * $this->discount / 100);
    }
    
    // Check availability
    public function isAvailable() {
        return $this->stock > 0;
    }
}

$product = new Product("Laptop", 1000, 5);
$product->setDiscount(10);

echo "Product: " . $product->getName();           // Getter
echo "Original Price: $" . $product->getPrice();  // Getter
echo "Final Price: $" . $product->getFinalPrice(); // Calculated getter
echo "In Stock: " . ($product->isAvailable() ? "Yes" : "No");
```

---

## Abstraction

### Q21. What is abstraction in PHP?
**Answer:** Abstraction is hiding complex implementation details and showing only essential features. It's achieved using abstract classes and interfaces.

**Real-time Example:**
```php
// Abstract class - cannot be instantiated
abstract class DatabaseDriver {
    protected $connection;
    protected $host;
    protected $username;
    protected $password;
    protected $database;
    
    public function __construct($host, $username, $password, $database) {
        $this->host = $host;
        $this->username = $username;
        $this->password = $password;
        $this->database = $database;
    }
    
    // Abstract methods - must be implemented by child classes
    abstract public function connect();
    abstract public function disconnect();
    abstract public function query($sql);
    abstract public function fetch($result);
    
    // Concrete method - shared implementation
    public function execute($sql) {
        if (!$this->connection) {
            $this->connect();
        }
        return $this->query($sql);
    }
}

class MySQLDriver extends DatabaseDriver {
    public function connect() {
        $this->connection = new PDO(
            "mysql:host={$this->host};dbname={$this->database}",
            $this->username,
            $this->password
        );
        return "MySQL Connected";
    }
    
    public function disconnect() {
        $this->connection = null;
        return "MySQL Disconnected";
    }
    
    public function query($sql) {
        return $this->connection->query($sql);
    }
    
    public function fetch($result) {
        return $result->fetchAll(PDO::FETCH_ASSOC);
    }
}

class PostgreSQLDriver extends DatabaseDriver {
    public function connect() {
        $this->connection = new PDO(
            "pgsql:host={$this->host};dbname={$this->database}",
            $this->username,
            $this->password
        );
        return "PostgreSQL Connected";
    }
    
    public function disconnect() {
        $this->connection = null;
        return "PostgreSQL Disconnected";
    }
    
    public function query($sql) {
        return $this->connection->query($sql);
    }
    
    public function fetch($result) {
        return $result->fetchAll(PDO::FETCH_ASSOC);
    }
}

// Usage - User doesn't need to know the complexity
function getUsers(DatabaseDriver $db) {
    $result = $db->execute("SELECT * FROM users");
    return $db->fetch($result);
}

$mysql = new MySQLDriver("localhost", "root", "pass", "mydb");
$pgsql = new PostgreSQLDriver("localhost", "postgres", "pass", "mydb");

$users = getUsers($mysql);  // Works with MySQL
$users = getUsers($pgsql);  // Works with PostgreSQL
```

### Q22. What is the difference between abstract class and interface?
**Answer:**

| Abstract Class | Interface |
|---------------|-----------|
| Can have both abstract and concrete methods | All methods are abstract (before PHP 8) |
| Can have properties | Cannot have properties (only constants) |
| Supports all access modifiers | All methods are public |
| Single inheritance only | Multiple interfaces can be implemented |
| Can have constructor | Cannot have constructor |
| Use when classes share common code | Use when classes share only method signatures |

**Real-time Example:**
```php
// Abstract Class - when classes share common implementation
abstract class PaymentProcessor {
    protected $amount;
    protected $currency = 'USD';
    
    public function __construct($amount) {
        $this->amount = $amount;
    }
    
    // Concrete method - shared by all payment processors
    public function validateAmount() {
        if ($this->amount <= 0) {
            throw new Exception("Amount must be greater than 0");
        }
        return true;
    }
    
    // Abstract method - must be implemented by each processor
    abstract public function processPayment();
    abstract public function getTransactionId();
    
    // Concrete method with logic
    public function formatAmount() {
        return $this->currency . ' ' . number_format($this->amount, 2);
    }
}

// Interface - when multiple classes share same contract but different implementation
interface Refundable {
    public function refund($amount);
    public function getRefundStatus();
}

interface Recurring {
    public function setupRecurring($interval);
    public function cancelRecurring();
}

// Class using abstract class and multiple interfaces
class StripePayment extends PaymentProcessor implements Refundable, Recurring {
    private $transactionId;
    private $recurringId;
    
    public function processPayment() {
        $this->validateAmount();
        $this->transactionId = 'stripe_' . uniqid();
        return "Processed {$this->formatAmount()} via Stripe";
    }
    
    public function getTransactionId() {
        return $this->transactionId;
    }
    
    public function refund($amount) {
        return "Refunded $$amount via Stripe";
    }
    
    public function getRefundStatus() {
        return "Refund status: Pending";
    }
    
    public function setupRecurring($interval) {
        $this->recurringId = 'recurring_' . uniqid();
        return "Setup recurring payment every $interval";
    }
    
    public function cancelRecurring() {
        return "Cancelled recurring payment {$this->recurringId}";
    }
}

$payment = new StripePayment(100);
echo $payment->processPayment();
echo $payment->refund(50);
echo $payment->setupRecurring('monthly');
```

### Q23. Can we instantiate an abstract class?
**Answer:** No, we cannot directly instantiate an abstract class. We must create a child class that implements all abstract methods.

**Real-time Example:**
```php
abstract class Report {
    protected $data;
    
    public function __construct($data) {
        $this->data = $data;
    }
    
    abstract public function generate();
    
    public function save($filename) {
        $content = $this->generate();
        file_put_contents($filename, $content);
        return "Report saved to $filename";
    }
}

// This will cause error:
// $report = new Report($data); // Error: Cannot instantiate abstract class

// Correct way:
class PDFReport extends Report {
    public function generate() {
        return "PDF Report: " . json_encode($this->data);
    }
}

class ExcelReport extends Report {
    public function generate() {
        return "Excel Report: " . implode(',', $this->data);
    }
}

$data = ['Sales' => 1000, 'Revenue' => 50000];

$pdf = new PDFReport($data);    // Works
echo $pdf->generate();
$pdf->save('report.pdf');

$excel = new ExcelReport($data); // Works
echo $excel->generate();
$excel->save('report.xlsx');
```

---

## Interfaces

### Q24. What is an interface in PHP?
**Answer:** An interface is a contract that defines method signatures without implementation. Classes that implement an interface must provide implementation for all its methods.

**Real-time Example:**
```php
interface CacheInterface {
    public function set($key, $value, $ttl = 3600);
    public function get($key);
    public function delete($key);
    public function clear();
    public function has($key);
}

class RedisCache implements CacheInterface {
    private $redis;
    
    public function __construct() {
        $this->redis = new Redis();
        $this->redis->connect('127.0.0.1', 6379);
    }
    
    public function set($key, $value, $ttl = 3600) {
        return $this->redis->setex($key, $ttl, serialize($value));
    }
    
    public function get($key) {
        $value = $this->redis->get($key);
        return $value ? unserialize($value) : null;
    }
    
    public function delete($key) {
        return $this->redis->del($key);
    }
    
    public function clear() {
        return $this->redis->flushAll();
    }
    
    public function has($key) {
        return $this->redis->exists($key);
    }
}

class FileCache implements CacheInterface {
    private $cacheDir;
    
    public function __construct($dir = './cache') {
        $this->cacheDir = $dir;
        if (!is_dir($dir)) {
            mkdir($dir, 0755, true);
        }
    }
    
    public function set($key, $value, $ttl = 3600) {
        $data = [
            'value' => $value,
            'expires' => time() + $ttl
        ];
        return file_put_contents(
            $this->getFilePath($key),
            serialize($data)
        );
    }
    
    public function get($key) {
        $file = $this->getFilePath($key);
        if (!file_exists($file)) {
            return null;
        }
        
        $data = unserialize(file_get_contents($file));
        
        if ($data['expires'] < time()) {
            $this->delete($key);
            return null;
        }
        
        return $data['value'];
    }
    
    public function delete($key) {
        $file = $this->getFilePath($key);
        return file_exists($file) ? unlink($file) : false;
    }
    
    public function clear() {
        $files = glob($this->cacheDir . '/*');
        foreach ($files as $file) {
            unlink($file);
        }
        return true;
    }
    
    public function has($key) {
        return file_exists($this->getFilePath($key));
    }
    
    private function getFilePath($key) {
        return $this->cacheDir . '/' . md5($key) . '.cache';
    }
}

// Usage - easily switchable
function cacheUserData(CacheInterface $cache, $userId, $userData) {
    $cache->set("user_$userId", $userData, 3600);
}

$redisCache = new RedisCache();
$fileCache = new FileCache();

cacheUserData($redisCache, 123, ['name' => 'John']);  // Redis
cacheUserData($fileCache, 123, ['name' => 'John']);   // File
```

### Q25. Can an interface extend another interface?
**Answer:** Yes, an interface can extend one or more interfaces using the `extends` keyword.

**Real-time Example:**
```php
interface Readable {
    public function read($id);
}

interface Writable {
    public function create($data);
    public function update($id, $data);
}

interface Deletable {
    public function delete($id);
}

// Interface extending multiple interfaces
interface CrudInterface extends Readable, Writable, Deletable {
    public function list($limit, $offset);
}

class UserRepository implements CrudInterface {
    private $pdo;
    
    public function __construct(PDO $pdo) {
        $this->pdo = $pdo;
    }
    
    public function read($id) {
        $stmt = $this->pdo->prepare("SELECT * FROM users WHERE id = ?");
        $stmt->execute([$id]);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }
    
    public function create($data) {
        $stmt = $this->pdo->prepare(
            "INSERT INTO users (name, email) VALUES (?, ?)"
        );
        $stmt->execute([$data['name'], $data['email']]);
        return $this->pdo->lastInsertId();
    }
    
    public function update($id, $data) {
        $stmt = $this->pdo->prepare(
            "UPDATE users SET name = ?, email = ? WHERE id = ?"
        );
        return $stmt->execute([$data['name'], $data['email'], $id]);
    }
    
    public function delete($id) {
        $stmt = $this->pdo->prepare("DELETE FROM users WHERE id = ?");
        return $stmt->execute([$id]);
    }
    
    public function list($limit, $offset) {
        $stmt = $this->pdo->prepare(
            "SELECT * FROM users LIMIT ? OFFSET ?"
        );
        $stmt->execute([$limit, $offset]);
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
}

// Usage
$pdo = new PDO("mysql:host=localhost;dbname=test", "root", "");
$userRepo = new UserRepository($pdo);

$userId = $userRepo->create(['name' => 'John', 'email' => 'john@example.com']);
$user = $userRepo->read($userId);
$userRepo->update($userId, ['name' => 'John Doe', 'email' => 'john@example.com']);
$users = $userRepo->list(10, 0);
$userRepo->delete($userId);
```

---

## Traits

### Q26. What are Traits in PHP?
**Answer:** Traits are a mechanism for code reuse in PHP's single inheritance language. A trait is similar to a class but is intended to group functionality in a fine-grained and consistent way.

**Real-time Example:**
```php
trait Timestampable {
    private $createdAt;
    private $updatedAt;
    
    public function setCreatedAt() {
        $this->createdAt = date('Y-m-d H:i:s');
    }
    
    public function setUpdatedAt() {
        $this->updatedAt = date('Y-m-d H:i:s');
    }
    
    public function getCreatedAt() {
        return $this->createdAt;
    }
    
    public function getUpdatedAt() {
        return $this->updatedAt;
    }
}

trait SoftDeletable {
    private $deletedAt = null;
    
    public function softDelete() {
        $this->deletedAt = date('Y-m-d H:i:s');
    }
    
    public function restore() {
        $this->deletedAt = null;
    }
    
    public function isDeleted() {
        return $this->deletedAt !== null;
    }
}

trait Sluggable {
    private $slug;
    
    public function generateSlug($text) {
        $this->slug = strtolower(trim(preg_replace('/[^A-Za-z0-9-]+/', '-', $text), '-'));
    }
    
    public function getSlug() {
        return $this->slug;
    }
}

// Using multiple traits
class BlogPost {
    use Timestampable, SoftDeletable, Sluggable;
    
    private $title;
    private $content;
    
    public function __construct($title, $content) {
        $this->title = $title;
        $this->content = $content;
        $this->setCreatedAt();
        $this->generateSlug($title);
    }
    
    public function update($title, $content) {
        $this->title = $title;
        $this->content = $content;
        $this->setUpdatedAt();
        $this->generateSlug($title);
    }
    
    public function getTitle() {
        return $this->title;
    }
}

$post = new BlogPost("My First Blog Post", "This is the content");

echo "Created: " . $post->getCreatedAt();
echo "Slug: " . $post->getSlug(); // my-first-blog-post

$post->update("My Updated Blog Post", "Updated content");
echo "Updated: " . $post->getUpdatedAt();

$post->softDelete();
echo "Deleted: " . ($post->isDeleted() ? "Yes" : "No");

$post->restore();
echo "Deleted after restore: " . ($post->isDeleted() ? "Yes" : "No");
```

### Q27. How to resolve trait method conflicts?
**Answer:** When multiple traits have methods with the same name, we use `insteadof` to choose one and `as` to alias another.

**Real-time Example:**
```php
trait Logger {
    public function log($message) {
        echo "[LOG] $message\n";
    }
    
    public function debug($message) {
        echo "[DEBUG] $message\n";
    }
}

trait ErrorHandler {
    public function log($message) {
        echo "[ERROR] $message\n";
    }
    
    public function handle($exception) {
        $this->log($exception->getMessage());
    }
}

class Application {
    use Logger, ErrorHandler {
        // Resolve conflict: use Logger's log method
        Logger::log insteadof ErrorHandler;
        
        // Create alias for ErrorHandler's log method
        ErrorHandler::log as logError;
        
        // Change visibility and create alias
        Logger::debug as private debugLog;
    }
    
    public function run() {
        $this->log("Application started");      // Uses Logger::log
        $this->logError("Something went wrong"); // Uses ErrorHandler::log (aliased)
        $this->debugLog("Debug info");          // Uses Logger::debug (aliased, private)
    }
}

$app = new Application();
$app->run();

// Output:
// [LOG] Application started
// [ERROR] Something went wrong
// [DEBUG] Debug info
```

### Q28. Can traits have properties?
**Answer:** Yes, traits can have properties, but if a class using the trait defines a property with the same name, it will cause a fatal error unless they have the same visibility and initial value.

**Real-time Example:**
```php
trait HasUUID {
    private $uuid;
    
    public function generateUUID() {
        $this->uuid = sprintf('%04x%04x-%04x-%04x-%04x-%04x%04x%04x',
            mt_rand(0, 0xffff), mt_rand(0, 0xffff),
            mt_rand(0, 0xffff),
            mt_rand(0, 0x0fff) | 0x4000,
            mt_rand(0, 0x3fff) | 0x8000,
            mt_rand(0, 0xffff), mt_rand(0, 0xffff), mt_rand(0, 0xffff)
        );
    }
    
    public function getUUID() {
        return $this->uuid;
    }
}

trait HasMetadata {
    private $metadata = [];
    
    public function setMeta($key, $value) {
        $this->metadata[$key] = $value;
    }
    
    public function getMeta($key) {
        return $this->metadata[$key] ?? null;
    }
    
    public function getAllMeta() {
        return $this->metadata;
    }
}

class User {
    use HasUUID, HasMetadata;
    
    private $name;
    private $email;
    
    public function __construct($name, $email) {
        $this->name = $name;
        $this->email = $email;
        $this->generateUUID();
        $this->setMeta('registered_at', date('Y-m-d H:i:s'));
    }
    
    public function getName() {
        return $this->name;
    }
}

$user = new User("John Doe", "john@example.com");
echo "UUID: " . $user->getUUID();
echo "Name: " . $user->getName();
echo "Registered: " . $user->getMeta('registered_at');

$user->setMeta('last_login', date('Y-m-d H:i:s'));
print_r($user->getAllMeta());
```

---

## Magic Methods

### Q29. What are magic methods in PHP?
**Answer:** Magic methods are special methods that start with double underscores (__). They are automatically called when certain actions are performed on an object.

**Common Magic Methods:**
- `__construct()` - Called when object is created
- `__destruct()` - Called when object is destroyed
- `__get()` - Called when accessing inaccessible property
- `__set()` - Called when setting inaccessible property
- `__call()` - Called when invoking inaccessible method
- `__toString()` - Called when object is treated as string
- `__invoke()` - Called when object is used as function
- `__clone()` - Called when object is cloned
- `__sleep()` - Called during serialize()
- `__wakeup()` - Called during unserialize()

**Real-time Example:**
```php
class SmartObject {
    private $data = [];
    private $methodCalls = [];
    
    // __construct - Initialize object
    public function __construct($initialData = []) {
        $this->data = $initialData;
        echo "Object created\n";
    }
    
    // __destruct - Cleanup when object is destroyed
    public function __destruct() {
        echo "Object destroyed. Method calls: " . count($this->methodCalls) . "\n";
    }
    
    // __get - Handle reading of inaccessible properties
    public function __get($name) {
        echo "__get called for property: $name\n";
        return $this->data[$name] ?? null;
    }
    
    // __set - Handle writing to inaccessible properties
    public function __set($name, $value) {
        echo "__set called for property: $name\n";
        $this->data[$name] = $value;
    }
    
    // __isset - Check if inaccessible property is set
    public function __isset($name) {
        return isset($this->data[$name]);
    }
    
    // __unset - Unset inaccessible property
    public function __unset($name) {
        unset($this->data[$name]);
    }
    
    // __call - Handle inaccessible method calls
    public function __call($name, $arguments) {
        echo "__call invoked for method: $name\n";
        $this->methodCalls[] = $name;
        return "Called $name with " . count($arguments) . " arguments";
    }
    
    // __callStatic - Handle inaccessible static method calls
    public static function __callStatic($name, $arguments) {
        echo "__callStatic invoked for method: $name\n";
        return "Static method $name called";
    }
    
    // __toString - Convert object to string
    public function __toString() {
        return json_encode($this->data);
    }
    
    // __invoke - Make object callable as function
    public function __invoke($param) {
        echo "__invoke called with parameter: $param\n";
        return "Object invoked with $param";
    }
    
    // __clone - Called when object is cloned
    public function __clone() {
        echo "__clone called\n";
        $this->data = array_map(function($item) {
            return is_object($item) ? clone $item : $item;
        }, $this->data);
    }
    
    // __sleep - Called during serialize
    public function __sleep() {
        echo "__sleep called\n";
        return ['data']; // Return properties to serialize
    }
    
    // __wakeup - Called during unserialize
    public function __wakeup() {
        echo "__wakeup called\n";
        $this->methodCalls = [];
    }
}

// Testing magic methods
$obj = new SmartObject(['name' => 'John', 'age' => 30]);

// __get and __set
$obj->email = "john@example.com";  // __set
echo $obj->email;                   // __get

// __isset and __unset
var_dump(isset($obj->email));      // __isset
unset($obj->email);                 // __unset

// __call
$obj->undefinedMethod('param1', 'param2');

// __callStatic
SmartObject::undefinedStaticMethod();

// __toString
echo $obj;  // Converts object to string

// __invoke
echo $obj('test');  // Use object as function

// __clone
$clone = clone $obj;

// __sleep and __wakeup
$serialized = serialize($obj);
$unserialized = unserialize($serialized);

// __destruct called automatically at end
```

### Q30. What is the __toString() magic method?
**Answer:** `__toString()` is called when an object is treated as a string. It must return a string value.

**Real-time Example:**
```php
class Invoice {
    private $invoiceNumber;
    private $customer;
    private $items = [];
    private $total = 0;
    
    public function __construct($invoiceNumber, $customer) {
        $this->invoiceNumber = $invoiceNumber;
        $this->customer = $customer;
    }
    
    public function addItem($name, $price, $quantity) {
        $this->items[] = [
            'name' => $name,
            'price' => $price,
            'quantity' => $quantity,
            'subtotal' => $price * $quantity
        ];
        $this->total += $price * $quantity;
    }
    
    public function __toString() {
        $output = "=====================================\n";
        $output .= "           INVOICE\n";
        $output .= "=====================================\n";
        $output .= "Invoice #: {$this->invoiceNumber}\n";
        $output .= "Customer: {$this->customer}\n";
        $output .= "Date: " . date('Y-m-d') . "\n";
        $output .= "-------------------------------------\n";
        
        foreach ($this->items as $item) {
            $output .= sprintf(
                "%-20s %2d x $%6.2f = $%8.2f\n",
                $item['name'],
                $item['quantity'],
                $item['price'],
                $item['subtotal']
            );
        }
        
        $output .= "-------------------------------------\n";
        $output .= sprintf("%-34s $%8.2f\n", "TOTAL:", $this->total);
        $output .= "=====================================\n";
        
        return $output;
    }
}

$invoice = new Invoice("INV-2025-001", "John Doe");
$invoice->addItem("Laptop", 999.99, 1);
$invoice->addItem("Mouse", 25.50, 2);
$invoice->addItem("Keyboard", 75.00, 1);

// Automatically calls __toString()
echo $invoice;

// Can also be used in concatenation
$emailBody = "Dear Customer,\n\nYour invoice:\n" . $invoice;
echo $emailBody;
```

---

## Design Patterns

### Q31. What is a Singleton design pattern?
**Answer:** Singleton ensures a class has only one instance and provides a global point of access to it.

**Real-time Example:**
```php
class DatabaseConnection {
    private static $instance = null;
    private $connection;
    
    // Private constructor prevents direct instantiation
    private function __construct() {
        $this->connection = new PDO(
            "mysql:host=localhost;dbname=test",
            "root",
            ""
        );
        echo "Database connection created\n";
    }
    
    // Prevent cloning
    private function __clone() {}
    
    // Prevent unserialization
    public function __wakeup() {
        throw new Exception("Cannot unserialize singleton");
    }
    
    // Get the single instance
    public static function getInstance() {
        if (self::$instance === null) {
            self::$instance = new self();
        }
        return self::$instance;
    }
    
    public function query($sql) {
        return $this->connection->query($sql);
    }
}

// Usage
$db1 = DatabaseConnection::getInstance();  // Creates connection
$db2 = DatabaseConnection::getInstance();  // Returns same instance

var_dump($db1 === $db2);  // true - same instance

// Cannot create new instance
// $db = new DatabaseConnection();  // Error: private constructor
```

### Q32. What is a Factory design pattern?
**Answer:** Factory pattern provides an interface for creating objects without specifying their exact class.

**Real-time Example:**
```php
interface PaymentGateway {
    public function processPayment($amount);
    public function getTransactionFee($amount);
}

class PayPalGateway implements PaymentGateway {
    public function processPayment($amount) {
        return "Processing $$amount via PayPal";
    }
    
    public function getTransactionFee($amount) {
        return $amount * 0.029 + 0.30;  // 2.9% + $0.30
    }
}

class StripeGateway implements PaymentGateway {
    public function processPayment($amount) {
        return "Processing $$amount via Stripe";
    }
    
    public function getTransactionFee($amount) {
        return $amount * 0.029 + 0.30;  // 2.9% + $0.30
    }
}

class BraintreeGateway implements PaymentGateway {
    public function processPayment($amount) {
        return "Processing $$amount via Braintree";
    }
    
    public function getTransactionFee($amount) {
        return $amount * 0.025 + 0.15;  // 2.5% + $0.15
    }
}

// Factory Class
class PaymentGatewayFactory {
    public static function create($type) {
        switch (strtolower($type)) {
            case 'paypal':
                return new PayPalGateway();
            case 'stripe':
                return new StripeGateway();
            case 'braintree':
                return new BraintreeGateway();
            default:
                throw new Exception("Invalid payment gateway type: $type");
        }
    }
}

// Usage
$userChoice = 'stripe';  // From user input or configuration

$gateway = PaymentGatewayFactory::create($userChoice);
echo $gateway->processPayment(100);
echo "Fee: $" . $gateway->getTransactionFee(100);

// Easy to switch
$gateway = PaymentGatewayFactory::create('paypal');
echo $gateway->processPayment(100);
```

### Q33. What is the Strategy design pattern?
**Answer:** Strategy pattern defines a family of algorithms, encapsulates each one, and makes them interchangeable.

**Real-time Example:**
```php
interface ShippingStrategy {
    public function calculate($weight, $distance);
    public function getEstimatedDays();
}

class StandardShipping implements ShippingStrategy {
    public function calculate($weight, $distance) {
        return ($weight * 0.5) + ($distance * 0.1);
    }
    
    public function getEstimatedDays() {
        return "5-7 business days";
    }
}

class ExpressShipping implements ShippingStrategy {
    public function calculate($weight, $distance) {
        return ($weight * 1.0) + ($distance * 0.2) + 10;
    }
    
    public function getEstimatedDays() {
        return "2-3 business days";
    }
}

class OvernightShipping implements ShippingStrategy {
    public function calculate($weight, $distance) {
        return ($weight * 2.0) + ($distance * 0.5) + 25;
    }
    
    public function getEstimatedDays() {
        return "1 business day";
    }
}

class ShippingCalculator {
    private $strategy;
    
    public function setStrategy(ShippingStrategy $strategy) {
        $this->strategy = $strategy;
    }
    
    public function calculateCost($weight, $distance) {
        if (!$this->strategy) {
            throw new Exception("Shipping strategy not set");
        }
        
        $cost = $this->strategy->calculate($weight, $distance);
        $days = $this->strategy->getEstimatedDays();
        
        return [
            'cost' => round($cost, 2),
            'estimated_delivery' => $days
        ];
    }
}

// Usage
$calculator = new ShippingCalculator();
$weight = 5;  // kg
$distance = 100;  // km

// Standard shipping
$calculator->setStrategy(new StandardShipping());
$result = $calculator->calculateCost($weight, $distance);
echo "Standard: $" . $result['cost'] . " - " . $result['estimated_delivery'] . "\n";

// Express shipping
$calculator->setStrategy(new ExpressShipping());
$result = $calculator->calculateCost($weight, $distance);
echo "Express: $" . $result['cost'] . " - " . $result['estimated_delivery'] . "\n";

// Overnight shipping
$calculator->setStrategy(new OvernightShipping());
$result = $calculator->calculateCost($weight, $distance);
echo "Overnight: $" . $result['cost'] . " - " . $result['estimated_delivery'] . "\n";
```

### Q34. What is the Observer design pattern?
**Answer:** Observer pattern defines a one-to-many dependency between objects so that when one object changes state, all its dependents are notified.

**Real-time Example:**
```php
interface Observer {
    public function update($event, $data);
}

class Subject {
    private $observers = [];
    
    public function attach(Observer $observer) {
        $this->observers[] = $observer;
    }
    
    public function detach(Observer $observer) {
        $key = array_search($observer, $this->observers, true);
        if ($key !== false) {
            unset($this->observers[$key]);
        }
    }
    
    protected function notify($event, $data = []) {
        foreach ($this->observers as $observer) {
            $observer->update($event, $data);
        }
    }
}

class Order extends Subject {
    private $orderId;
    private $status;
    
    public function __construct($orderId) {
        $this->orderId = $orderId;
        $this->status = 'pending';
    }
    
    public function setStatus($status) {
        $this->status = $status;
        $this->notify('status_changed', [
            'order_id' => $this->orderId,
            'status' => $status
        ]);
    }
    
    public function getStatus() {
        return $this->status;
    }
}

class EmailNotifier implements Observer {
    public function update($event, $data) {
        if ($event === 'status_changed') {
            echo "EMAIL: Order #{$data['order_id']} status changed to {$data['status']}\n";
            // Send email logic here
        }
    }
}

class SMSNotifier implements Observer {
    public function update($event, $data) {
        if ($event === 'status_changed') {
            echo "SMS: Your order #{$data['order_id']} is now {$data['status']}\n";
            // Send SMS logic here
        }
    }
}

class LoggerObserver implements Observer {
    public function update($event, $data) {
        echo "LOG: " . date('Y-m-d H:i:s') . " - Event: $event - " . json_encode($data) . "\n";
        // Write to log file
    }
}

class InventoryUpdater implements Observer {
    public function update($event, $data) {
        if ($event === 'status_changed' && $data['status'] === 'completed') {
            echo "INVENTORY: Updating stock for order #{$data['order_id']}\n";
            // Update inventory logic
        }
    }
}

// Usage
$order = new Order(12345);

// Attach observers
$order->attach(new EmailNotifier());
$order->attach(new SMSNotifier());
$order->attach(new LoggerObserver());
$order->attach(new InventoryUpdater());

// Change order status - all observers notified automatically
$order->setStatus('processing');
echo "\n";
$order->setStatus('shipped');
echo "\n";
$order->setStatus('completed');
```

### Q35. What is Dependency Injection?
**Answer:** Dependency Injection is a design pattern where dependencies are provided to a class rather than the class creating them itself.

**Real-time Example:**
```php
// Bad practice - tight coupling
class BadUserService {
    private $db;
    
    public function __construct() {
        // Creating dependency inside - hard to test
        $this->db = new PDO("mysql:host=localhost;dbname=test", "root", "");
    }
    
    public function getUser($id) {
        return $this->db->query("SELECT * FROM users WHERE id = $id")->fetch();
    }
}

// Good practice - dependency injection
interface DatabaseInterface {
    public function query($sql);
}

class MySQLDatabase implements DatabaseInterface {
    private $pdo;
    
    public function __construct($host, $dbname, $username, $password) {
        $this->pdo = new PDO("mysql:host=$host;dbname=$dbname", $username, $password);
    }
    
    public function query($sql) {
        return $this->pdo->query($sql);
    }
}

class MockDatabase implements DatabaseInterface {
    public function query($sql) {
        // Return mock data for testing
        return ['id' => 1, 'name' => 'Test User', 'email' => 'test@example.com'];
    }
}

class UserService {
    private $database;
    private $logger;
    private $cache;
    
    // Constructor Injection
    public function __construct(
        DatabaseInterface $database,
        LoggerInterface $logger,
        CacheInterface $cache
    ) {
        $this->database = $database;
        $this->logger = $logger;
        $this->cache = $cache;
    }
    
    public function getUser($id) {
        // Check cache first
        $cacheKey = "user_$id";
        $cached = $this->cache->get($cacheKey);
        
        if ($cached) {
            $this->logger->info("User $id loaded from cache");
            return $cached;
        }
        
        // Load from database
        $user = $this->database->query("SELECT * FROM users WHERE id = $id");
        
        // Store in cache
        $this->cache->set($cacheKey, $user, 3600);
        
        $this->logger->info("User $id loaded from database");
        
        return $user;
    }
}

interface LoggerInterface {
    public function info($message);
}

class FileLogger implements LoggerInterface {
    public function info($message) {
        echo "[LOG] $message\n";
    }
}

interface CacheInterface {
    public function get($key);
    public function set($key, $value, $ttl);
}

class ArrayCache implements CacheInterface {
    private $cache = [];
    
    public function get($key) {
        return $this->cache[$key] ?? null;
    }
    
    public function set($key, $value, $ttl) {
        $this->cache[$key] = $value;
    }
}

// Usage - Easy to inject different implementations
$database = new MySQLDatabase('localhost', 'test', 'root', '');
$logger = new FileLogger();
$cache = new ArrayCache();

$userService = new UserService($database, $logger, $cache);
$user = $userService->getUser(1);

// For testing - inject mock dependencies
$mockDatabase = new MockDatabase();
$testService = new UserService($mockDatabase, $logger, $cache);
$testUser = $testService->getUser(1);  // Uses mock data
```

---

## Additional Important Questions

### Q36. What is type hinting in PHP?
**Answer:** Type hinting allows you to specify the expected data type of an argument in a function declaration.

**Real-time Example:**
```php
class User {
    private $id;
    private $name;
    
    public function __construct(int $id, string $name) {
        $this->id = $id;
        $this->name = $name;
    }
    
    public function getId(): int {
        return $this->id;
    }
    
    public function getName(): string {
        return $this->name;
    }
}

class UserRepository {
    private $users = [];
    
    // Type hinting for parameters and return type
    public function add(User $user): void {
        $this->users[$user->getId()] = $user;
    }
    
    public function findById(int $id): ?User {
        return $this->users[$id] ?? null;
    }
    
    public function getAll(): array {
        return $this->users;
    }
    
    public function count(): int {
        return count($this->users);
    }
    
    public function remove(User $user): bool {
        if (isset($this->users[$user->getId()])) {
            unset($this->users[$user->getId()]);
            return true;
        }
        return false;
    }
}

$repo = new UserRepository();
$user1 = new User(1, "John Doe");
$user2 = new User(2, "Jane Smith");

$repo->add($user1);
$repo->add($user2);

$found = $repo->findById(1);
echo $found->getName();  // John Doe

// This would cause a TypeError:
// $repo->add("not a user");  // Error: must be instance of User
// $repo->findById("string"); // Error: must be int
```

### Q37. What is the difference between == and === in PHP?
**Answer:**
- `==` (loose comparison) - Checks value equality with type juggling
- `===` (strict comparison) - Checks both value and type equality

**Real-time Example:**
```php
class ComparisonExample {
    public function demonstrate() {
        // Numeric comparisons
        var_dump(5 == "5");      // true - loose comparison
        var_dump(5 === "5");     // false - different types
        
        // Boolean comparisons
        var_dump(1 == true);     // true
        var_dump(1 === true);    // false - different types
        
        var_dump(0 == false);    // true
        var_dump(0 === false);   // false
        
        // Array comparisons
        $arr1 = ['a' => 1, 'b' => 2];
        $arr2 = ['b' => 2, 'a' => 1];
        var_dump($arr1 == $arr2);   // true - same values
        var_dump($arr1 === $arr2);  // false - different order
        
        // Null comparisons
        var_dump(null == false);    // true
        var_dump(null === false);   // false
        
        // String comparisons
        var_dump("0" == false);     // true
        var_dump("0" === false);    // false
    }
    
    // Real-world usage example
    public function processPayment($paymentStatus) {
        // Bad practice - can lead to bugs
        if ($paymentStatus == true) {
            // This will execute for "1", 1, "yes", true, etc.
            echo "Payment successful\n";
        }
        
        // Good practice - explicit check
        if ($paymentStatus === true) {
            // Only executes for boolean true
            echo "Payment successful\n";
        }
        
        // Better for checking existence
        if ($paymentStatus !== null) {
            echo "Payment status exists\n";
        }
    }
}

// Practical example
function getUserRole($userId) {
    $roles = [
        1 => 'admin',
        2 => 'moderator',
        3 => 'user'
    ];
    
    // Using ===for array key existence is important
    if (isset($roles[$userId])) {
        return $roles[$userId];
    }
    
    return null;
}

// This could cause issues with ==
$role = getUserRole("1");  // String "1"

if ($role == $roles[1]) {  // Works but not type-safe
    echo "Admin\n";
}

if ($role === $roles[1]) {  // Better - type-safe comparison
    echo "Admin\n";
}
```

### Q38. What is late static binding in PHP?
**Answer:** Late static binding allows referencing the called class in a context of static inheritance using the `static` keyword instead of `self`.

**Real-time Example:**
```php
class Model {
    protected static $table;
    
    public static function getTable() {
        // self refers to Model class
        return self::$table;
    }
    
    public static function getTableLateBinding() {
        // static refers to the called class
        return static::$table;
    }
    
    public static function find($id) {
        return "SELECT * FROM " . static::$table . " WHERE id = $id";
    }
    
    public static function create($data) {
        $columns = implode(', ', array_keys($data));
        $values = implode(', ', array_map(function($v) {
            return "'$v'";
        }, array_values($data)));
        
        return "INSERT INTO " . static::$table . " ($columns) VALUES ($values)";
    }
}

class User extends Model {
    protected static $table = 'users';
}

class Product extends Model {
    protected static $table = 'products';
}

class Order extends Model {
    protected static $table = 'orders';
}

// Late static binding in action
echo User::find(1);
// Output: SELECT * FROM users WHERE id = 1

echo Product::find(5);
// Output: SELECT * FROM products WHERE id = 5

echo Order::create(['user_id' => 1, 'total' => 100]);
// Output: INSERT INTO orders (user_id, total) VALUES ('1', '100')

// Demonstrating difference between self and static
class A {
    public static function who() {
        echo __CLASS__;
    }
    
    public static function testSelf() {
        self::who();  // Always calls A::who()
    }
    
    public static function testStatic() {
        static::who();  // Calls the actual class's who()
    }
}

class B extends A {
    public static function who() {
        echo __CLASS__;
    }
}

B::testSelf();    // Output: A (self binds to class where it's written)
B::testStatic();  // Output: B (static binds to called class)
```

### Q39. What is namespace in PHP?
**Answer:** Namespaces are a way to encapsulate and organize code, preventing name collisions between classes, functions, and constants.

**Real-time Example:**
```php
// File: app/Models/User.php
namespace App\Models;

class User {
    public function save() {
        return "Saving user to database";
    }
}

// File: app/Services/User.php
namespace App\Services;

class User {
    public function sendWelcomeEmail() {
        return "Sending welcome email";
    }
}

// File: app/Controllers/UserController.php
namespace App\Controllers;

use App\Models\User as UserModel;
use App\Services\User as UserService;

class UserController {
    private $userModel;
    private $userService;
    
    public function __construct() {
        $this->userModel = new UserModel();
        $this->userService = new UserService();
    }
    
    public function register($data) {
        // Use fully qualified class name
        $validator = new \App\Validators\UserValidator();
        
        if ($validator->validate($data)) {
            $this->userModel->save();
            $this->userService->sendWelcomeEmail();
        }
    }
}

// File: app/Validators/UserValidator.php
namespace App\Validators;

class UserValidator {
    public function validate($data) {
        return !empty($data['email']) && !empty($data['password']);
    }
}

// File: app/Helpers/functions.php
namespace App\Helpers;

function formatDate($date) {
    return date('Y-m-d', strtotime($date));
}

function slugify($text) {
    return strtolower(trim(preg_replace('/[^A-Za-z0-9-]+/', '-', $text), '-'));
}

// Usage in another file
namespace App;

use App\Controllers\UserController;
use App\Models\User;
use function App\Helpers\formatDate;
use function App\Helpers\slugify;
use const App\Config\DB_HOST;

$controller = new UserController();
$user = new User();

$formattedDate = formatDate('2025-10-12');
$slug = slugify('Hello World!');

// Accessing global classes
$datetime = new \DateTime();  // Global DateTime class
$pdo = new \PDO();            // Global PDO class
```

### Q40. What is autoloading in PHP?
**Answer:** Autoloading automatically loads class files when they're needed, eliminating the need for manual `require` or `include` statements.

**Real-time Example:**
```php
// File: autoload.php
spl_autoload_register(function ($class) {
    // PSR-4 compliant autoloader
    $prefix = 'App\\';
    $baseDir = __DIR__ . '/src/';
    
    // Check if class uses the namespace prefix
    $len = strlen($prefix);
    if (strncmp($prefix, $class, $len) !== 0) {
        return;
    }
    
    // Get relative class name
    $relativeClass = substr($class, $len);
    
    // Replace namespace separators with directory separators
    $file = $baseDir . str_replace('\\', '/', $relativeClass) . '.php';
    
    // If file exists, require it
    if (file_exists($file)) {
        require $file;
    }
});

// File: src/Database/Connection.php
namespace App\Database;

class Connection {
    public function connect() {
        return "Database connected";
    }
}

// File: src/Models/User.php
namespace App\Models;

class User {
    public function find($id) {
        return "Finding user $id";
    }
}

// File: src/Services/EmailService.php
namespace App\Services;

class EmailService {
    public function send($to, $subject, $message) {
        return "Sending email to $to";
    }
}

// File: index.php
require 'autoload.php';

// Classes are automatically loaded when used
$db = new App\Database\Connection();  // Autoloads Connection.php
echo $db->connect();

$user = new App\Models\User();        // Autoloads User.php
echo $user->find(1);

$email = new App\Services\EmailService();  // Autoloads EmailService.php
echo $email->send('user@example.com', 'Welcome', 'Hello!');

// Using Composer's autoloader (modern approach)
// composer.json:
/*
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
*/

// After running: composer dump-autoload
// File: index.php with Composer
require 'vendor/autoload.php';

use App\Models\User;
use App\Services\EmailService;

$user = new User();
$email = new EmailService();
```

---

## Summary

This README covers essential OOP concepts for PHP developers with 7+ years of experience:

1. **Basic OOP** - Classes, Objects, Constructors, Destructors
2. **Classes & Objects** - Access modifiers, Static members, Method chaining
3. **Inheritance** - Parent-child relationships, Method overriding, Final keyword
4. **Polymorphism** - Multiple forms, Runtime polymorphism
5. **Encapsulation** - Data hiding, Getters/Setters
6. **Abstraction** - Abstract classes, Hiding complexity
7. **Interfaces** - Contracts for classes
8. **Traits** - Code reuse mechanism
9. **Magic Methods** - Special PHP methods
10. **Design Patterns** - Singleton, Factory, Strategy, Observer, Dependency Injection

All examples are real-world scenarios commonly used in production applications, making them easy to understand and implement in interviews.

