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

This is a comprehensive start to the string functions guide. Would you like me to continue with JavaScript/Node.js string methods, Regular Expressions, and more real-world examples?
