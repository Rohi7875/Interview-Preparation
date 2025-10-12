# PHP Core Interview Questions & Answers
### For 7+ Years Experienced PHP Developers

---

## Table of Contents
1. [PHP Basics](#php-basics)
2. [Data Types & Variables](#data-types--variables)
3. [Functions](#functions)
4. [Arrays](#arrays)
5. [String Manipulation](#string-manipulation)
6. [File Handling](#file-handling)
7. [Sessions & Cookies](#sessions--cookies)
8. [Error Handling & Exceptions](#error-handling--exceptions)
9. [Security](#security)
10. [Performance & Best Practices](#performance--best-practices)

---

## PHP Basics

### Q1. What are the differences between PHP 7 and PHP 8?
**Answer:** 

**PHP 8 Features:**
- JIT (Just-In-Time) Compiler
- Named Arguments
- Union Types
- Match Expression
- Nullsafe Operator
- Constructor Property Promotion
- Attributes (Annotations)
- Mixed Type
- Static Return Type

**Real-time Example:**
```php
// Named Arguments (PHP 8)
function create_user($name, $email, $role = 'user', $active = true) {
    return compact('name', 'email', 'role', 'active');
}

// PHP 7 way
$user = create_user('John', 'john@example.com', 'admin', true);

// PHP 8 way - can skip and reorder arguments
$user = create_user(
    email: 'john@example.com',
    name: 'John',
    active: false
);

// Union Types (PHP 8)
function process_id(int|string $id): int|string {
    return is_int($id) ? $id : (int) $id;
}

// Match Expression (PHP 8)
$status = 'pending';

// Old way - switch
switch ($status) {
    case 'pending':
        $message = 'Order is pending';
        break;
    case 'processing':
        $message = 'Order is processing';
        break;
    case 'completed':
        $message = 'Order is completed';
        break;
    default:
        $message = 'Unknown status';
}

// PHP 8 match expression
$message = match($status) {
    'pending' => 'Order is pending',
    'processing' => 'Order is processing',
    'completed' => 'Order is completed',
    default => 'Unknown status'
};

// Nullsafe Operator (PHP 8)
// Old way
$country = null;
if ($user !== null) {
    if ($user->getAddress() !== null) {
        $country = $user->getAddress()->getCountry();
    }
}

// PHP 8 way
$country = $user?->getAddress()?->getCountry();

// Constructor Property Promotion (PHP 8)
// Old way
class UserOld {
    private string $name;
    private string $email;
    
    public function __construct(string $name, string $email) {
        $this->name = $name;
        $this->email = $email;
    }
}

// PHP 8 way - much cleaner
class User {
    public function __construct(
        private string $name,
        private string $email,
        private string $role = 'user'
    ) {}
    
    public function getName(): string {
        return $this->name;
    }
}

// Attributes (PHP 8)
#[Route('/api/users', methods: ['GET', 'POST'])]
#[Authorize('admin')]
class UserController {
    #[Route('/api/users/{id}', methods: ['GET'])]
    public function show(int $id) {
        return User::find($id);
    }
}
```

### Q2. What is the difference between include, require, include_once, and require_once?
**Answer:**
- `include` - Includes file, warning on failure, script continues
- `require` - Includes file, fatal error on failure, script stops
- `include_once` - Includes file only once, warning on failure
- `require_once` - Includes file only once, fatal error on failure

**Real-time Example:**
```php
// config.php
<?php
define('DB_HOST', 'localhost');
define('DB_USER', 'root');
define('DB_PASS', 'password');
define('DB_NAME', 'myapp');

// functions.php
<?php
function sanitize_input($data) {
    return htmlspecialchars(strip_tags(trim($data)));
}

function redirect($url) {
    header("Location: $url");
    exit();
}

// Application structure example
// index.php
<?php
// Must have configuration - use require_once
require_once 'config.php';

// Must have database connection
require_once 'database.php';

// Must have functions - include only once
require_once 'functions.php';

// Optional analytics - won't break app if missing
include_once 'analytics.php';

// Optional debugging tools - only in development
if (ENVIRONMENT === 'development') {
    include 'debug_toolbar.php';
}

// Load page
require 'pages/home.php';

// Real-world autoloader example
spl_autoload_register(function ($class) {
    $file = str_replace('\\', '/', $class) . '.php';
    
    if (file_exists($file)) {
        require_once $file;
    }
});
```

### Q3. What are PHP superglobals?
**Answer:** Superglobals are built-in variables that are always available in all scopes.

**Superglobals:**
- `$GLOBALS` - References all variables in global scope
- `$_SERVER` - Server and execution environment information
- `$_GET` - HTTP GET variables
- `$_POST` - HTTP POST variables
- `$_FILES` - HTTP file upload variables
- `$_COOKIE` - HTTP cookies
- `$_SESSION` - Session variables
- `$_REQUEST` - HTTP request variables (GET, POST, COOKIE)
- `$_ENV` - Environment variables

**Real-time Example:**
```php
// Request handling class
class Request {
    
    // Get client IP address
    public static function getIpAddress() {
        if (!empty($_SERVER['HTTP_CLIENT_IP'])) {
            return $_SERVER['HTTP_CLIENT_IP'];
        } elseif (!empty($_SERVER['HTTP_X_FORWARDED_FOR'])) {
            return $_SERVER['HTTP_X_FORWARDED_FOR'];
        } else {
            return $_SERVER['REMOTE_ADDR'];
        }
    }
    
    // Get request method
    public static function getMethod() {
        return $_SERVER['REQUEST_METHOD'];
    }
    
    // Check if HTTPS
    public static function isSecure() {
        return (!empty($_SERVER['HTTPS']) && $_SERVER['HTTPS'] !== 'off')
            || $_SERVER['SERVER_PORT'] == 443;
    }
    
    // Get current URL
    public static function getCurrentUrl() {
        $protocol = self::isSecure() ? 'https' : 'http';
        return $protocol . '://' . $_SERVER['HTTP_HOST'] . $_SERVER['REQUEST_URI'];
    }
    
    // Get user agent
    public static function getUserAgent() {
        return $_SERVER['HTTP_USER_AGENT'] ?? '';
    }
    
    // Get input from GET/POST
    public static function input($key, $default = null, $method = 'post') {
        $source = ($method === 'get') ? $_GET : $_POST;
        
        if (isset($source[$key])) {
            return self::sanitize($source[$key]);
        }
        
        return $default;
    }
    
    // Sanitize input
    private static function sanitize($data) {
        if (is_array($data)) {
            return array_map([self::class, 'sanitize'], $data);
        }
        
        return htmlspecialchars(strip_tags(trim($data)), ENT_QUOTES, 'UTF-8');
    }
    
    // Get all POST data
    public static function all() {
        return array_map([self::class, 'sanitize'], $_POST);
    }
    
    // Check if has file
    public static function hasFile($key) {
        return isset($_FILES[$key]) && $_FILES[$key]['error'] !== UPLOAD_ERR_NO_FILE;
    }
    
    // Get uploaded file
    public static function file($key) {
        if (self::hasFile($key)) {
            return $_FILES[$key];
        }
        return null;
    }
}

// Usage examples
$ip = Request::getIpAddress();
$method = Request::getMethod();
$isHttps = Request::isSecure();
$currentUrl = Request::getCurrentUrl();

if ($method === 'POST') {
    $username = Request::input('username');
    $email = Request::input('email');
    $allData = Request::all();
    
    if (Request::hasFile('avatar')) {
        $avatar = Request::file('avatar');
        // Process file upload
    }
}

// Session management example
class Session {
    
    public static function start() {
        if (session_status() === PHP_SESSION_NONE) {
            session_start();
        }
    }
    
    public static function set($key, $value) {
        self::start();
        $_SESSION[$key] = $value;
    }
    
    public static function get($key, $default = null) {
        self::start();
        return $_SESSION[$key] ?? $default;
    }
    
    public static function has($key) {
        self::start();
        return isset($_SESSION[$key]);
    }
    
    public static function remove($key) {
        self::start();
        unset($_SESSION[$key]);
    }
    
    public static function destroy() {
        self::start();
        session_destroy();
    }
    
    public static function flash($key, $value) {
        self::set($key, $value);
        self::set($key . '_flash', true);
    }
    
    public static function getFlash($key, $default = null) {
        $value = self::get($key, $default);
        
        if (self::get($key . '_flash')) {
            self::remove($key);
            self::remove($key . '_flash');
        }
        
        return $value;
    }
}

// Usage
Session::set('user_id', 123);
Session::flash('success', 'Login successful!');

$userId = Session::get('user_id');
$message = Session::getFlash('success');
```

---

## Data Types & Variables

### Q4. What are the data types in PHP?
**Answer:** 

**Scalar Types:**
- Boolean (bool)
- Integer (int)
- Float (float, double)
- String (string)

**Compound Types:**
- Array (array)
- Object (object)
- Callable (callable)
- Iterable (iterable)

**Special Types:**
- Resource (resource)
- NULL (null)

**Real-time Example:**
```php
// Type juggling and strict types
declare(strict_types=1);

class TypeExample {
    
    // Scalar types
    public function processPayment(float $amount, string $currency, bool $isRecurring): int {
        $transactionId = random_int(100000, 999999);
        
        // Type casting
        $amountCents = (int) ($amount * 100);
        $currencyCode = (string) $currency;
        $recurring = (bool) $isRecurring;
        
        return $transactionId;
    }
    
    // Array type
    public function processOrders(array $orders): array {
        $processed = [];
        
        foreach ($orders as $order) {
            $processed[] = $this->processOrder($order);
        }
        
        return $processed;
    }
    
    // Callable type
    public function filterData(array $data, callable $callback): array {
        return array_filter($data, $callback);
    }
    
    // Object type
    public function saveUser(object $user): bool {
        // Save user to database
        return true;
    }
    
    // Nullable types
    public function findUser(int $id): ?object {
        // May return User object or null
        return $id > 0 ? new stdClass() : null;
    }
    
    // Union types (PHP 8+)
    public function processId(int|string $id): int {
        return is_int($id) ? $id : (int) $id;
    }
    
    // Mixed type (PHP 8+)
    public function processValue(mixed $value): mixed {
        if (is_array($value)) {
            return count($value);
        } elseif (is_string($value)) {
            return strlen($value);
        } elseif (is_numeric($value)) {
            return $value * 2;
        }
        
        return $value;
    }
    
    // Type checking
    public function checkTypes($var) {
        $info = [
            'value' => $var,
            'type' => gettype($var),
            'is_bool' => is_bool($var),
            'is_int' => is_int($var),
            'is_float' => is_float($var),
            'is_string' => is_string($var),
            'is_array' => is_array($var),
            'is_object' => is_object($var),
            'is_null' => is_null($var),
            'is_numeric' => is_numeric($var),
            'is_scalar' => is_scalar($var),
            'is_callable' => is_callable($var),
        ];
        
        return $info;
    }
}

// Usage
$example = new TypeExample();

// Strict types enforce type checking
$transactionId = $example->processPayment(99.99, 'USD', true);

// Type casting examples
$strNumber = "123";
$intNumber = (int) $strNumber;  // 123
$floatNumber = (float) $strNumber;  // 123.0

$intValue = 100;
$stringValue = (string) $intValue;  // "100"

$truthyValue = 1;
$boolValue = (bool) $truthyValue;  // true

// Array casting
$object = new stdClass();
$object->name = "John";
$object->age = 30;
$array = (array) $object;  // ['name' => 'John', 'age' => 30]

// Variable type checking
$value = "123";
if (is_numeric($value)) {
    $numericValue = (int) $value;
}

// Callable examples
$result = $example->filterData(
    [1, 2, 3, 4, 5],
    function($n) { return $n % 2 === 0; }
);
```

### Q5. What is type hinting and type declaration?
**Answer:** Type hinting (declaration) specifies the expected data type of arguments and return values.

**Real-time Example:**
```php
// Type hinting examples
class ProductService {
    
    // Constructor with type hinting
    public function __construct(
        private PDO $database,
        private CacheInterface $cache,
        private LoggerInterface $logger
    ) {}
    
    // Parameter and return type hinting
    public function createProduct(
        string $name,
        float $price,
        int $quantity,
        ?string $description = null
    ): int {
        $data = [
            'name' => $name,
            'price' => $price,
            'quantity' => $quantity,
            'description' => $description,
            'created_at' => date('Y-m-d H:i:s')
        ];
        
        $stmt = $this->database->prepare(
            "INSERT INTO products (name, price, quantity, description, created_at) 
             VALUES (:name, :price, :quantity, :description, :created_at)"
        );
        
        $stmt->execute($data);
        
        return (int) $this->database->lastInsertId();
    }
    
    // Array type hint
    public function bulkCreate(array $products): array {
        $createdIds = [];
        
        foreach ($products as $product) {
            $createdIds[] = $this->createProduct(
                $product['name'],
                $product['price'],
                $product['quantity'],
                $product['description'] ?? null
            );
        }
        
        return $createdIds;
    }
    
    // Object type hint with nullable return
    public function findProduct(int $id): ?Product {
        $cacheKey = "product_$id";
        
        // Check cache
        if ($this->cache->has($cacheKey)) {
            return $this->cache->get($cacheKey);
        }
        
        // Query database
        $stmt = $this->database->prepare("SELECT * FROM products WHERE id = ?");
        $stmt->execute([$id]);
        $data = $stmt->fetch(PDO::FETCH_ASSOC);
        
        if (!$data) {
            return null;
        }
        
        $product = new Product($data);
        
        // Store in cache
        $this->cache->set($cacheKey, $product, 3600);
        
        return $product;
    }
    
    // Callable type hint
    public function filterProducts(array $products, callable $filter): array {
        return array_filter($products, $filter);
    }
    
    // Void return type
    public function logActivity(string $action, array $data): void {
        $this->logger->info("Product $action", $data);
    }
    
    // Self return type (for fluent interface)
    public function setDatabase(PDO $database): self {
        $this->database = $database;
        return $this;
    }
    
    // Static return type
    public static function getInstance(): static {
        return new static();
    }
    
    // Union types (PHP 8+)
    public function processPrice(int|float|string $price): float {
        return (float) $price;
    }
    
    // Intersection types (PHP 8.1+)
    public function process(Countable&Iterator $collection): int {
        return count($collection);
    }
}

// Product class
class Product {
    public function __construct(
        private array $data
    ) {}
    
    public function getId(): int {
        return (int) $this->data['id'];
    }
    
    public function getName(): string {
        return (string) $this->data['name'];
    }
    
    public function getPrice(): float {
        return (float) $this->data['price'];
    }
    
    public function getQuantity(): int {
        return (int) $this->data['quantity'];
    }
}

// Interface examples
interface CacheInterface {
    public function get(string $key): mixed;
    public function set(string $key, mixed $value, int $ttl): bool;
    public function has(string $key): bool;
    public function delete(string $key): bool;
}

interface LoggerInterface {
    public function info(string $message, array $context = []): void;
    public function error(string $message, array $context = []): void;
}
```

---

## Functions

### Q6. What are anonymous functions and closures?
**Answer:** Anonymous functions (closures) are functions without a specified name. They can be stored in variables and passed as arguments.

**Real-time Example:**
```php
// Basic anonymous function
$greet = function($name) {
    return "Hello, $name!";
};

echo $greet('John');  // Hello, John!

// Closure with use keyword
$multiplier = 10;

$multiply = function($number) use ($multiplier) {
    return $number * $multiplier;
};

echo $multiply(5);  // 50

// Closure with reference
$counter = 0;

$increment = function() use (&$counter) {
    $counter++;
    return $counter;
};

echo $increment();  // 1
echo $increment();  // 2
echo $increment();  // 3

// Real-world examples

// 1. Array filtering
class ProductFilter {
    
    public function filterByPrice(array $products, float $minPrice, float $maxPrice): array {
        return array_filter($products, function($product) use ($minPrice, $maxPrice) {
            return $product['price'] >= $minPrice && $product['price'] <= $maxPrice;
        });
    }
    
    public function filterByCategory(array $products, string $category): array {
        return array_filter($products, function($product) use ($category) {
            return $product['category'] === $category;
        });
    }
    
    public function sortByPrice(array $products, string $direction = 'asc'): array {
        usort($products, function($a, $b) use ($direction) {
            if ($direction === 'asc') {
                return $a['price'] <=> $b['price'];
            }
            return $b['price'] <=> $a['price'];
        });
        
        return $products;
    }
}

// 2. Route handling with closures
class Router {
    private array $routes = [];
    
    public function get(string $path, callable $handler): void {
        $this->routes['GET'][$path] = $handler;
    }
    
    public function post(string $path, callable $handler): void {
        $this->routes['POST'][$path] = $handler;
    }
    
    public function dispatch(string $method, string $path): mixed {
        if (isset($this->routes[$method][$path])) {
            return call_user_func($this->routes[$method][$path]);
        }
        
        return null;
    }
}

// Usage
$router = new Router();

$router->get('/users', function() {
    return ['users' => User::all()];
});

$router->post('/users', function() {
    $data = json_decode(file_get_contents('php://input'), true);
    return User::create($data);
});

// 3. Middleware pattern
class Middleware {
    
    public static function auth(callable $next): callable {
        return function() use ($next) {
            if (!isset($_SESSION['user_id'])) {
                http_response_code(401);
                return ['error' => 'Unauthorized'];
            }
            
            return $next();
        };
    }
    
    public static function admin(callable $next): callable {
        return function() use ($next) {
            if ($_SESSION['role'] !== 'admin') {
                http_response_code(403);
                return ['error' => 'Forbidden'];
            }
            
            return $next();
        };
    }
    
    public static function rateLimit(int $limit, callable $next): callable {
        return function() use ($limit, $next) {
            $key = 'rate_limit_' . $_SERVER['REMOTE_ADDR'];
            
            if (!isset($_SESSION[$key])) {
                $_SESSION[$key] = ['count' => 0, 'reset' => time() + 60];
            }
            
            if (time() > $_SESSION[$key]['reset']) {
                $_SESSION[$key] = ['count' => 0, 'reset' => time() + 60];
            }
            
            $_SESSION[$key]['count']++;
            
            if ($_SESSION[$key]['count'] > $limit) {
                http_response_code(429);
                return ['error' => 'Too many requests'];
            }
            
            return $next();
        };
    }
}

// 4. Event handling
class EventEmitter {
    private array $listeners = [];
    
    public function on(string $event, callable $listener): void {
        $this->listeners[$event][] = $listener;
    }
    
    public function emit(string $event, ...$args): void {
        if (isset($this->listeners[$event])) {
            foreach ($this->listeners[$event] as $listener) {
                call_user_func_array($listener, $args);
            }
        }
    }
}

// Usage
$emitter = new EventEmitter();

$emitter->on('user.created', function($user) {
    // Send welcome email
    mail($user['email'], 'Welcome!', 'Thanks for signing up!');
});

$emitter->on('user.created', function($user) {
    // Log activity
    error_log("New user created: {$user['email']}");
});

$emitter->emit('user.created', ['email' => 'john@example.com']);

// 5. Array transformation
class ArrayHelper {
    
    public static function map(array $array, callable $callback): array {
        return array_map($callback, $array);
    }
    
    public static function reduce(array $array, callable $callback, $initial = null): mixed {
        return array_reduce($array, $callback, $initial);
    }
    
    public static function pipe(...$functions): callable {
        return function($value) use ($functions) {
            foreach ($functions as $function) {
                $value = $function($value);
            }
            return $value;
        };
    }
}

// Usage
$numbers = [1, 2, 3, 4, 5];

// Map
$doubled = ArrayHelper::map($numbers, fn($n) => $n * 2);
// [2, 4, 6, 8, 10]

// Reduce
$sum = ArrayHelper::reduce($numbers, fn($carry, $n) => $carry + $n, 0);
// 15

// Pipe
$process = ArrayHelper::pipe(
    fn($n) => $n * 2,
    fn($n) => $n + 10,
    fn($n) => $n / 2
);

echo $process(5);  // (5 * 2 + 10) / 2 = 10
```

This covers core PHP concepts with practical examples. Would you like me to continue with the MySQL README?

