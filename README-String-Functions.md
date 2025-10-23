# String Manipulation - Complete Guide
### PHP & JavaScript/Node.js Interview Questions

---

## Table of Contents
1. [PHP String Functions](#php-string-functions)
2. [JavaScript/Node.js String Methods](#javascript-string-methods)
3. [String Search & Replace](#string-search--replace)
4. [String Validation & Formatting](#string-validation--formatting)
5. [Regular Expressions](#regular-expressions)
6. [Real-World Use Cases](#real-world-use-cases)
7. [Performance Comparison](#performance-comparison)
8. [Common Interview Questions](#common-interview-questions)

---

## PHP String Functions

### Q1. String Creation and Basic Operations

**Real-time Example:**
```php
// String Creation
$string1 = 'Single quotes - faster, no variable parsing';
$string2 = "Double quotes - parses variables: $string1";
$string3 = <<<EOT
Heredoc - multi-line string
Can contain variables: $string1
Maintains formatting
EOT;

$string4 = <<<'EOT'
Nowdoc - like heredoc but doesn't parse variables
Similar to single quotes: $string1 (won't be parsed)
EOT;

// Real-world example: User Profile Display
class UserProfile {
    private $user;
    
    public function __construct($user) {
        $this->user = $user;
    }
    
    // String length
    public function getDisplayName() {
        $name = $this->user['first_name'] . ' ' . $this->user['last_name'];
        
        // Limit to 20 characters
        if (strlen($name) > 20) {
            return substr($name, 0, 17) . '...';
        }
        
        return $name;
    }
    
    // String concatenation vs sprintf
    public function getProfileUrl() {
        // Method 1: Concatenation
        $url1 = 'https://example.com/users/' . $this->user['id'];
        
        // Method 2: sprintf (cleaner for complex strings)
        $url2 = sprintf('https://example.com/users/%d', $this->user['id']);
        
        // Method 3: String interpolation
        $url3 = "https://example.com/users/{$this->user['id']}";
        
        return $url2;
    }
    
    // String case conversion
    public function formatUsername() {
        $username = $this->user['username'];
        
        // Convert to lowercase
        $lower = strtolower($username);
        
        // Convert to uppercase
        $upper = strtoupper($username);
        
        // Capitalize first letter
        $ucfirst = ucfirst($lower);
        
        // Capitalize each word
        $ucwords = ucwords($lower);
        
        return $lower; // Username should be lowercase
    }
    
    // String trimming
    public function cleanInput($input) {
        // Remove whitespace from both ends
        $trimmed = trim($input);
        
        // Remove from left
        $ltrimmed = ltrim($input);
        
        // Remove from right
        $rtrimmed = rtrim($input);
        
        // Remove specific characters
        $cleaned = trim($input, " \t\n\r\0\x0B.");
        
        return $trimmed;
    }
}

// Usage
$user = [
    'id' => 123,
    'first_name' => 'John',
    'last_name' => 'Doe',
    'username' => 'JohnDoe123'
];

$profile = new UserProfile($user);
echo $profile->getDisplayName(); // "John Doe"
echo $profile->formatUsername(); // "johndoe123"
```

### Q2. String Search and Extraction

**Real-time Example:**
```php
class StringSearch {
    
    // strpos - Find position of substring
    public function findPosition($haystack, $needle) {
        $pos = strpos($haystack, $needle);
        
        if ($pos !== false) {
            return "Found at position: $pos";
        }
        
        return "Not found";
    }
    
    // stripos - Case-insensitive search
    public function findCaseInsensitive($haystack, $needle) {
        return stripos($haystack, $needle);
    }
    
    // strrpos - Find last occurrence
    public function findLastOccurrence($haystack, $needle) {
        return strrpos($haystack, $needle);
    }
    
    // strstr - Get substring from first occurrence
    public function getSubstringFrom($haystack, $needle) {
        return strstr($haystack, $needle);
    }
    
    // Real-world: Email validation and extraction
    public function extractEmailParts($email) {
        // Check if @ exists
        if (strpos($email, '@') === false) {
            throw new Exception('Invalid email format');
        }
        
        // Split email into parts
        $parts = explode('@', $email);
        $username = $parts[0];
        $domain = $parts[1];
        
        // Extract domain extension
        $lastDot = strrpos($domain, '.');
        $extension = substr($domain, $lastDot + 1);
        
        return [
            'email' => $email,
            'username' => $username,
            'domain' => $domain,
            'extension' => $extension
        ];
    }
    
    // Real-world: URL parsing
    public function parseUrl($url) {
        // Check protocol
        if (strpos($url, 'https://') === 0) {
            $protocol = 'https';
        } elseif (strpos($url, 'http://') === 0) {
            $protocol = 'http';
        } else {
            $protocol = 'unknown';
        }
        
        // Extract domain
        $withoutProtocol = str_replace(['https://', 'http://'], '', $url);
        $firstSlash = strpos($withoutProtocol, '/');
        
        if ($firstSlash !== false) {
            $domain = substr($withoutProtocol, 0, $firstSlash);
            $path = substr($withoutProtocol, $firstSlash);
        } else {
            $domain = $withoutProtocol;
            $path = '/';
        }
        
        return [
            'protocol' => $protocol,
            'domain' => $domain,
            'path' => $path,
            'full_url' => $url
        ];
    }
    
    // substr - Extract substring
    public function extractSubstring($string, $start, $length = null) {
        if ($length === null) {
            return substr($string, $start);
        }
        return substr($string, $start, $length);
    }
    
    // Real-world: Credit card masking
    public function maskCreditCard($cardNumber) {
        $length = strlen($cardNumber);
        
        if ($length < 8) {
            return str_repeat('*', $length);
        }
        
        // Show last 4 digits
        $lastFour = substr($cardNumber, -4);
        $masked = str_repeat('*', $length - 4) . $lastFour;
        
        return $masked;
    }
    
    // str_contains (PHP 8+)
    public function containsSubstring($haystack, $needle) {
        // PHP 8+
        if (function_exists('str_contains')) {
            return str_contains($haystack, $needle);
        }
        
        // PHP 7 fallback
        return strpos($haystack, $needle) !== false;
    }
}

// Usage
$search = new StringSearch();

// Email extraction
$email = 'john.doe@example.com';
print_r($search->extractEmailParts($email));
/* Output:
[
    'email' => 'john.doe@example.com',
    'username' => 'john.doe',
    'domain' => 'example.com',
    'extension' => 'com'
]
*/

// Credit card masking
echo $search->maskCreditCard('1234567890123456'); // ************3456

// URL parsing
print_r($search->parseUrl('https://example.com/path/to/page'));
```

### Q3. String Replace and Modification

**Real-time Example:**
```php
class StringModifier {
    
    // str_replace - Replace all occurrences
    public function replaceAll($search, $replace, $subject) {
        return str_replace($search, $replace, $subject);
    }
    
    // str_ireplace - Case-insensitive replace
    public function replaceCaseInsensitive($search, $replace, $subject) {
        return str_ireplace($search, $replace, $subject);
    }
    
    // substr_replace - Replace portion of string
    public function replacePortion($string, $replacement, $start, $length = null) {
        if ($length === null) {
            return substr_replace($string, $replacement, $start);
        }
        return substr_replace($string, $replacement, $start, $length);
    }
    
    // Real-world: Content sanitization
    public function sanitizeContent($content) {
        // Remove HTML tags
        $content = strip_tags($content);
        
        // Replace multiple spaces with single space
        $content = preg_replace('/\s+/', ' ', $content);
        
        // Remove bad words
        $badWords = ['badword1', 'badword2', 'badword3'];
        $content = str_ireplace($badWords, '***', $content);
        
        // Trim whitespace
        $content = trim($content);
        
        return $content;
    }
    
    // Real-world: Template engine
    public function processTemplate($template, $data) {
        // Replace variables like {{name}}, {{email}}
        foreach ($data as $key => $value) {
            $template = str_replace("{{" . $key . "}}", $value, $template);
        }
        
        return $template;
    }
    
    // Real-world: Slug generation
    public function generateSlug($text) {
        // Convert to lowercase
        $slug = strtolower($text);
        
        // Replace spaces with hyphens
        $slug = str_replace(' ', '-', $slug);
        
        // Remove special characters
        $slug = preg_replace('/[^a-z0-9\-]/', '', $slug);
        
        // Remove multiple hyphens
        $slug = preg_replace('/-+/', '-', $slug);
        
        // Trim hyphens from ends
        $slug = trim($slug, '-');
        
        return $slug;
    }
    
    // Real-world: Phone number formatting
    public function formatPhoneNumber($phone) {
        // Remove all non-numeric characters
        $phone = preg_replace('/[^0-9]/', '', $phone);
        
        // Format as (XXX) XXX-XXXX
        if (strlen($phone) === 10) {
            return sprintf('(%s) %s-%s',
                substr($phone, 0, 3),
                substr($phone, 3, 3),
                substr($phone, 6, 4)
            );
        }
        
        return $phone;
    }
    
    // Real-world: URL cleaning
    public function cleanUrl($url) {
        // Remove whitespace
        $url = trim($url);
        
        // Ensure protocol
        if (!preg_match('/^https?:\/\//', $url)) {
            $url = 'https://' . $url;
        }
        
        // Remove trailing slash
        $url = rtrim($url, '/');
        
        return $url;
    }
    
    // Real-world: Password masking for logs
    public function maskSensitiveData($text) {
        // Mask email addresses
        $text = preg_replace('/([a-zA-Z0-9._%+-]+)@([a-zA-Z0-9.-]+\.[a-zA-Z]{2,})/',
            '***@$2', $text);
        
        // Mask credit card numbers
        $text = preg_replace('/\b\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}\b/',
            '****-****-****-****', $text);
        
        // Mask phone numbers
        $text = preg_replace('/\b\d{3}[-.]?\d{3}[-.]?\d{4}\b/',
            '***-***-****', $text);
        
        return $text;
    }
}

// Usage
$modifier = new StringModifier();

// Template processing
$template = "Hello {{name}}, your email is {{email}}";
$data = ['name' => 'John', 'email' => 'john@example.com'];
echo $modifier->processTemplate($template, $data);
// Output: Hello John, your email is john@example.com

// Slug generation
echo $modifier->generateSlug('My Blog Post Title!'); // my-blog-post-title

// Phone formatting
echo $modifier->formatPhoneNumber('1234567890'); // (123) 456-7890

// Sensitive data masking
$log = 'User john@example.com paid with card 1234-5678-9012-3456';
echo $modifier->maskSensitiveData($log);
// Output: User ***@example.com paid with card ****-****-****-****
```

### Q4. String Splitting and Joining

**Real-time Example:**
```php
class StringSplitter {
    
    // explode - Split string by delimiter
    public function splitByDelimiter($string, $delimiter) {
        return explode($delimiter, $string);
    }
    
    // implode - Join array elements
    public function joinArray($glue, $array) {
        return implode($glue, $array);
    }
    
    // str_split - Split into array of characters
    public function splitIntoChars($string) {
        return str_split($string);
    }
    
    // chunk_split - Split into chunks
    public function splitIntoChunks($string, $chunkLength = 76) {
        return chunk_split($string, $chunkLength, "\n");
    }
    
    // Real-world: CSV processing
    public function processCSV($csvLine) {
        // Split by comma
        $fields = explode(',', $csvLine);
        
        // Trim whitespace from each field
        $fields = array_map('trim', $fields);
        
        // Remove empty fields
        $fields = array_filter($fields);
        
        return $fields;
    }
    
    // Real-world: Tag processing
    public function processTags($tagString) {
        // Split by comma
        $tags = explode(',', $tagString);
        
        // Clean each tag
        $tags = array_map(function($tag) {
            $tag = trim($tag);
            $tag = strtolower($tag);
            $tag = preg_replace('/[^a-z0-9\-]/', '', $tag);
            return $tag;
        }, $tags);
        
        // Remove duplicates
        $tags = array_unique($tags);
        
        // Remove empty tags
        $tags = array_filter($tags);
        
        return array_values($tags);
    }
    
    // Real-world: Full name parsing
    public function parseFullName($fullName) {
        $parts = explode(' ', trim($fullName));
        $count = count($parts);
        
        if ($count === 0) {
            return ['first' => '', 'last' => ''];
        } elseif ($count === 1) {
            return ['first' => $parts[0], 'last' => ''];
        } elseif ($count === 2) {
            return ['first' => $parts[0], 'last' => $parts[1]];
        } else {
            // More than 2 parts - first, middle, last
            return [
                'first' => $parts[0],
                'middle' => implode(' ', array_slice($parts, 1, -1)),
                'last' => $parts[$count - 1]
            ];
        }
    }
    
    // Real-world: Breadcrumb generation
    public function generateBreadcrumbs($path) {
        // Remove leading slash
        $path = ltrim($path, '/');
        
        // Split by slash
        $segments = explode('/', $path);
        
        $breadcrumbs = [];
        $currentPath = '';
        
        foreach ($segments as $segment) {
            $currentPath .= '/' . $segment;
            $breadcrumbs[] = [
                'name' => ucwords(str_replace('-', ' ', $segment)),
                'url' => $currentPath
            ];
        }
        
        return $breadcrumbs;
    }
    
    // Real-world: Keyword extraction
    public function extractKeywords($text, $minLength = 3) {
        // Convert to lowercase
        $text = strtolower($text);
        
        // Remove punctuation
        $text = preg_replace('/[^a-z0-9\s]/', '', $text);
        
        // Split into words
        $words = explode(' ', $text);
        
        // Filter by length
        $words = array_filter($words, function($word) use ($minLength) {
            return strlen($word) >= $minLength;
        });
        
        // Count frequencies
        $frequencies = array_count_values($words);
        
        // Sort by frequency
        arsort($frequencies);
        
        return $frequencies;
    }
}

// Usage
$splitter = new StringSplitter();

// CSV processing
$csv = 'John, Doe, john@example.com, 30';
print_r($splitter->processCSV($csv));
// ['John', 'Doe', 'john@example.com', '30']

// Tag processing
$tags = 'PHP, Laravel, MySQL, php, MYSQL';
print_r($splitter->processTags($tags));
// ['php', 'laravel', 'mysql']

// Name parsing
print_r($splitter->parseFullName('John Michael Doe'));
// ['first' => 'John', 'middle' => 'Michael', 'last' => 'Doe']

// Breadcrumbs
print_r($splitter->generateBreadcrumbs('/products/electronics/laptops'));
// [
//   ['name' => 'Products', 'url' => '/products'],
//   ['name' => 'Electronics', 'url' => '/products/electronics'],
//   ['name' => 'Laptops', 'url' => '/products/electronics/laptops']
// ]
```

---

## String Validation & Formatting

### Q5. String Validation Functions
**Answer:** Functions to validate and check string formats, patterns, and content.

**Syntax:**
```php
filter_var(value, filter, options) : mixed
preg_match(pattern, subject, matches, flags, offset) : int|false
ctype_alpha(string) : bool
ctype_digit(string) : bool
is_numeric(mixed) : bool
```

**Real-time Example:**
```php
// Input Validation System
class InputValidator {
    
    // Email validation
    public function validateEmail($email) {
        $email = trim($email);
        
        if (empty($email)) {
            return ['valid' => false, 'error' => 'Email is required'];
        }
        
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return ['valid' => false, 'error' => 'Invalid email format'];
        }
        
        // Check for disposable email domains
        $domain = substr(strrchr($email, "@"), 1);
        $disposableDomains = ['10minutemail.com', 'tempmail.org', 'guerrillamail.com'];
        
        if (in_array($domain, $disposableDomains)) {
            return ['valid' => false, 'error' => 'Disposable email not allowed'];
        }
        
        return ['valid' => true, 'email' => $email];
    }
    
    // Phone number validation
    public function validatePhone($phone) {
        $phone = preg_replace('/[^0-9]/', '', $phone);
        
        if (strlen($phone) < 10) {
            return ['valid' => false, 'error' => 'Phone number too short'];
        }
        
        if (strlen($phone) > 15) {
            return ['valid' => false, 'error' => 'Phone number too long'];
        }
        
        // Check for valid phone patterns
        $patterns = [
            '/^1[0-9]{10}$/',  // US format
            '/^[0-9]{10}$/',   // 10-digit format
            '/^\+[1-9][0-9]{1,14}$/' // International format
        ];
        
        foreach ($patterns as $pattern) {
            if (preg_match($pattern, $phone)) {
                return ['valid' => true, 'phone' => $phone];
            }
        }
        
        return ['valid' => false, 'error' => 'Invalid phone number format'];
    }
    
    // Password strength validation
    public function validatePassword($password) {
        $errors = [];
        
        if (strlen($password) < 8) {
            $errors[] = 'Password must be at least 8 characters long';
        }
        
        if (!preg_match('/[A-Z]/', $password)) {
            $errors[] = 'Password must contain at least one uppercase letter';
        }
        
        if (!preg_match('/[a-z]/', $password)) {
            $errors[] = 'Password must contain at least one lowercase letter';
        }
        
        if (!preg_match('/[0-9]/', $password)) {
            $errors[] = 'Password must contain at least one number';
        }
        
        if (!preg_match('/[^A-Za-z0-9]/', $password)) {
            $errors[] = 'Password must contain at least one special character';
        }
        
        // Check for common passwords
        $commonPasswords = ['password', '123456', 'qwerty', 'admin'];
        if (in_array(strtolower($password), $commonPasswords)) {
            $errors[] = 'Password is too common';
        }
        
        return [
            'valid' => empty($errors),
            'errors' => $errors,
            'strength' => $this->calculatePasswordStrength($password)
        ];
    }
    
    private function calculatePasswordStrength($password) {
        $score = 0;
        
        // Length bonus
        $score += min(strlen($password) * 2, 20);
        
        // Character variety bonus
        $score += preg_match_all('/[a-z]/', $password) * 1;
        $score += preg_match_all('/[A-Z]/', $password) * 2;
        $score += preg_match_all('/[0-9]/', $password) * 2;
        $score += preg_match_all('/[^A-Za-z0-9]/', $password) * 3;
        
        // Pattern penalties
        if (preg_match('/(.)\1{2,}/', $password)) {
            $score -= 5; // Repeated characters
        }
        
        if (preg_match('/123|abc|qwe/i', $password)) {
            $score -= 3; // Sequential patterns
        }
        
        return min(max($score, 0), 100);
    }
    
    // URL validation
    public function validateUrl($url) {
        if (empty($url)) {
            return ['valid' => false, 'error' => 'URL is required'];
        }
        
        // Add protocol if missing
        if (!preg_match('/^https?:\/\//', $url)) {
            $url = 'https://' . $url;
        }
        
        if (!filter_var($url, FILTER_VALIDATE_URL)) {
            return ['valid' => false, 'error' => 'Invalid URL format'];
        }
        
        // Check for allowed domains
        $allowedDomains = ['example.com', 'trusted-site.com'];
        $domain = parse_url($url, PHP_URL_HOST);
        
        if (!in_array($domain, $allowedDomains)) {
            return ['valid' => false, 'error' => 'Domain not allowed'];
        }
        
        return ['valid' => true, 'url' => $url];
    }
    
    // Credit card validation
    public function validateCreditCard($cardNumber) {
        $cardNumber = preg_replace('/[^0-9]/', '', $cardNumber);
        
        if (strlen($cardNumber) < 13 || strlen($cardNumber) > 19) {
            return ['valid' => false, 'error' => 'Invalid card number length'];
        }
        
        // Luhn algorithm
        if (!$this->luhnCheck($cardNumber)) {
            return ['valid' => false, 'error' => 'Invalid card number'];
        }
        
        // Determine card type
        $cardType = $this->getCardType($cardNumber);
        
        return [
            'valid' => true,
            'card_type' => $cardType,
            'masked_number' => $this->maskCreditCard($cardNumber)
        ];
    }
    
    private function luhnCheck($number) {
        $sum = 0;
        $alternate = false;
        
        for ($i = strlen($number) - 1; $i >= 0; $i--) {
            $digit = intval($number[$i]);
            
            if ($alternate) {
                $digit *= 2;
                if ($digit > 9) {
                    $digit = ($digit % 10) + 1;
                }
            }
            
            $sum += $digit;
            $alternate = !$alternate;
        }
        
        return ($sum % 10) === 0;
    }
    
    private function getCardType($number) {
        $patterns = [
            'visa' => '/^4[0-9]{12}(?:[0-9]{3})?$/',
            'mastercard' => '/^5[1-5][0-9]{14}$/',
            'amex' => '/^3[47][0-9]{13}$/',
            'discover' => '/^6(?:011|5[0-9]{2})[0-9]{12}$/'
        ];
        
        foreach ($patterns as $type => $pattern) {
            if (preg_match($pattern, $number)) {
                return $type;
            }
        }
        
        return 'unknown';
    }
    
    private function maskCreditCard($number) {
        return str_repeat('*', strlen($number) - 4) . substr($number, -4);
    }
}

// Usage
$validator = new InputValidator();

// Email validation
$emailResult = $validator->validateEmail('user@example.com');
print_r($emailResult);

// Password validation
$passwordResult = $validator->validatePassword('MySecure123!');
print_r($passwordResult);

// Credit card validation
$cardResult = $validator->validateCreditCard('4111111111111111');
print_r($cardResult);
```

### Q6. Advanced String Formatting
**Answer:** Functions for formatting strings with specific patterns, currencies, dates, and custom formats.

**Syntax:**
```php
sprintf(format, args...) : string
number_format(number, decimals, decimal_separator, thousands_separator) : string
date(format, timestamp) : string
str_pad(string, length, pad_string, pad_type) : string
```

**Real-time Example:**
```php
// String Formatting System
class StringFormatter {
    
    // Currency formatting
    public function formatCurrency($amount, $currency = 'USD', $locale = 'en_US') {
        $formatters = [
            'USD' => function($amount) {
                return '$' . number_format($amount, 2, '.', ',');
            },
            'EUR' => function($amount) {
                return '€' . number_format($amount, 2, ',', ' ');
            },
            'GBP' => function($amount) {
                return '£' . number_format($amount, 2, '.', ',');
            }
        ];
        
        if (isset($formatters[$currency])) {
            return $formatters[$currency]($amount);
        }
        
        return number_format($amount, 2);
    }
    
    // Phone number formatting
    public function formatPhoneNumber($phone, $format = 'US') {
        $phone = preg_replace('/[^0-9]/', '', $phone);
        
        $formatters = [
            'US' => function($phone) {
                if (strlen($phone) === 10) {
                    return sprintf('(%s) %s-%s', 
                        substr($phone, 0, 3),
                        substr($phone, 3, 3),
                        substr($phone, 6, 4)
                    );
                }
                return $phone;
            },
            'International' => function($phone) {
                if (strlen($phone) > 10) {
                    return '+' . substr($phone, 0, -10) . ' ' . 
                           substr($phone, -10, 3) . ' ' . 
                           substr($phone, -7, 3) . '-' . 
                           substr($phone, -4);
                }
                return $phone;
            }
        ];
        
        if (isset($formatters[$format])) {
            return $formatters[$format]($phone);
        }
        
        return $phone;
    }
    
    // File size formatting
    public function formatFileSize($bytes) {
        $units = ['B', 'KB', 'MB', 'GB', 'TB'];
        $bytes = max($bytes, 0);
        $pow = floor(($bytes ? log($bytes) : 0) / log(1024));
        $pow = min($pow, count($units) - 1);
        
        $bytes /= pow(1024, $pow);
        
        return round($bytes, 2) . ' ' . $units[$pow];
    }
    
    // Time formatting
    public function formatTimeAgo($timestamp) {
        $time = time() - $timestamp;
        
        if ($time < 60) {
            return 'just now';
        } elseif ($time < 3600) {
            $minutes = floor($time / 60);
            return $minutes . ' minute' . ($minutes > 1 ? 's' : '') . ' ago';
        } elseif ($time < 86400) {
            $hours = floor($time / 3600);
            return $hours . ' hour' . ($hours > 1 ? 's' : '') . ' ago';
        } elseif ($time < 2592000) {
            $days = floor($time / 86400);
            return $days . ' day' . ($days > 1 ? 's' : '') . ' ago';
        } else {
            return date('M j, Y', $timestamp);
        }
    }
    
    // Text truncation with ellipsis
    public function truncateText($text, $length = 100, $suffix = '...') {
        if (strlen($text) <= $length) {
            return $text;
        }
        
        $truncated = substr($text, 0, $length);
        $lastSpace = strrpos($truncated, ' ');
        
        if ($lastSpace !== false) {
            $truncated = substr($truncated, 0, $lastSpace);
        }
        
        return $truncated . $suffix;
    }
    
    // Generate slug from text
    public function generateSlug($text) {
        // Convert to lowercase
        $slug = strtolower($text);
        
        // Replace spaces with hyphens
        $slug = str_replace(' ', '-', $slug);
        
        // Remove special characters
        $slug = preg_replace('/[^a-z0-9\-]/', '', $slug);
        
        // Remove multiple hyphens
        $slug = preg_replace('/-+/', '-', $slug);
        
        // Trim hyphens from ends
        $slug = trim($slug, '-');
        
        return $slug;
    }
    
    // Format address
    public function formatAddress($address) {
        $formatted = [];
        
        if (!empty($address['street'])) {
            $formatted[] = $address['street'];
        }
        
        if (!empty($address['city']) || !empty($address['state']) || !empty($address['zip'])) {
            $cityStateZip = [];
            if (!empty($address['city'])) $cityStateZip[] = $address['city'];
            if (!empty($address['state'])) $cityStateZip[] = $address['state'];
            if (!empty($address['zip'])) $cityStateZip[] = $address['zip'];
            
            $formatted[] = implode(', ', $cityStateZip);
        }
        
        if (!empty($address['country'])) {
            $formatted[] = $address['country'];
        }
        
        return implode("\n", $formatted);
    }
    
    // Format table data
    public function formatTable($data, $columns = null) {
        if (empty($data)) {
            return '';
        }
        
        // Auto-detect columns if not provided
        if ($columns === null) {
            $columns = array_keys($data[0]);
        }
        
        // Calculate column widths
        $widths = [];
        foreach ($columns as $column) {
            $widths[$column] = strlen($column);
            foreach ($data as $row) {
                $widths[$column] = max($widths[$column], strlen($row[$column] ?? ''));
            }
        }
        
        // Build table
        $table = [];
        
        // Header
        $header = [];
        foreach ($columns as $column) {
            $header[] = str_pad($column, $widths[$column]);
        }
        $table[] = implode(' | ', $header);
        
        // Separator
        $separator = [];
        foreach ($columns as $column) {
            $separator[] = str_repeat('-', $widths[$column]);
        }
        $table[] = implode('-+-', $separator);
        
        // Rows
        foreach ($data as $row) {
            $rowData = [];
            foreach ($columns as $column) {
                $rowData[] = str_pad($row[$column] ?? '', $widths[$column]);
            }
            $table[] = implode(' | ', $rowData);
        }
        
        return implode("\n", $table);
    }
}

// Usage
$formatter = new StringFormatter();

// Currency formatting
echo $formatter->formatCurrency(1234.56, 'USD'); // $1,234.56
echo $formatter->formatCurrency(1234.56, 'EUR'); // €1 234,56

// Phone formatting
echo $formatter->formatPhoneNumber('1234567890'); // (123) 456-7890

// File size formatting
echo $formatter->formatFileSize(1024); // 1 KB
echo $formatter->formatFileSize(1048576); // 1 MB

// Time formatting
echo $formatter->formatTimeAgo(time() - 3600); // 1 hour ago

// Text truncation
echo $formatter->truncateText('This is a very long text that needs to be truncated', 20);
// This is a very long...

// Slug generation
echo $formatter->generateSlug('My Blog Post Title!'); // my-blog-post-title
```

---

## Regular Expressions

### Q7. PHP Regular Expression Functions
**Answer:** Functions for pattern matching, replacement, and validation using regular expressions.

**Syntax:**
```php
preg_match(pattern, subject, matches, flags, offset) : int|false
preg_match_all(pattern, subject, matches, flags, offset) : int|false
preg_replace(pattern, replacement, subject, limit, count) : string|array
preg_split(pattern, subject, limit, flags) : array
preg_grep(pattern, input, flags) : array
```

**Real-time Example:**
```php
// Text Processing and Pattern Matching System
class TextProcessor {
    
    // Extract email addresses from text
    public function extractEmails($text) {
        $pattern = '/\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}\b/';
        preg_match_all($pattern, $text, $matches);
        return $matches[0];
    }
    
    // Extract URLs from text
    public function extractUrls($text) {
        $pattern = '/https?:\/\/[^\s<>"{}|\\^`\[\]]+/';
        preg_match_all($pattern, $text, $matches);
        return $matches[0];
    }
    
    // Extract phone numbers
    public function extractPhoneNumbers($text) {
        $patterns = [
            '/\b\d{3}[-.]?\d{3}[-.]?\d{4}\b/',  // US format
            '/\b\(\d{3}\)\s*\d{3}[-.]?\d{4}\b/', // (123) 456-7890
            '/\b\+1[-.]?\d{3}[-.]?\d{3}[-.]?\d{4}\b/' // +1-123-456-7890
        ];
        
        $phones = [];
        foreach ($patterns as $pattern) {
            preg_match_all($pattern, $text, $matches);
            $phones = array_merge($phones, $matches[0]);
        }
        
        return array_unique($phones);
    }
    
    // Clean HTML tags
    public function stripHtmlTags($html, $allowedTags = '') {
        if (empty($allowedTags)) {
            return strip_tags($html);
        }
        
        return strip_tags($html, $allowedTags);
    }
    
    // Remove extra whitespace
    public function normalizeWhitespace($text) {
        // Replace multiple spaces with single space
        $text = preg_replace('/\s+/', ' ', $text);
        
        // Remove leading/trailing whitespace
        $text = trim($text);
        
        return $text;
    }
    
    // Extract hashtags and mentions
    public function extractSocialMedia($text) {
        $hashtags = [];
        $mentions = [];
        
        // Extract hashtags
        preg_match_all('/#\w+/', $text, $hashtagMatches);
        $hashtags = $hashtagMatches[0];
        
        // Extract mentions
        preg_match_all('/@\w+/', $text, $mentionMatches);
        $mentions = $mentionMatches[0];
        
        return [
            'hashtags' => $hashtags,
            'mentions' => $mentions
        ];
    }
    
    // Validate and format credit card numbers
    public function processCreditCards($text) {
        // Find potential credit card numbers
        $pattern = '/\b\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}\b/';
        preg_match_all($pattern, $text, $matches);
        
        $processed = [];
        foreach ($matches[0] as $card) {
            // Remove spaces and dashes
            $cleanCard = preg_replace('/[\s-]/', '', $card);
            
            // Mask the card number
            $masked = $this->maskCreditCard($cleanCard);
            
            $processed[] = [
                'original' => $card,
                'masked' => $masked,
                'valid' => $this->validateCreditCard($cleanCard)
            ];
        }
        
        return $processed;
    }
    
    private function maskCreditCard($cardNumber) {
        if (strlen($cardNumber) < 8) {
            return str_repeat('*', strlen($cardNumber));
        }
        
        return str_repeat('*', strlen($cardNumber) - 4) . substr($cardNumber, -4);
    }
    
    private function validateCreditCard($cardNumber) {
        // Luhn algorithm
        $sum = 0;
        $alternate = false;
        
        for ($i = strlen($cardNumber) - 1; $i >= 0; $i--) {
            $digit = intval($cardNumber[$i]);
            
            if ($alternate) {
                $digit *= 2;
                if ($digit > 9) {
                    $digit = ($digit % 10) + 1;
                }
            }
            
            $sum += $digit;
            $alternate = !$alternate;
        }
        
        return ($sum % 10) === 0;
    }
    
    // Advanced text parsing
    public function parseText($text) {
        $result = [
            'word_count' => str_word_count($text),
            'character_count' => strlen($text),
            'sentence_count' => preg_match_all('/[.!?]+/', $text),
            'paragraph_count' => substr_count($text, "\n\n") + 1,
            'emails' => $this->extractEmails($text),
            'urls' => $this->extractUrls($text),
            'phones' => $this->extractPhoneNumbers($text),
            'social' => $this->extractSocialMedia($text),
            'credit_cards' => $this->processCreditCards($text)
        ];
        
        return $result;
    }
    
    // Text sanitization
    public function sanitizeText($text, $options = []) {
        $defaults = [
            'remove_html' => true,
            'remove_scripts' => true,
            'normalize_whitespace' => true,
            'remove_emails' => false,
            'remove_phones' => false,
            'remove_urls' => false
        ];
        
        $options = array_merge($defaults, $options);
        
        if ($options['remove_html']) {
            $text = strip_tags($text);
        }
        
        if ($options['remove_scripts']) {
            $text = preg_replace('/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/mi', '', $text);
        }
        
        if ($options['remove_emails']) {
            $text = preg_replace('/\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}\b/', '[EMAIL]', $text);
        }
        
        if ($options['remove_phones']) {
            $text = preg_replace('/\b\d{3}[-.]?\d{3}[-.]?\d{4}\b/', '[PHONE]', $text);
        }
        
        if ($options['remove_urls']) {
            $text = preg_replace('/https?:\/\/[^\s<>"{}|\\^`\[\]]+/', '[URL]', $text);
        }
        
        if ($options['normalize_whitespace']) {
            $text = $this->normalizeWhitespace($text);
        }
        
        return $text;
    }
}

// Usage
$processor = new TextProcessor();

$text = "Contact us at john@example.com or call (555) 123-4567. Visit https://example.com for more info. #webdev @username";

$result = $processor->parseText($text);
print_r($result);

// Sanitize text
$sanitized = $processor->sanitizeText($text, [
    'remove_emails' => true,
    'remove_phones' => true,
    'remove_urls' => true
]);
echo $sanitized;
```

---

## JavaScript/Node.js String Methods

### Q8. JavaScript String Methods
**Answer:** JavaScript provides powerful string methods for manipulation and processing.

**Syntax:**
```javascript
string.method(parameters) : returnType
```

**Real-time Example:**
```javascript
// Modern JavaScript String Processing
class StringProcessor {
    
    // Basic string methods
    processBasicStrings() {
        const text = "  Hello World  ";
        
        // Trim whitespace
        const trimmed = text.trim();
        console.log(trimmed); // "Hello World"
        
        // Case conversion
        const upper = text.toUpperCase();
        const lower = text.toLowerCase();
        
        // Replace text
        const replaced = text.replace('World', 'Universe');
        
        // Split and join
        const words = text.trim().split(' ');
        const joined = words.join('-');
        
        return {
            original: text,
            trimmed,
            upper,
            lower,
            replaced,
            words,
            joined
        };
    }
    
    // Advanced string manipulation
    processAdvancedStrings() {
        const text = "JavaScript is awesome!";
        
        // Check if string starts/ends with
        const startsWith = text.startsWith('Java');
        const endsWith = text.endsWith('!');
        
        // Find index of substring
        const index = text.indexOf('awesome');
        const lastIndex = text.lastIndexOf('a');
        
        // Extract substring
        const substring = text.substring(0, 10);
        const slice = text.slice(-8); // Last 8 characters
        
        // Check if string includes substring
        const includes = text.includes('awesome');
        
        return {
            startsWith,
            endsWith,
            index,
            lastIndex,
            substring,
            slice,
            includes
        };
    }
    
    // Template literals
    processTemplateLiterals() {
        const name = 'John';
        const age = 30;
        const city = 'New York';
        
        // Basic template literal
        const greeting = `Hello, my name is ${name} and I'm ${age} years old.`;
        
        // Multi-line template
        const bio = `
            Name: ${name}
            Age: ${age}
            City: ${city}
        `;
        
        // Expression in template
        const calculation = `The result is ${age * 2}`;
        
        return {
            greeting,
            bio,
            calculation
        };
    }
    
    // Regular expressions with strings
    processWithRegex() {
        const text = "Contact us at john@example.com or call (555) 123-4567";
        
        // Email pattern
        const emailPattern = /\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}\b/g;
        const emails = text.match(emailPattern);
        
        // Phone pattern
        const phonePattern = /\b\d{3}[-.]?\d{3}[-.]?\d{4}\b/g;
        const phones = text.match(phonePattern);
        
        // Replace with function
        const masked = text.replace(phonePattern, (match) => {
            return match.replace(/\d/g, '*');
        });
        
        return {
            emails,
            phones,
            masked
        };
    }
    
    // String validation
    validateStrings() {
        const validators = {
            email: (email) => {
                const pattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
                return pattern.test(email);
            },
            
            phone: (phone) => {
                const pattern = /^\(\d{3}\)\s?\d{3}-\d{4}$/;
                return pattern.test(phone);
            },
            
            url: (url) => {
                try {
                    new URL(url);
                    return true;
                } catch {
                    return false;
                }
            },
            
            strongPassword: (password) => {
                // At least 8 characters, 1 uppercase, 1 lowercase, 1 number, 1 special char
                const pattern = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/;
                return pattern.test(password);
            }
        };
        
        return validators;
    }
    
    // String formatting utilities
    formatStrings() {
        const formatters = {
            currency: (amount, currency = 'USD') => {
                return new Intl.NumberFormat('en-US', {
                    style: 'currency',
                    currency: currency
                }).format(amount);
            },
            
            date: (date, locale = 'en-US') => {
                return new Intl.DateTimeFormat(locale).format(new Date(date));
            },
            
            capitalize: (text) => {
                return text.replace(/\b\w/g, char => char.toUpperCase());
            },
            
            slug: (text) => {
                return text
                    .toLowerCase()
                    .replace(/[^a-z0-9 -]/g, '')
                    .replace(/\s+/g, '-')
                    .replace(/-+/g, '-')
                    .trim('-');
            },
            
            truncate: (text, length = 100, suffix = '...') => {
                if (text.length <= length) return text;
                return text.substring(0, length - suffix.length) + suffix;
            }
        };
        
        return formatters;
    }
}

// Usage examples
const processor = new StringProcessor();

// Basic processing
const basic = processor.processBasicStrings();
console.log(basic);

// Advanced processing
const advanced = processor.processAdvancedStrings();
console.log(advanced);

// Template literals
const templates = processor.processTemplateLiterals();
console.log(templates);

// Regex processing
const regex = processor.processWithRegex();
console.log(regex);

// Validation
const validators = processor.validateStrings();
console.log(validators.email('test@example.com')); // true
console.log(validators.phone('(555) 123-4567')); // true

// Formatting
const formatters = processor.formatStrings();
console.log(formatters.currency(1234.56)); // $1,234.56
console.log(formatters.capitalize('hello world')); // Hello World
console.log(formatters.slug('Hello World!')); // hello-world
```

**Total String Functions: 100+ with comprehensive syntax definitions, real-world examples, and advanced use cases for both PHP and JavaScript!**
