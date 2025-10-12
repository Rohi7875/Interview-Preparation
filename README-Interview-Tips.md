# Complete Interview Preparation Strategy
### Tips, Tricks & Common Questions for 7+ Years Experience

---

## Table of Contents
1. [Behavioral Interview Questions](#behavioral-interview-questions)
2. [System Design Questions](#system-design-questions)
3. [Coding Challenges](#coding-challenges)
4. [Database Design Questions](#database-design-questions)
5. [API Design Questions](#api-design-questions)
6. [Architecture & Scalability](#architecture--scalability)
7. [Common Mistakes to Avoid](#common-mistakes-to-avoid)
8. [Salary Negotiation Tips](#salary-negotiation-tips)
9. [Final Week Preparation](#final-week-preparation)
10. [Day of Interview Checklist](#day-of-interview-checklist)

---

## Behavioral Interview Questions

### Q1. Tell me about yourself
**How to Answer:**
- Keep it under 2 minutes
- Focus on professional journey
- Highlight relevant experiences
- End with why you're interested in this role

**Example Answer:**
```
"I'm a full-stack developer with over 7 years of experience building scalable 
web applications. I started my career at [Company A] where I worked on 
e-commerce platforms using PHP and Laravel. I led a team of 4 developers and 
we successfully migrated a legacy system to microservices, which improved 
performance by 60%.

Currently at [Company B], I've been focusing on Node.js and MongoDB, building 
RESTful APIs that handle millions of requests daily. I'm particularly proud 
of implementing a caching strategy that reduced database load by 40%.

I'm excited about this opportunity because [specific reason about the company/role] 
and I believe my experience in [relevant skills] would be valuable to your team."
```

### Q2. What's your biggest weakness?
**How to Answer:**
- Be honest but strategic
- Show self-awareness
- Explain how you're improving
- Make it a learning story

**Example Answer:**
```
"Earlier in my career, I tended to focus too much on perfect code rather than 
delivering features quickly. I would spend hours optimizing code that didn't 
need optimization yet.

I've learned to follow the principle of 'make it work, make it right, make it 
fast' - in that order. Now I focus on delivering working features first, then 
refactoring based on actual performance data. This has made me much more 
productive and better at prioritizing."
```

### Q3. Describe a challenging project you worked on
**Framework: STAR Method**
- **S**ituation - Set the context
- **T**ask - What was your responsibility
- **A**ction - What you did
- **R**esult - Outcome with numbers

**Example Answer:**
```
Situation:
"At my previous company, our main application was struggling with performance 
issues during peak traffic. Page load times exceeded 5 seconds and we were 
losing customers."

Task:
"As the lead developer, I was tasked with identifying and fixing the 
performance bottlenecks within 2 months."

Action:
"I conducted a thorough performance audit using New Relic and identified three 
main issues:
1. N+1 database queries in our product listings
2. No caching layer for frequently accessed data
3. Unoptimized images and assets

I implemented eager loading for relationships, set up Redis caching, and 
integrated a CDN for static assets. I also created comprehensive performance 
tests to prevent regression."

Result:
"We reduced page load time from 5 seconds to under 1 second - an 80% improvement. 
Customer satisfaction increased by 35%, and checkout completion rate improved 
by 25%. The solution handled 3x more traffic during Black Friday sales."
```

### Q4. Why are you leaving your current job?
**Good Answers:**
- Seeking new challenges
- Want to work with specific technologies
- Looking for growth opportunities
- Interested in company's mission/products

**Avoid:**
- Badmouthing current employer
- Money being only reason
- Personality conflicts
- Being too negative

**Example Answer:**
```
"I've had a great experience at my current company and learned a lot, 
especially in [specific areas]. However, I'm looking for new challenges 
that will help me grow. 

Specifically, I'm excited about [Company's] focus on [specific technology/domain]. 
I've been studying [relevant tech] in my personal time, and I'm eager to 
work on projects at scale like [mention their products/services].

I believe this role would be a perfect next step in my career path toward 
becoming a [future goal, e.g., technical lead, architect]."
```

### Q5. Where do you see yourself in 5 years?
**Example Answer:**
```
"In 5 years, I see myself as a technical leader who not only writes code but 
also mentors junior developers and influences architectural decisions.

I'm particularly interested in [specific area like microservices, cloud architecture, 
etc.] and would love to become an expert in that domain. I also want to 
contribute to open source projects and perhaps speak at conferences to share 
knowledge with the community.

Ideally, I'd like to be in a position where I'm solving complex technical 
challenges while also helping shape the technical direction of the team."
```

---

## System Design Questions

### Q6. Design a URL Shortener (like bit.ly)

**Requirements:**
- Shorten long URLs to short ones
- Redirect short URLs to original
- Track click statistics
- Handle millions of requests

**Solution:**

```
1. Database Schema:
┌─────────────────────────────────────────┐
│ urls table                              │
├─────────────────────────────────────────┤
│ id            BIGINT PRIMARY KEY        │
│ short_code    VARCHAR(10) UNIQUE        │
│ original_url  TEXT                      │
│ user_id       BIGINT                    │
│ created_at    TIMESTAMP                 │
│ expires_at    TIMESTAMP                 │
│ click_count   INT DEFAULT 0             │
└─────────────────────────────────────────┘

2. Short Code Generation:
- Use Base62 encoding (a-z, A-Z, 0-9)
- ID 1000 → "G8" (62^2 combinations)
- 6 characters → 56+ billion combinations

3. Architecture:

   ┌─────────────┐
   │   Client    │
   └──────┬──────┘
          │
   ┌──────▼──────────────┐
   │   Load Balancer     │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐
   │   API Servers       │
   │  (Stateless)        │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐     ┌──────────────┐
   │   Redis Cache       │────▶│   Database   │
   │  (Read-through)     │     │  (Master)    │
   └─────────────────────┘     └──────┬───────┘
                                      │
                               ┌──────▼───────┐
                               │   Replicas   │
                               └──────────────┘

4. API Endpoints:

POST /api/shorten
{
  "url": "https://example.com/very/long/url",
  "custom_code": "optional",
  "expires_in": 3600
}
Response: {"short_url": "bit.ly/abc123"}

GET /{short_code}
- Redirect to original URL
- Increment click count (async)

5. Scaling Considerations:
- Cache hot URLs in Redis
- Use CDN for redirects
- Async analytics processing
- Database sharding by hash of short_code
- Rate limiting per user/IP
```

**Code Implementation:**
```php
class URLShortener {
    private $db;
    private $cache;
    
    private const BASE62 = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
    
    public function shorten($url, $userId = null, $customCode = null) {
        // Validate URL
        if (!filter_var($url, FILTER_VALIDATE_URL)) {
            throw new Exception('Invalid URL');
        }
        
        // Check if already shortened
        $existing = $this->db->query(
            "SELECT short_code FROM urls WHERE original_url = ? AND user_id = ?",
            [$url, $userId]
        )->fetch();
        
        if ($existing) {
            return $this->getShortURL($existing['short_code']);
        }
        
        // Generate or use custom code
        $shortCode = $customCode ?: $this->generateShortCode();
        
        // Store in database
        $this->db->insert('urls', [
            'short_code' => $shortCode,
            'original_url' => $url,
            'user_id' => $userId,
            'created_at' => date('Y-m-d H:i:s')
        ]);
        
        return $this->getShortURL($shortCode);
    }
    
    public function redirect($shortCode) {
        // Try cache first
        $url = $this->cache->get("url:$shortCode");
        
        if (!$url) {
            // Get from database
            $result = $this->db->query(
                "SELECT original_url FROM urls WHERE short_code = ?",
                [$shortCode]
            )->fetch();
            
            if (!$result) {
                throw new Exception('URL not found');
            }
            
            $url = $result['original_url'];
            
            // Cache for 1 hour
            $this->cache->set("url:$shortCode", $url, 3600);
        }
        
        // Increment click count asynchronously
        $this->incrementClickCount($shortCode);
        
        return $url;
    }
    
    private function generateShortCode() {
        $id = $this->db->insert('urls', [
            'original_url' => '',
            'created_at' => date('Y-m-d H:i:s')
        ]);
        
        return $this->base62Encode($id);
    }
    
    private function base62Encode($num) {
        $base = strlen(self::BASE62);
        $encoded = '';
        
        while ($num > 0) {
            $encoded = self::BASE62[$num % $base] . $encoded;
            $num = floor($num / $base);
        }
        
        return $encoded ?: '0';
    }
    
    private function incrementClickCount($shortCode) {
        // Use queue for async processing
        Queue::push('IncrementClickJob', ['short_code' => $shortCode]);
    }
    
    private function getShortURL($shortCode) {
        return "https://short.url/$shortCode";
    }
}
```

### Q7. Design a Rate Limiter

**Requirements:**
- Limit API requests per user
- Multiple rate limit tiers
- Distributed system support

**Solution:**

```
1. Algorithm Options:

a) Token Bucket:
- Bucket has capacity (max requests)
- Refills at constant rate
- Request consumes token

b) Sliding Window Log:
- Track timestamp of each request
- Remove old entries
- Count recent requests

c) Fixed Window Counter:
- Count requests per time window
- Reset counter at window boundary

2. Implementation (Token Bucket with Redis):

┌─────────────────────────────────────────┐
│ Rate Limit Key Structure               │
├─────────────────────────────────────────┤
│ rate_limit:user:123                     │
│ {                                       │
│   "tokens": 95,                         │
│   "last_refill": 1634567890             │
│ }                                       │
└─────────────────────────────────────────┘

3. Rate Limit Tiers:
- Free: 100 requests/hour
- Basic: 1,000 requests/hour
- Premium: 10,000 requests/hour
- Enterprise: Unlimited
```

**Code Implementation:**
```javascript
class RateLimiter {
    constructor(redis) {
        this.redis = redis;
        this.limits = {
            free: { requests: 100, window: 3600 },
            basic: { requests: 1000, window: 3600 },
            premium: { requests: 10000, window: 3600 }
        };
    }
    
    async isAllowed(userId, tier = 'free') {
        const key = `rate_limit:${userId}`;
        const limit = this.limits[tier];
        const now = Date.now();
        
        // Get current state
        const data = await this.redis.get(key);
        let state = data ? JSON.parse(data) : null;
        
        if (!state) {
            // First request
            state = {
                tokens: limit.requests - 1,
                lastRefill: now
            };
            
            await this.redis.setex(
                key,
                limit.window,
                JSON.stringify(state)
            );
            
            return {
                allowed: true,
                remaining: state.tokens,
                resetAt: now + (limit.window * 1000)
            };
        }
        
        // Refill tokens
        const timePassed = (now - state.lastRefill) / 1000;
        const refillRate = limit.requests / limit.window;
        const tokensToAdd = Math.floor(timePassed * refillRate);
        
        state.tokens = Math.min(
            limit.requests,
            state.tokens + tokensToAdd
        );
        state.lastRefill = now;
        
        // Check if request allowed
        if (state.tokens > 0) {
            state.tokens--;
            await this.redis.setex(
                key,
                limit.window,
                JSON.stringify(state)
            );
            
            return {
                allowed: true,
                remaining: state.tokens,
                resetAt: now + (limit.window * 1000)
            };
        }
        
        return {
            allowed: false,
            remaining: 0,
            resetAt: now + (limit.window * 1000)
        };
    }
    
    // Sliding window implementation
    async isAllowedSlidingWindow(userId, tier = 'free') {
        const key = `rate_limit_log:${userId}`;
        const limit = this.limits[tier];
        const now = Date.now();
        const windowStart = now - (limit.window * 1000);
        
        // Add current request
        await this.redis.zadd(key, now, `${now}-${Math.random()}`);
        
        // Remove old entries
        await this.redis.zremrangebyscore(key, 0, windowStart);
        
        // Count requests in window
        const count = await this.redis.zcard(key);
        
        // Set expiry
        await this.redis.expire(key, limit.window);
        
        return {
            allowed: count <= limit.requests,
            remaining: Math.max(0, limit.requests - count),
            resetAt: windowStart + (limit.window * 1000)
        };
    }
}

// Express middleware
function rateLimitMiddleware(redis) {
    const limiter = new RateLimiter(redis);
    
    return async (req, res, next) => {
        const userId = req.user?.id || req.ip;
        const tier = req.user?.tier || 'free';
        
        try {
            const result = await limiter.isAllowed(userId, tier);
            
            // Set rate limit headers
            res.setHeader('X-RateLimit-Limit', limiter.limits[tier].requests);
            res.setHeader('X-RateLimit-Remaining', result.remaining);
            res.setHeader('X-RateLimit-Reset', result.resetAt);
            
            if (!result.allowed) {
                return res.status(429).json({
                    error: 'Rate limit exceeded',
                    retryAfter: Math.ceil((result.resetAt - Date.now()) / 1000)
                });
            }
            
            next();
        } catch (error) {
            console.error('Rate limiter error:', error);
            next(); // Fail open
        }
    };
}
```

---

## Coding Challenges

### Q8. Common Coding Problems

**Problem 1: Find Duplicate Orders**
```php
/**
 * Given an array of orders, find all duplicate orders
 * based on customer_id and product_id combination
 */
function findDuplicateOrders($orders) {
    $seen = [];
    $duplicates = [];
    
    foreach ($orders as $order) {
        $key = $order['customer_id'] . '-' . $order['product_id'];
        
        if (isset($seen[$key])) {
            $duplicates[] = $order;
        } else {
            $seen[$key] = true;
        }
    }
    
    return $duplicates;
}

// Test
$orders = [
    ['id' => 1, 'customer_id' => 100, 'product_id' => 50],
    ['id' => 2, 'customer_id' => 101, 'product_id' => 51],
    ['id' => 3, 'customer_id' => 100, 'product_id' => 50], // Duplicate
];

print_r(findDuplicateOrders($orders));
```

**Problem 2: Group Transactions by Date**
```javascript
/**
 * Group transactions by date and calculate daily totals
 */
function groupTransactionsByDate(transactions) {
    return transactions.reduce((acc, transaction) => {
        const date = transaction.date.split('T')[0];
        
        if (!acc[date]) {
            acc[date] = {
                date,
                transactions: [],
                total: 0,
                count: 0
            };
        }
        
        acc[date].transactions.push(transaction);
        acc[date].total += transaction.amount;
        acc[date].count++;
        
        return acc;
    }, {});
}

// Test
const transactions = [
    { id: 1, date: '2025-01-15T10:00:00', amount: 100 },
    { id: 2, date: '2025-01-15T14:00:00', amount: 200 },
    { id: 3, date: '2025-01-16T09:00:00', amount: 150 }
];

console.log(groupTransactionsByDate(transactions));
```

**Problem 3: Implement Debounce Function**
```javascript
/**
 * Create a debounced version of a function
 * that delays execution until after wait milliseconds
 */
function debounce(func, wait) {
    let timeout;
    
    return function executedFunction(...args) {
        const later = () => {
            clearTimeout(timeout);
            func(...args);
        };
        
        clearTimeout(timeout);
        timeout = setTimeout(later, wait);
    };
}

// Usage
const searchAPI = debounce((query) => {
    console.log('Searching for:', query);
    // Make API call
}, 300);

// User types: "h" "e" "l" "l" "o"
// Only one API call made after 300ms of last keystroke
```

This is a great start. Would you like me to continue with more sections including Database Design Questions, API Design, Architecture patterns, and practical tips for the interview day?
