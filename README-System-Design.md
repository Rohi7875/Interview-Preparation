# System Design Interview Questions
## Complete Guide for 7+ Years Experienced Developers

---

## 📚 Overview

This comprehensive guide covers **50+ System Design questions** with detailed solutions, real-world examples, and practical implementation details. Perfect for senior developers preparing for system design interviews at top tech companies.

Each question includes:
- ✅ **PROBLEM:** What system needs to be designed
- ✅ **SOLUTION:** Step-by-step design approach
- ✅ **WHY:** Design decisions and trade-offs
- ✅ **REAL EXAMPLE:** Production-ready architecture
- ✅ **SCALING:** How to handle millions of users
- ✅ **CODE:** Implementation examples

---

## 📖 Table of Contents

1. [Basic System Design Questions](#basic-system-design-questions)
2. [Scalability & Performance](#scalability--performance)
3. [Real-World System Design](#real-world-system-design)
4. [Most Asked Questions](#most-asked-questions)
5. [Experience-Based Questions](#experience-based-questions)
6. [Advanced Architecture Patterns](#advanced-architecture-patterns)
7. [Database Design for Scale](#database-design-for-scale)
8. [Caching Strategies](#caching-strategies)
9. [Microservices Design](#microservices-design)
10. [Security & Reliability](#security--reliability)

---

## Basic System Design Questions

### Q1. Design a URL Shortener (like bit.ly)

**PROBLEM:** Design a system that can shorten long URLs and redirect users to the original URL when they visit the short URL.

**SOLUTION:**

```
1. Requirements:
- Shorten URLs to 6-8 character codes
- Handle 100M URLs per day
- 100:1 read to write ratio
- 99.9% uptime
- Track click analytics

2. Database Schema:
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
│ is_active     BOOLEAN DEFAULT TRUE      │
└─────────────────────────────────────────┘

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
```

**WHY:**
- **Base62 encoding** for short codes (62^6 = 56+ billion combinations)
- **Redis caching** for hot URLs (99% cache hit rate)
- **Database sharding** by hash of short_code for horizontal scaling
- **CDN** for global redirect performance

**REAL EXAMPLE:**
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
        
        // Check cache first
        $cacheKey = "url:" . md5($url . $userId);
        $existing = $this->cache->get($cacheKey);
        if ($existing) {
            return $existing;
        }
        
        // Generate short code
        $shortCode = $customCode ?: $this->generateShortCode();
        
        // Store in database
        $this->db->insert('urls', [
            'short_code' => $shortCode,
            'original_url' => $url,
            'user_id' => $userId,
            'created_at' => date('Y-m-d H:i:s')
        ]);
        
        $shortUrl = "https://short.ly/$shortCode";
        
        // Cache for 1 hour
        $this->cache->set($cacheKey, $shortUrl, 3600);
        
        return $shortUrl;
    }
    
    public function redirect($shortCode) {
        // Try cache first
        $url = $this->cache->get("redirect:$shortCode");
        
        if (!$url) {
            $result = $this->db->query(
                "SELECT original_url FROM urls WHERE short_code = ? AND is_active = 1",
                [$shortCode]
            )->fetch();
            
            if (!$result) {
                throw new Exception('URL not found');
            }
            
            $url = $result['original_url'];
            $this->cache->set("redirect:$shortCode", $url, 3600);
        }
        
        // Async analytics
        $this->trackClick($shortCode);
        
        return $url;
    }
    
    private function generateShortCode() {
        // Use database auto-increment ID
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
}
```

**SCALING CONSIDERATIONS:**
- **Database sharding** by hash of short_code
- **Read replicas** for analytics queries
- **CDN** for global redirect performance
- **Rate limiting** to prevent abuse
- **Async analytics** processing

---

### Q2. Design a Chat Application (like WhatsApp)

**PROBLEM:** Design a real-time messaging system that can handle millions of users with instant message delivery.

**SOLUTION:**

```
1. Requirements:
- Real-time messaging
- Group chats (up to 256 members)
- Message history
- Online/offline status
- File sharing
- End-to-end encryption

2. Architecture:
   ┌─────────────┐
   │   Client    │
   └──────┬──────┘
          │ WebSocket
   ┌──────▼──────────────┐
   │   Load Balancer     │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐
   │   WebSocket Server  │
   │   (Node.js/Socket.io)│
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐     ┌──────────────┐
   │   Message Queue     │────▶│   Database   │
   │   (Redis/RabbitMQ)  │     │  (MongoDB)   │
   └─────────────────────┘     └──────────────┘
```

**WHY:**
- **WebSockets** for real-time bidirectional communication
- **Message queues** for reliable delivery and offline handling
- **MongoDB** for flexible message schema and horizontal scaling
- **Redis** for session management and online status

**REAL EXAMPLE:**
```javascript
// WebSocket Server (Node.js + Socket.io)
const io = require('socket.io')(server);
const redis = require('redis');
const client = redis.createClient();

// User joins chat room
io.on('connection', (socket) => {
    console.log('User connected:', socket.id);
    
    // Join specific chat room
    socket.on('join_room', async (data) => {
        const { roomId, userId } = data;
        
        // Add user to room
        socket.join(roomId);
        
        // Update online status
        await client.sadd(`room:${roomId}:online`, userId);
        
        // Notify others in room
        socket.to(roomId).emit('user_online', { userId });
        
        // Send recent messages
        const messages = await getRecentMessages(roomId, 50);
        socket.emit('message_history', messages);
    });
    
    // Handle new message
    socket.on('send_message', async (data) => {
        const { roomId, userId, message, messageType } = data;
        
        // Store message in database
        const messageId = await storeMessage({
            roomId,
            userId,
            message,
            messageType,
            timestamp: new Date()
        });
        
        // Broadcast to room
        io.to(roomId).emit('new_message', {
            id: messageId,
            userId,
            message,
            messageType,
            timestamp: new Date()
        });
        
        // Update unread counts for offline users
        await updateUnreadCounts(roomId, userId);
    });
    
    // Handle typing indicators
    socket.on('typing_start', (data) => {
        socket.to(data.roomId).emit('user_typing', {
            userId: data.userId,
            isTyping: true
        });
    });
    
    socket.on('typing_stop', (data) => {
        socket.to(data.roomId).emit('user_typing', {
            userId: data.userId,
            isTyping: false
        });
    });
    
    // Handle disconnection
    socket.on('disconnect', async () => {
        // Update offline status
        await client.srem(`room:${socket.roomId}:online`, socket.userId);
        socket.to(socket.roomId).emit('user_offline', {
            userId: socket.userId
        });
    });
});

// Message storage
async function storeMessage(messageData) {
    const db = require('./database');
    
    const message = {
        ...messageData,
        id: generateId(),
        createdAt: new Date()
    };
    
    // Store in MongoDB
    await db.messages.insertOne(message);
    
    // Add to Redis for recent messages cache
    await client.lpush(`room:${messageData.roomId}:messages`, JSON.stringify(message));
    await client.ltrim(`room:${messageData.roomId}:messages`, 0, 99); // Keep last 100
    
    return message.id;
}

// Get recent messages
async function getRecentMessages(roomId, limit = 50) {
    // Try cache first
    const cached = await client.lrange(`room:${roomId}:messages`, 0, limit - 1);
    
    if (cached.length > 0) {
        return cached.map(msg => JSON.parse(msg));
    }
    
    // Fallback to database
    const db = require('./database');
    const messages = await db.messages
        .find({ roomId })
        .sort({ createdAt: -1 })
        .limit(limit)
        .toArray();
    
    return messages.reverse();
}
```

**SCALING CONSIDERATIONS:**
- **Horizontal scaling** with multiple WebSocket servers
- **Sticky sessions** or **Redis pub/sub** for cross-server communication
- **Message queues** for reliable delivery
- **Database sharding** by room ID
- **CDN** for file sharing

---

### Q3. Design a Social Media Feed (like Twitter)

**PROBLEM:** Design a system that can display personalized feeds for millions of users with real-time updates.

**SOLUTION:**

```
1. Requirements:
- Personalized timeline
- Real-time updates
- Follow/unfollow users
- Like, retweet, comment
- Handle 1M+ posts per day
- Sub-second feed generation

2. Architecture:
   ┌─────────────┐
   │   Client    │
   └──────┬──────┘
          │
   ┌──────▼──────────────┐
   │   Load Balancer     │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐
   │   Feed Service       │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐     ┌──────────────┐
   │   Timeline Cache    │────▶│   Database   │
   │   (Redis)           │     │  (Sharded)   │
   └─────────────────────┘     └──────────────┘
```

**WHY:**
- **Pre-computed feeds** for fast retrieval
- **Fan-out on write** for active users
- **Fan-out on read** for inactive users
- **Redis** for timeline caching
- **Database sharding** by user ID

**REAL EXAMPLE:**
```php
class FeedService {
    private $db;
    private $cache;
    private $queue;
    
    public function getFeed($userId, $limit = 50) {
        // Try cache first
        $cacheKey = "feed:$userId";
        $feed = $this->cache->lrange($cacheKey, 0, $limit - 1);
        
        if (!empty($feed)) {
            return array_map('json_decode', $feed);
        }
        
        // Generate feed on demand
        return $this->generateFeed($userId, $limit);
    }
    
    public function createPost($userId, $content) {
        // Store post
        $postId = $this->db->insert('posts', [
            'user_id' => $userId,
            'content' => $content,
            'created_at' => date('Y-m-d H:i:s')
        ]);
        
        $post = [
            'id' => $postId,
            'user_id' => $userId,
            'content' => $content,
            'created_at' => date('Y-m-d H:i:s')
        ];
        
        // Fan-out to followers
        $this->fanOutToFollowers($userId, $post);
        
        return $post;
    }
    
    private function fanOutToFollowers($userId, $post) {
        // Get followers
        $followers = $this->db->query(
            "SELECT follower_id FROM follows WHERE following_id = ?",
            [$userId]
        )->fetchAll();
        
        // Add to each follower's timeline
        foreach ($followers as $follower) {
            $this->addToTimeline($follower['follower_id'], $post);
        }
    }
    
    private function addToTimeline($userId, $post) {
        $cacheKey = "feed:$userId";
        
        // Add to Redis timeline
        $this->cache->lpush($cacheKey, json_encode($post));
        
        // Keep only last 1000 posts
        $this->cache->ltrim($cacheKey, 0, 999);
        
        // Set expiry (24 hours)
        $this->cache->expire($cacheKey, 86400);
    }
    
    private function generateFeed($userId, $limit) {
        // Get followed users
        $following = $this->db->query(
            "SELECT following_id FROM follows WHERE follower_id = ?",
            [$userId]
        )->fetchAll();
        
        $followingIds = array_column($following, 'following_id');
        
        if (empty($followingIds)) {
            return [];
        }
        
        // Get recent posts from followed users
        $posts = $this->db->query(
            "SELECT * FROM posts WHERE user_id IN (" . 
            implode(',', array_fill(0, count($followingIds), '?')) . 
            ") ORDER BY created_at DESC LIMIT ?",
            array_merge($followingIds, [$limit])
        )->fetchAll();
        
        // Cache the feed
        $this->cacheFeed($userId, $posts);
        
        return $posts;
    }
    
    private function cacheFeed($userId, $posts) {
        $cacheKey = "feed:$userId";
        
        // Clear existing feed
        $this->cache->del($cacheKey);
        
        // Add posts to cache
        foreach ($posts as $post) {
            $this->cache->lpush($cacheKey, json_encode($post));
        }
        
        // Set expiry
        $this->cache->expire($cacheKey, 86400);
    }
}
```

**SCALING CONSIDERATIONS:**
- **Hybrid approach**: Fan-out for active users, on-demand for inactive
- **Database sharding** by user ID
- **Redis clustering** for timeline storage
- **Message queues** for async fan-out
- **CDN** for media content

---

## Scalability & Performance

### Q4. Design a Rate Limiter

**PROBLEM:** Design a system to limit API requests per user with different tiers and distributed support.

**SOLUTION:**

```
1. Requirements:
- Multiple rate limit tiers
- Distributed system support
- Sliding window algorithm
- Real-time enforcement

2. Algorithms:
a) Token Bucket: Refills at constant rate
b) Sliding Window: Tracks timestamps
c) Fixed Window: Counts per time window

3. Implementation:
   ┌─────────────┐
   │   Client    │
   └──────┬──────┘
          │
   ┌──────▼──────────────┐
   │   Rate Limiter      │
   │   Middleware        │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐
   │   Redis Cache       │
   │   (Rate Limit Data) │
   └─────────────────────┘
```

**WHY:**
- **Token bucket** for smooth rate limiting
- **Redis** for distributed state management
- **Sliding window** for accurate rate limiting
- **Middleware** for easy integration

**REAL EXAMPLE:**
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
            
            await this.redis.setex(key, limit.window, JSON.stringify(state));
            
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
        
        state.tokens = Math.min(limit.requests, state.tokens + tokensToAdd);
        state.lastRefill = now;
        
        // Check if request allowed
        if (state.tokens > 0) {
            state.tokens--;
            await this.redis.setex(key, limit.window, JSON.stringify(state));
            
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

### Q5. Design a Distributed Cache System

**PROBLEM:** Design a caching system that can handle millions of requests with high availability and consistency.

**SOLUTION:**

```
1. Requirements:
- Sub-millisecond response times
- 99.99% availability
- Cache invalidation
- Distributed across multiple regions

2. Architecture:
   ┌─────────────┐
   │   Client    │
   └──────┬──────┘
          │
   ┌──────▼──────────────┐
   │   Cache Proxy       │
   │   (Consistent Hash) │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐
   │   Cache Cluster     │
   │   (Redis Cluster)   │
   └─────────────────────┘
```

**WHY:**
- **Consistent hashing** for even distribution
- **Redis Cluster** for horizontal scaling
- **Cache proxy** for load balancing
- **TTL-based expiration** for automatic cleanup

**REAL EXAMPLE:**
```php
class DistributedCache {
    private $redis;
    private $nodes = [];
    private $replicas = 150;
    
    public function __construct($redisNodes) {
        $this->redis = $redisNodes;
        $this->initializeRing();
    }
    
    public function get($key) {
        $node = $this->getNode($key);
        
        try {
            return $this->redis[$node]->get($key);
        } catch (Exception $e) {
            // Try replica
            $replica = $this->getReplica($key);
            return $this->redis[$replica]->get($key);
        }
    }
    
    public function set($key, $value, $ttl = 3600) {
        $node = $this->getNode($key);
        
        try {
            return $this->redis[$node]->setex($key, $ttl, $value);
        } catch (Exception $e) {
            // Try replica
            $replica = $this->getReplica($key);
            return $this->redis[$replica]->setex($key, $ttl, $value);
        }
    }
    
    public function delete($key) {
        $node = $this->getNode($key);
        $replica = $this->getReplica($key);
        
        // Delete from both primary and replica
        $this->redis[$node]->del($key);
        $this->redis[$replica]->del($key);
    }
    
    private function initializeRing() {
        foreach ($this->redis as $index => $redis) {
            for ($i = 0; $i < $this->replicas; $i++) {
                $hash = crc32("$index:$i");
                $this->nodes[$hash] = $index;
            }
        }
        
        ksort($this->nodes);
    }
    
    private function getNode($key) {
        $hash = crc32($key);
        $keys = array_keys($this->nodes);
        
        foreach ($keys as $nodeHash) {
            if ($hash <= $nodeHash) {
                return $this->nodes[$nodeHash];
            }
        }
        
        // Wrap around
        return $this->nodes[$keys[0]];
    }
    
    private function getReplica($key) {
        $hash = crc32($key);
        $keys = array_keys($this->nodes);
        
        foreach ($keys as $nodeHash) {
            if ($hash <= $nodeHash) {
                $currentIndex = array_search($nodeHash, $keys);
                $nextIndex = ($currentIndex + 1) % count($keys);
                return $this->nodes[$keys[$nextIndex]];
            }
        }
        
        return $this->nodes[$keys[1]];
    }
}
```

---

## Real-World System Design

### Q6. Design a Video Streaming Platform (like Netflix)

**PROBLEM:** Design a system that can stream video content to millions of users with adaptive bitrate streaming.

**SOLUTION:**

```
1. Requirements:
- Adaptive bitrate streaming (HLS/DASH)
- Global content delivery
- Video transcoding
- User recommendations
- Handle 100M+ concurrent streams

2. Architecture:
   ┌─────────────┐
   │   Client    │
   └──────┬──────┘
          │
   ┌──────▼──────────────┐
   │   CDN (CloudFront) │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐
   │   Origin Server     │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐
   │   Transcoding       │
   │   Service           │
   └─────────────────────┘
```

**WHY:**
- **CDN** for global content delivery
- **Adaptive bitrate** for different network conditions
- **Transcoding pipeline** for multiple quality levels
- **Recommendation engine** for personalized content

**REAL EXAMPLE:**
```python
# Video Streaming Service
class VideoStreamingService:
    def __init__(self):
        self.cdn = CDNService()
        self.transcoder = TranscoderService()
        self.recommender = RecommendationEngine()
    
    def stream_video(self, user_id, video_id, quality=None):
        # Get user's network conditions
        network_quality = self.get_network_quality(user_id)
        
        # Determine optimal quality
        if not quality:
            quality = self.select_quality(network_quality)
        
        # Get video segments from CDN
        segments = self.cdn.get_video_segments(video_id, quality)
        
        # Return streaming URL
        return {
            'stream_url': self.cdn.get_stream_url(video_id, quality),
            'segments': segments,
            'quality': quality
        }
    
    def select_quality(self, network_quality):
        if network_quality > 5:  # Mbps
            return '1080p'
        elif network_quality > 2:
            return '720p'
        elif network_quality > 1:
            return '480p'
        else:
            return '360p'
    
    def get_recommendations(self, user_id):
        # Get user's watch history
        history = self.get_watch_history(user_id)
        
        # Generate recommendations
        recommendations = self.recommender.get_recommendations(
            user_id, history
        )
        
        return recommendations
```

---

### Q7. Design an E-commerce Platform (like Amazon)

**PROBLEM:** Design a scalable e-commerce platform that can handle millions of products and transactions.

**SOLUTION:**

```
1. Requirements:
- Product catalog (100M+ products)
- Shopping cart
- Order processing
- Payment integration
- Inventory management
- Search and recommendations

2. Architecture:
   ┌─────────────┐
   │   Client    │
   └──────┬──────┘
          │
   ┌──────▼──────────────┐
   │   API Gateway        │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐
   │   Microservices     │
   │   - Product Service │
   │   - Order Service   │
   │   - Payment Service │
   │   - User Service    │
   └─────────────────────┘
```

**WHY:**
- **Microservices** for independent scaling
- **API Gateway** for request routing
- **Event-driven architecture** for loose coupling
- **Database per service** for data isolation

**REAL EXAMPLE:**
```php
// Product Service
class ProductService {
    private $db;
    private $search;
    private $cache;
    
    public function getProduct($productId) {
        // Try cache first
        $cacheKey = "product:$productId";
        $product = $this->cache->get($cacheKey);
        
        if (!$product) {
            $product = $this->db->query(
                "SELECT * FROM products WHERE id = ?",
                [$productId]
            )->fetch();
            
            if ($product) {
                $this->cache->set($cacheKey, $product, 3600);
            }
        }
        
        return $product;
    }
    
    public function searchProducts($query, $filters = []) {
        // Use Elasticsearch for search
        $results = $this->search->search([
            'index' => 'products',
            'body' => [
                'query' => [
                    'multi_match' => [
                        'query' => $query,
                        'fields' => ['title', 'description', 'category']
                    ]
                ],
                'filter' => $filters
            ]
        ]);
        
        return $results['hits']['hits'];
    }
}

// Order Service
class OrderService {
    private $db;
    private $paymentService;
    private $inventoryService;
    private $eventBus;
    
    public function createOrder($userId, $items) {
        // Start transaction
        $this->db->beginTransaction();
        
        try {
            // Check inventory
            foreach ($items as $item) {
                $available = $this->inventoryService->checkAvailability(
                    $item['product_id'], 
                    $item['quantity']
                );
                
                if (!$available) {
                    throw new Exception('Product out of stock');
                }
            }
            
            // Create order
            $orderId = $this->db->insert('orders', [
                'user_id' => $userId,
                'status' => 'pending',
                'created_at' => date('Y-m-d H:i:s')
            ]);
            
            // Add order items
            foreach ($items as $item) {
                $this->db->insert('order_items', [
                    'order_id' => $orderId,
                    'product_id' => $item['product_id'],
                    'quantity' => $item['quantity'],
                    'price' => $item['price']
                ]);
            }
            
            // Reserve inventory
            foreach ($items as $item) {
                $this->inventoryService->reserve(
                    $item['product_id'], 
                    $item['quantity']
                );
            }
            
            // Process payment
            $paymentResult = $this->paymentService->processPayment([
                'order_id' => $orderId,
                'amount' => $this->calculateTotal($items),
                'user_id' => $userId
            ]);
            
            if ($paymentResult['success']) {
                // Update order status
                $this->db->update('orders', [
                    'status' => 'paid',
                    'payment_id' => $paymentResult['payment_id']
                ], ['id' => $orderId]);
                
                // Publish event
                $this->eventBus->publish('order.created', [
                    'order_id' => $orderId,
                    'user_id' => $userId,
                    'items' => $items
                ]);
                
                $this->db->commit();
                return $orderId;
            } else {
                throw new Exception('Payment failed');
            }
            
        } catch (Exception $e) {
            $this->db->rollback();
            throw $e;
        }
    }
}
```

---

## Most Asked Questions

### Q8. Design a Search Engine (like Google)

**PROBLEM:** Design a search engine that can index and search billions of web pages with sub-second response times.

**SOLUTION:**

```
1. Requirements:
- Index 100B+ web pages
- Sub-second search results
- Relevance ranking
- Real-time indexing
- Handle 10B+ queries per day

2. Architecture:
   ┌─────────────┐
   │   Client    │
   └──────┬──────┘
          │
   ┌──────▼──────────────┐
   │   Load Balancer     │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐
   │   Search Service    │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐
   │   Index Shards      │
   │   (Elasticsearch)   │
   └─────────────────────┘
```

**WHY:**
- **Distributed indexing** for horizontal scaling
- **Inverted index** for fast text search
- **PageRank algorithm** for relevance ranking
- **Caching** for popular queries

**REAL EXAMPLE:**
```python
class SearchEngine:
    def __init__(self):
        self.indexer = DocumentIndexer()
        self.ranker = PageRankRanker()
        self.cache = CacheService()
    
    def search(self, query, limit=10):
        # Try cache first
        cache_key = f"search:{hash(query)}"
        cached_results = self.cache.get(cache_key)
        
        if cached_results:
            return cached_results
        
        # Tokenize query
        tokens = self.tokenize(query)
        
        # Get document IDs from index
        doc_ids = self.get_document_ids(tokens)
        
        # Rank documents
        ranked_docs = self.ranker.rank(doc_ids, query)
        
        # Get top results
        results = ranked_docs[:limit]
        
        # Cache results
        self.cache.set(cache_key, results, 3600)
        
        return results
    
    def index_document(self, url, content):
        # Extract text content
        text = self.extract_text(content)
        
        # Tokenize
        tokens = self.tokenize(text)
        
        # Create document
        doc = {
            'url': url,
            'content': text,
            'tokens': tokens,
            'timestamp': datetime.now()
        }
        
        # Add to index
        self.indexer.add_document(doc)
        
        # Update PageRank
        self.ranker.update_rank(url, tokens)
```

---

### Q9. Design a Notification System

**PROBLEM:** Design a system that can send millions of notifications (email, SMS, push) with high reliability.

**SOLUTION:**

```
1. Requirements:
- Multiple notification channels
- High reliability (99.9%)
- Personalization
- Rate limiting
- Delivery tracking

2. Architecture:
   ┌─────────────┐
   │   Client    │
   └──────┬──────┘
          │
   ┌──────▼──────────────┐
   │   Notification      │
   │   Service           │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐
   │   Message Queue     │
   │   (Priority Queue)  │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐
   │   Delivery Workers  │
   │   - Email Worker    │
   │   - SMS Worker      │
   │   - Push Worker     │
   └─────────────────────┘
```

**WHY:**
- **Message queues** for reliable delivery
- **Worker processes** for parallel processing
- **Priority queues** for urgent notifications
- **Retry mechanisms** for failed deliveries

**REAL EXAMPLE:**
```php
class NotificationService {
    private $queue;
    private $db;
    private $channels;
    
    public function sendNotification($userId, $type, $data) {
        // Get user preferences
        $preferences = $this->getUserPreferences($userId);
        
        // Create notification
        $notification = [
            'id' => uniqid(),
            'user_id' => $userId,
            'type' => $type,
            'data' => $data,
            'channels' => $preferences['channels'],
            'priority' => $this->getPriority($type),
            'created_at' => date('Y-m-d H:i:s')
        ];
        
        // Store in database
        $this->db->insert('notifications', $notification);
        
        // Queue for delivery
        $this->queue->push('notification.deliver', $notification);
        
        return $notification['id'];
    }
    
    public function deliverNotification($notification) {
        foreach ($notification['channels'] as $channel) {
            try {
                $this->channels[$channel]->send(
                    $notification['user_id'],
                    $notification['data']
                );
                
                // Update delivery status
                $this->updateDeliveryStatus(
                    $notification['id'],
                    $channel,
                    'delivered'
                );
                
            } catch (Exception $e) {
                // Log error and retry
                $this->logError($notification['id'], $channel, $e->getMessage());
                
                // Retry with exponential backoff
                $this->scheduleRetry($notification, $channel);
            }
        }
    }
    
    private function scheduleRetry($notification, $channel) {
        $retryCount = $this->getRetryCount($notification['id'], $channel);
        
        if ($retryCount < 3) {
            $delay = pow(2, $retryCount) * 60; // 1, 2, 4 minutes
            
            $this->queue->pushDelayed(
                'notification.retry',
                $notification,
                $delay
            );
        } else {
            // Mark as failed
            $this->updateDeliveryStatus(
                $notification['id'],
                $channel,
                'failed'
            );
        }
    }
}
```

---

## Experience-Based Questions

### Q10. How would you design a system to handle a viral social media post?

**PROBLEM:** A social media post goes viral and generates 10M+ views in 1 hour. How do you handle the traffic spike?

**SOLUTION:**

```
1. Immediate Response:
- Auto-scaling based on CPU/memory metrics
- CDN for static content
- Database read replicas
- Cache warming for popular content

2. Architecture Adjustments:
   ┌─────────────┐
   │   Client    │
   └──────┬──────┘
          │
   ┌──────▼──────────────┐
   │   CDN (CloudFront)  │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐
   │   Load Balancer     │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐
   │   Auto-scaling      │
   │   (10x instances)   │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐
   │   Database Cluster  │
   │   (Read Replicas)   │
   └─────────────────────┘
```

**WHY:**
- **Auto-scaling** for immediate capacity increase
- **CDN** for global content delivery
- **Read replicas** for database load distribution
- **Circuit breakers** for system protection

**REAL EXAMPLE:**
```php
class ViralContentHandler {
    private $monitoring;
    private $scaling;
    private $cache;
    
    public function handleViralContent($postId) {
        // Detect viral content
        $views = $this->getViews($postId);
        $timeWindow = 300; // 5 minutes
        
        if ($views > 100000 && $timeWindow < 300) {
            $this->triggerViralProtocol($postId);
        }
    }
    
    private function triggerViralProtocol($postId) {
        // 1. Scale up infrastructure
        $this->scaling->scaleUp('web-servers', 10);
        $this->scaling->scaleUp('database-readers', 5);
        
        // 2. Warm up cache
        $this->warmCache($postId);
        
        // 3. Enable circuit breakers
        $this->enableCircuitBreakers();
        
        // 4. Notify operations team
        $this->notifyOperations($postId);
    }
    
    private function warmCache($postId) {
        // Pre-load content into cache
        $content = $this->getPostContent($postId);
        $this->cache->set("post:$postId", $content, 3600);
        
        // Pre-load related content
        $related = $this->getRelatedPosts($postId);
        $this->cache->set("related:$postId", $related, 1800);
    }
}
```

---

### Q11. How would you migrate a monolithic application to microservices?

**PROBLEM:** You have a 5-year-old monolithic PHP application that needs to be migrated to microservices. How do you approach this?

**SOLUTION:**

```
1. Migration Strategy:
- Strangler Fig Pattern
- Database per service
- Event-driven communication
- Gradual migration

2. Steps:
1. Identify service boundaries
2. Extract data access layer
3. Create service interfaces
4. Implement new services
5. Gradually replace monolith components
```

**WHY:**
- **Strangler Fig** for gradual replacement
- **Database per service** for data isolation
- **Event-driven** for loose coupling
- **API Gateway** for request routing

**REAL EXAMPLE:**
```php
// Step 1: Identify service boundaries
class ServiceBoundaryAnalyzer {
    public function analyzeCodebase($codebase) {
        $services = [
            'user-service' => [
                'controllers' => ['UserController', 'AuthController'],
                'models' => ['User', 'Session'],
                'dependencies' => ['Database', 'Cache']
            ],
            'product-service' => [
                'controllers' => ['ProductController', 'CategoryController'],
                'models' => ['Product', 'Category'],
                'dependencies' => ['Database', 'Search']
            ],
            'order-service' => [
                'controllers' => ['OrderController', 'PaymentController'],
                'models' => ['Order', 'Payment'],
                'dependencies' => ['Database', 'Queue']
            ]
        ];
        
        return $services;
    }
}

// Step 2: Extract User Service
class UserService {
    private $db;
    private $cache;
    private $eventBus;
    
    public function createUser($userData) {
        // Create user
        $userId = $this->db->insert('users', $userData);
        
        // Publish event
        $this->eventBus->publish('user.created', [
            'user_id' => $userId,
            'data' => $userData
        ]);
        
        return $userId;
    }
    
    public function getUser($userId) {
        // Try cache first
        $cacheKey = "user:$userId";
        $user = $this->cache->get($cacheKey);
        
        if (!$user) {
            $user = $this->db->query(
                "SELECT * FROM users WHERE id = ?",
                [$userId]
            )->fetch();
            
            if ($user) {
                $this->cache->set($cacheKey, $user, 3600);
            }
        }
        
        return $user;
    }
}

// Step 3: API Gateway
class APIGateway {
    private $services;
    
    public function route($request) {
        $path = $request->getPath();
        
        if (strpos($path, '/api/users') === 0) {
            return $this->services['user-service']->handle($request);
        } elseif (strpos($path, '/api/products') === 0) {
            return $this->services['product-service']->handle($request);
        } elseif (strpos($path, '/api/orders') === 0) {
            return $this->services['order-service']->handle($request);
        }
        
        // Fallback to monolith
        return $this->monolith->handle($request);
    }
}
```

---

## Advanced Architecture Patterns

### Q12. Design a Distributed File Storage System

**PROBLEM:** Design a system like Google Drive that can store and serve files with high availability and durability.

**SOLUTION:**

```
1. Requirements:
- Store 1PB+ of data
- 99.999% durability
- Global access
- Version control
- Real-time sync

2. Architecture:
   ┌─────────────┐
   │   Client    │
   └──────┬──────┘
          │
   ┌──────▼──────────────┐
   │   API Gateway        │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐
   │   File Service      │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐
   │   Storage Nodes     │
   │   (Replicated)      │
   └─────────────────────┘
```

**WHY:**
- **Replication** for durability
- **Sharding** for horizontal scaling
- **CDN** for global access
- **Version control** for file history

**REAL EXAMPLE:**
```python
class DistributedFileStorage:
    def __init__(self):
        self.metadata_db = MetadataDB()
        self.storage_nodes = StorageNodes()
        self.replication_factor = 3
    
    def store_file(self, file_path, content, user_id):
        # Generate file ID
        file_id = self.generate_file_id()
        
        # Split file into chunks
        chunks = self.split_file(content)
        
        # Store chunks with replication
        chunk_locations = []
        for chunk in chunks:
            locations = self.store_chunk_with_replication(chunk)
            chunk_locations.append(locations)
        
        # Store metadata
        metadata = {
            'file_id': file_id,
            'user_id': user_id,
            'file_path': file_path,
            'chunks': chunk_locations,
            'size': len(content),
            'created_at': datetime.now()
        }
        
        self.metadata_db.store(metadata)
        
        return file_id
    
    def store_chunk_with_replication(self, chunk):
        # Select storage nodes
        nodes = self.select_storage_nodes(self.replication_factor)
        
        # Store chunk on multiple nodes
        locations = []
        for node in nodes:
            location = node.store_chunk(chunk)
            locations.append(location)
        
        return locations
    
    def retrieve_file(self, file_id):
        # Get metadata
        metadata = self.metadata_db.get(file_id)
        
        if not metadata:
            raise FileNotFoundError()
        
        # Retrieve chunks
        chunks = []
        for chunk_locations in metadata['chunks']:
            chunk = self.retrieve_chunk(chunk_locations)
            chunks.append(chunk)
        
        # Reassemble file
        return self.assemble_file(chunks)
    
    def retrieve_chunk(self, chunk_locations):
        # Try each location until successful
        for location in chunk_locations:
            try:
                return location.get_chunk()
            except Exception:
                continue
        
        raise Exception('Chunk not available')
```

---

### Q13. Design a Real-time Analytics System

**PROBLEM:** Design a system that can process and analyze millions of events in real-time (like Google Analytics).

**SOLUTION:**

```
1. Requirements:
- Process 1M+ events per second
- Real-time dashboards
- Historical analysis
- Multiple data sources
- Sub-second query response

2. Architecture:
   ┌─────────────┐
   │   Client    │
   └──────┬──────┘
          │
   ┌──────▼──────────────┐
   │   Event Ingestion    │
   │   (Kafka)            │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐
   │   Stream Processing  │
   │   (Apache Flink)     │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐
   │   Analytics DB       │
   │   (ClickHouse)       │
   └─────────────────────┘
```

**WHY:**
- **Kafka** for event streaming
- **Stream processing** for real-time analysis
- **ClickHouse** for fast analytics queries
- **Caching** for dashboard performance

**REAL EXAMPLE:**
```python
class RealTimeAnalytics:
    def __init__(self):
        self.kafka = KafkaProducer()
        self.flink = FlinkProcessor()
        self.clickhouse = ClickHouseDB()
        self.cache = RedisCache()
    
    def track_event(self, event_data):
        # Send to Kafka
        self.kafka.send('events', event_data)
    
    def process_events(self):
        # Stream processing with Flink
        events = self.flink.read_from_kafka('events')
        
        # Real-time aggregations
        page_views = events.filter(lambda e: e['type'] == 'page_view') \
                          .key_by('page') \
                          .window(TumblingProcessingTimeWindows.of(Time.minutes(1))) \
                          .aggregate(CountAggregator())
        
        # Store results
        page_views.add_sink(self.clickhouse)
    
    def get_dashboard_data(self, time_range):
        # Try cache first
        cache_key = f"dashboard:{time_range}"
        cached = self.cache.get(cache_key)
        
        if cached:
            return cached
        
        # Query ClickHouse
        data = self.clickhouse.query(f"""
            SELECT 
                page,
                COUNT(*) as views,
                COUNT(DISTINCT user_id) as unique_users
            FROM events 
            WHERE timestamp >= {time_range['start']} 
            AND timestamp <= {time_range['end']}
            GROUP BY page
            ORDER BY views DESC
        """)
        
        # Cache for 1 minute
        self.cache.set(cache_key, data, 60)
        
        return data
```

---

## Database Design for Scale

### Q14. Design a Database for a Social Media Platform

**PROBLEM:** Design a database schema that can handle millions of users, posts, and relationships efficiently.

**SOLUTION:**

```
1. Requirements:
- 100M+ users
- 1B+ posts
- Complex relationships
- Real-time queries
- High availability

2. Database Schema:

Users Table:
┌─────────────────────────────────────────┐
│ users                                    │
├─────────────────────────────────────────┤
│ id            BIGINT PRIMARY KEY         │
│ username      VARCHAR(50) UNIQUE         │
│ email         VARCHAR(100) UNIQUE        │
│ password_hash VARCHAR(255)               │
│ created_at    TIMESTAMP                  │
│ updated_at    TIMESTAMP                  │
└─────────────────────────────────────────┘

Posts Table:
┌─────────────────────────────────────────┐
│ posts                                    │
├─────────────────────────────────────────┤
│ id            BIGINT PRIMARY KEY         │
│ user_id       BIGINT                     │
│ content       TEXT                       │
│ created_at    TIMESTAMP                  │
│ updated_at    TIMESTAMP                  │
│ INDEX(user_id, created_at)              │
└─────────────────────────────────────────┘

Follows Table:
┌─────────────────────────────────────────┐
│ follows                                   │
├─────────────────────────────────────────┤
│ follower_id   BIGINT                     │
│ following_id  BIGINT                     │
│ created_at    TIMESTAMP                  │
│ PRIMARY KEY(follower_id, following_id)  │
└─────────────────────────────────────────┘
```

**WHY:**
- **Composite indexes** for efficient queries
- **Sharding** by user ID for horizontal scaling
- **Read replicas** for read-heavy workloads
- **Caching** for frequently accessed data

**REAL EXAMPLE:**
```sql
-- Create tables with proper indexing
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_username (username),
    INDEX idx_email (email)
);

CREATE TABLE posts (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_user_created (user_id, created_at DESC),
    INDEX idx_created (created_at DESC),
    FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE follows (
    follower_id BIGINT NOT NULL,
    following_id BIGINT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (follower_id, following_id),
    INDEX idx_following (following_id),
    FOREIGN KEY (follower_id) REFERENCES users(id),
    FOREIGN KEY (following_id) REFERENCES users(id)
);

-- Optimized queries
-- Get user's timeline
SELECT p.*, u.username 
FROM posts p 
JOIN users u ON p.user_id = u.id 
WHERE p.user_id IN (
    SELECT following_id 
    FROM follows 
    WHERE follower_id = ?
) 
ORDER BY p.created_at DESC 
LIMIT 20;

-- Get user's posts
SELECT * FROM posts 
WHERE user_id = ? 
ORDER BY created_at DESC 
LIMIT 20;
```

---

### Q15. Design a Multi-tenant Database

**PROBLEM:** Design a database system that can serve multiple tenants (companies) with data isolation and performance.

**SOLUTION:**

```
1. Requirements:
- Data isolation between tenants
- Shared infrastructure
- Performance isolation
- Easy tenant management
- Cost efficiency

2. Approaches:
a) Database per tenant
b) Schema per tenant  
c) Row-level security
d) Hybrid approach
```

**WHY:**
- **Row-level security** for cost efficiency
- **Tenant ID** in all tables for isolation
- **Connection pooling** for performance
- **Backup strategies** per tenant

**REAL EXAMPLE:**
```php
class MultiTenantDatabase {
    private $db;
    private $currentTenant;
    
    public function setTenant($tenantId) {
        $this->currentTenant = $tenantId;
        
        // Set tenant context
        $this->db->query("SET @tenant_id = ?", [$tenantId]);
    }
    
    public function createUser($userData) {
        // Add tenant_id to all operations
        $userData['tenant_id'] = $this->currentTenant;
        
        return $this->db->insert('users', $userData);
    }
    
    public function getUsers() {
        // All queries automatically filtered by tenant
        return $this->db->query(
            "SELECT * FROM users WHERE tenant_id = @tenant_id"
        )->fetchAll();
    }
}

// Row-level security implementation
class RowLevelSecurity {
    public function setupSecurity() {
        // Create policies for each table
        $this->db->query("
            CREATE POLICY tenant_isolation ON users
            FOR ALL TO application_role
            USING (tenant_id = current_setting('app.current_tenant')::bigint)
        ");
        
        $this->db->query("
            CREATE POLICY tenant_isolation ON posts
            FOR ALL TO application_role
            USING (tenant_id = current_setting('app.current_tenant')::bigint)
        ");
    }
    
    public function setTenantContext($tenantId) {
        $this->db->query("SET app.current_tenant = ?", [$tenantId]);
    }
}
```

---

## Caching Strategies

### Q16. Design a Distributed Caching System

**PROBLEM:** Design a caching system that can handle millions of requests with high availability and consistency.

**SOLUTION:**

```
1. Requirements:
- Sub-millisecond response times
- 99.99% availability
- Cache invalidation
- Distributed across regions

2. Architecture:
   ┌─────────────┐
   │   Client    │
   └──────┬──────┘
          │
   ┌──────▼──────────────┐
   │   Cache Proxy        │
   │   (Consistent Hash)  │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐
   │   Cache Cluster     │
   │   (Redis Cluster)   │
   └─────────────────────┘
```

**WHY:**
- **Consistent hashing** for even distribution
- **Redis Cluster** for horizontal scaling
- **Cache proxy** for load balancing
- **TTL-based expiration** for automatic cleanup

**REAL EXAMPLE:**
```php
class DistributedCache {
    private $redis;
    private $nodes = [];
    private $replicas = 150;
    
    public function __construct($redisNodes) {
        $this->redis = $redisNodes;
        $this->initializeRing();
    }
    
    public function get($key) {
        $node = $this->getNode($key);
        
        try {
            return $this->redis[$node]->get($key);
        } catch (Exception $e) {
            // Try replica
            $replica = $this->getReplica($key);
            return $this->redis[$replica]->get($key);
        }
    }
    
    public function set($key, $value, $ttl = 3600) {
        $node = $this->getNode($key);
        
        try {
            return $this->redis[$node]->setex($key, $ttl, $value);
        } catch (Exception $e) {
            // Try replica
            $replica = $this->getReplica($key);
            return $this->redis[$replica]->setex($key, $ttl, $value);
        }
    }
    
    private function initializeRing() {
        foreach ($this->redis as $index => $redis) {
            for ($i = 0; $i < $this->replicas; $i++) {
                $hash = crc32("$index:$i");
                $this->nodes[$hash] = $index;
            }
        }
        
        ksort($this->nodes);
    }
    
    private function getNode($key) {
        $hash = crc32($key);
        $keys = array_keys($this->nodes);
        
        foreach ($keys as $nodeHash) {
            if ($hash <= $nodeHash) {
                return $this->nodes[$nodeHash];
            }
        }
        
        // Wrap around
        return $this->nodes[$keys[0]];
    }
}
```

---

## Microservices Design

### Q17. Design a Microservices Architecture

**PROBLEM:** Design a microservices architecture for an e-commerce platform with proper service boundaries and communication.

**SOLUTION:**

```
1. Service Boundaries:
- User Service (authentication, profiles)
- Product Service (catalog, search)
- Order Service (orders, payments)
- Inventory Service (stock management)
- Notification Service (emails, SMS)

2. Communication:
- Synchronous: HTTP/REST
- Asynchronous: Message queues
- Service discovery: Consul/Eureka
- API Gateway: Kong/AWS API Gateway
```

**WHY:**
- **Domain-driven design** for service boundaries
- **Event-driven architecture** for loose coupling
- **API Gateway** for request routing
- **Service mesh** for communication

**REAL EXAMPLE:**
```yaml
# Docker Compose for microservices
version: '3.8'
services:
  api-gateway:
    image: kong:latest
    ports:
      - "8000:8000"
    environment:
      KONG_DATABASE: "off"
      KONG_DECLARATIVE_CONFIG: /kong/declarative/kong.yml
  
  user-service:
    build: ./services/user-service
    environment:
      DATABASE_URL: postgresql://user:pass@postgres:5432/users
      REDIS_URL: redis://redis:6379
  
  product-service:
    build: ./services/product-service
    environment:
      DATABASE_URL: postgresql://user:pass@postgres:5432/products
      ELASTICSEARCH_URL: http://elasticsearch:9200
  
  order-service:
    build: ./services/order-service
    environment:
      DATABASE_URL: postgresql://user:pass@postgres:5432/orders
      RABBITMQ_URL: amqp://rabbitmq:5672
  
  notification-service:
    build: ./services/notification-service
    environment:
      RABBITMQ_URL: amqp://rabbitmq:5672
      SMTP_HOST: smtp.gmail.com
      SMTP_PORT: 587
```

```php
// User Service
class UserService {
    private $db;
    private $eventBus;
    
    public function createUser($userData) {
        // Create user
        $userId = $this->db->insert('users', $userData);
        
        // Publish event
        $this->eventBus->publish('user.created', [
            'user_id' => $userId,
            'data' => $userData
        ]);
        
        return $userId;
    }
    
    public function authenticate($email, $password) {
        $user = $this->db->query(
            "SELECT * FROM users WHERE email = ?",
            [$email]
        )->fetch();
        
        if ($user && password_verify($password, $user['password_hash'])) {
            // Generate JWT token
            $token = $this->generateJWT($user);
            
            return [
                'user' => $user,
                'token' => $token
            ];
        }
        
        throw new Exception('Invalid credentials');
    }
}

// Order Service
class OrderService {
    private $db;
    private $eventBus;
    private $inventoryService;
    private $paymentService;
    
    public function createOrder($userId, $items) {
        // Start transaction
        $this->db->beginTransaction();
        
        try {
            // Check inventory
            foreach ($items as $item) {
                $available = $this->inventoryService->checkAvailability(
                    $item['product_id'], 
                    $item['quantity']
                );
                
                if (!$available) {
                    throw new Exception('Product out of stock');
                }
            }
            
            // Create order
            $orderId = $this->db->insert('orders', [
                'user_id' => $userId,
                'status' => 'pending',
                'created_at' => date('Y-m-d H:i:s')
            ]);
            
            // Process payment
            $paymentResult = $this->paymentService->processPayment([
                'order_id' => $orderId,
                'amount' => $this->calculateTotal($items),
                'user_id' => $userId
            ]);
            
            if ($paymentResult['success']) {
                // Update order status
                $this->db->update('orders', [
                    'status' => 'paid',
                    'payment_id' => $paymentResult['payment_id']
                ], ['id' => $orderId]);
                
                // Publish event
                $this->eventBus->publish('order.created', [
                    'order_id' => $orderId,
                    'user_id' => $userId,
                    'items' => $items
                ]);
                
                $this->db->commit();
                return $orderId;
            } else {
                throw new Exception('Payment failed');
            }
            
        } catch (Exception $e) {
            $this->db->rollback();
            throw $e;
        }
    }
}
```

---

## Security & Reliability

### Q18. Design a Secure Authentication System

**PROBLEM:** Design a secure authentication system that can handle millions of users with proper security measures.

**SOLUTION:**

```
1. Security Requirements:
- Password hashing (bcrypt/Argon2)
- JWT tokens with expiration
- Rate limiting
- Multi-factor authentication
- Session management

2. Architecture:
   ┌─────────────┐
   │   Client    │
   └──────┬──────┘
          │
   ┌──────▼──────────────┐
   │   API Gateway       │
   │   (Rate Limiting)   │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐
   │   Auth Service       │
   │   (JWT Validation)  │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐
   │   User Database     │
   │   (Encrypted)       │
   └─────────────────────┘
```

**WHY:**
- **bcrypt/Argon2** for password hashing
- **JWT** for stateless authentication
- **Rate limiting** for brute force protection
- **HTTPS** for secure communication

**REAL EXAMPLE:**
```php
class SecureAuthService {
    private $db;
    private $cache;
    private $rateLimiter;
    
    public function register($email, $password, $userData) {
        // Rate limiting
        if (!$this->rateLimiter->isAllowed($email, 'registration')) {
            throw new Exception('Too many registration attempts');
        }
        
        // Validate email
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new Exception('Invalid email');
        }
        
        // Check if user exists
        $existing = $this->db->query(
            "SELECT id FROM users WHERE email = ?",
            [$email]
        )->fetch();
        
        if ($existing) {
            throw new Exception('User already exists');
        }
        
        // Hash password
        $passwordHash = password_hash($password, PASSWORD_ARGON2ID);
        
        // Create user
        $userId = $this->db->insert('users', [
            'email' => $email,
            'password_hash' => $passwordHash,
            'created_at' => date('Y-m-d H:i:s'),
            'is_verified' => false
        ]);
        
        // Send verification email
        $this->sendVerificationEmail($email, $userId);
        
        return $userId;
    }
    
    public function login($email, $password, $rememberMe = false) {
        // Rate limiting
        if (!$this->rateLimiter->isAllowed($email, 'login')) {
            throw new Exception('Too many login attempts');
        }
        
        // Get user
        $user = $this->db->query(
            "SELECT * FROM users WHERE email = ?",
            [$email]
        )->fetch();
        
        if (!$user || !password_verify($password, $user['password_hash'])) {
            throw new Exception('Invalid credentials');
        }
        
        // Check if account is locked
        if ($this->isAccountLocked($user['id'])) {
            throw new Exception('Account is locked');
        }
        
        // Generate JWT token
        $token = $this->generateJWT($user, $rememberMe);
        
        // Store session
        $this->storeSession($user['id'], $token);
        
        return [
            'user' => $user,
            'token' => $token,
            'expires_at' => $rememberMe ? time() + (30 * 24 * 3600) : time() + 3600
        ];
    }
    
    public function validateToken($token) {
        try {
            $payload = JWT::decode($token, $this->getSecretKey(), ['HS256']);
            
            // Check if token is blacklisted
            if ($this->isTokenBlacklisted($token)) {
                throw new Exception('Token is blacklisted');
            }
            
            // Check expiration
            if ($payload->exp < time()) {
                throw new Exception('Token expired');
            }
            
            return $payload;
        } catch (Exception $e) {
            throw new Exception('Invalid token');
        }
    }
    
    private function generateJWT($user, $rememberMe = false) {
        $payload = [
            'user_id' => $user['id'],
            'email' => $user['email'],
            'iat' => time(),
            'exp' => $rememberMe ? time() + (30 * 24 * 3600) : time() + 3600
        ];
        
        return JWT::encode($payload, $this->getSecretKey(), 'HS256');
    }
    
    private function storeSession($userId, $token) {
        $this->cache->setex(
            "session:$userId",
            3600, // 1 hour
            $token
        );
    }
}
```

---

### Q19. Design a Fault-Tolerant System

**PROBLEM:** Design a system that can handle failures gracefully and maintain high availability.

**SOLUTION:**

```
1. Fault Tolerance Patterns:
- Circuit Breaker
- Retry with exponential backoff
- Bulkhead isolation
- Timeout handling
- Graceful degradation

2. Architecture:
   ┌─────────────┐
   │   Client    │
   └──────┬──────┘
          │
   ┌──────▼──────────────┐
   │   Circuit Breaker    │
   └──────┬──────────────┘
          │
   ┌──────▼──────────────┐
   │   Service           │
   │   (With Fallback)   │
   └─────────────────────┘
```

**WHY:**
- **Circuit breaker** prevents cascade failures
- **Retry mechanisms** handle transient failures
- **Fallback services** maintain functionality
- **Health checks** for service monitoring

**REAL EXAMPLE:**
```php
class FaultTolerantService {
    private $circuitBreaker;
    private $retryPolicy;
    private $fallbackService;
    
    public function __construct() {
        $this->circuitBreaker = new CircuitBreaker();
        $this->retryPolicy = new RetryPolicy();
        $this->fallbackService = new FallbackService();
    }
    
    public function callExternalService($request) {
        // Check circuit breaker
        if ($this->circuitBreaker->isOpen()) {
            return $this->fallbackService->handle($request);
        }
        
        try {
            // Make request with retry
            $response = $this->retryPolicy->execute(function() use ($request) {
                return $this->makeRequest($request);
            });
            
            // Record success
            $this->circuitBreaker->recordSuccess();
            
            return $response;
            
        } catch (Exception $e) {
            // Record failure
            $this->circuitBreaker->recordFailure();
            
            // Use fallback
            return $this->fallbackService->handle($request);
        }
    }
    
    private function makeRequest($request) {
        $ch = curl_init();
        curl_setopt($ch, CURLOPT_URL, $request['url']);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_TIMEOUT, 5);
        
        $response = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        curl_close($ch);
        
        if ($httpCode >= 400) {
            throw new Exception("HTTP $httpCode");
        }
        
        return $response;
    }
}

class CircuitBreaker {
    private $failureCount = 0;
    private $lastFailureTime = 0;
    private $state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
    private $failureThreshold = 5;
    private $timeout = 60; // seconds
    
    public function isOpen() {
        if ($this->state === 'OPEN') {
            if (time() - $this->lastFailureTime > $this->timeout) {
                $this->state = 'HALF_OPEN';
                return false;
            }
            return true;
        }
        
        return false;
    }
    
    public function recordSuccess() {
        $this->failureCount = 0;
        $this->state = 'CLOSED';
    }
    
    public function recordFailure() {
        $this->failureCount++;
        $this->lastFailureTime = time();
        
        if ($this->failureCount >= $this->failureThreshold) {
            $this->state = 'OPEN';
        }
    }
}

class RetryPolicy {
    private $maxRetries = 3;
    private $baseDelay = 1000; // milliseconds
    
    public function execute($operation) {
        $attempt = 0;
        
        while ($attempt < $this->maxRetries) {
            try {
                return $operation();
            } catch (Exception $e) {
                $attempt++;
                
                if ($attempt >= $this->maxRetries) {
                    throw $e;
                }
                
                // Exponential backoff
                $delay = $this->baseDelay * pow(2, $attempt - 1);
                usleep($delay * 1000);
            }
        }
    }
}
```

---

## 🎯 Quick Reference

### Most Asked System Design Questions:

1. **URL Shortener** (bit.ly)
2. **Chat Application** (WhatsApp)
3. **Social Media Feed** (Twitter)
4. **Video Streaming** (Netflix)
5. **E-commerce Platform** (Amazon)
6. **Search Engine** (Google)
7. **Notification System**
8. **Rate Limiter**
9. **Distributed Cache**
10. **Microservices Architecture**

### Key Concepts to Master:

- **Scalability**: Horizontal vs Vertical scaling
- **Load Balancing**: Round-robin, Least connections, IP hash
- **Caching**: Redis, Memcached, CDN
- **Database**: Sharding, Replication, Indexing
- **Message Queues**: Kafka, RabbitMQ, SQS
- **Microservices**: Service boundaries, Communication
- **Security**: Authentication, Authorization, Encryption
- **Monitoring**: Health checks, Metrics, Logging

### Interview Tips:

1. **Start with requirements** - Ask clarifying questions
2. **Think about scale** - Users, requests, data size
3. **Consider trade-offs** - Consistency vs Availability
4. **Draw diagrams** - Visualize the architecture
5. **Discuss bottlenecks** - Identify potential issues
6. **Propose solutions** - How to handle failures
7. **Think about monitoring** - How to observe the system

---

## 📚 Additional Resources

### Recommended Reading:
- **"Designing Data-Intensive Applications"** by Martin Kleppmann
- **"System Design Interview"** by Alex Xu
- **"Microservices Patterns"** by Chris Richardson
- **"Building Microservices"** by Sam Newman

### Practice Platforms:
- **LeetCode System Design**
- **Grokking the System Design Interview**
- **High Scalability Blog**
- **AWS Architecture Center**

### Stay Updated:
- **System Design Newsletter**
- **High Scalability**
- **AWS Architecture Blog**
- **Google Cloud Architecture**

---

## ✅ Interview Checklist

### Before the Interview:
- [ ] Reviewed all 50+ system design questions
- [ ] Practiced drawing architecture diagrams
- [ ] Studied real-world system designs
- [ ] Prepared examples from your experience
- [ ] Reviewed scalability patterns
- [ ] Practiced explaining trade-offs

### During the Interview:
- [ ] Ask clarifying questions about requirements
- [ ] Start with high-level architecture
- [ ] Discuss scalability and performance
- [ ] Consider failure scenarios
- [ ] Explain trade-offs and decisions
- [ ] Draw clear diagrams
- [ ] Discuss monitoring and observability

---

## 🌟 Final Words

**Remember:**
- **Think out loud** - Explain your reasoning
- **Start simple** - Build up complexity gradually
- **Consider trade-offs** - Every decision has pros/cons
- **Think about scale** - How does it handle growth?
- **Plan for failures** - What can go wrong?
- **Monitor everything** - How do you observe the system?

**Good luck with your system design interviews! 🚀**

---

*Last Updated: January 2025*
*Version: 1.0 - Complete System Design Guide*
