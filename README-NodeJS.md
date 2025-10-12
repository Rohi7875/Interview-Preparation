# Node.js Interview Questions & Answers
### For 7+ Years Experienced Developers

---

## Table of Contents
1. [Node.js Basics](#nodejs-basics)
2. [Event Loop & Asynchronous Programming](#event-loop--asynchronous-programming)
3. [Modules & NPM](#modules--npm)
4. [Express.js Framework](#expressjs-framework)
5. [RESTful APIs](#restful-apis)
6. [Authentication & Security](#authentication--security)
7. [Database Integration](#database-integration)
8. [Streams & Buffers](#streams--buffers)
9. [Error Handling](#error-handling)
10. [Testing & Debugging](#testing--debugging)
11. [Performance & Scalability](#performance--scalability)
12. [Microservices & Advanced Patterns](#microservices--advanced-patterns)

---

## Node.js Basics

### Q1. What is Node.js and how is it different from traditional web servers?
**Answer:** Node.js is a JavaScript runtime built on Chrome's V8 engine. It uses an event-driven, non-blocking I/O model, making it lightweight and efficient.

**Key Differences:**
- **Single-threaded** but handles concurrent connections via event loop
- **Non-blocking I/O** - asynchronous operations
- **JavaScript** on both frontend and backend
- **Event-driven architecture**
- **NPM** - largest ecosystem of packages

**Real-time Example:**
```javascript
// Traditional blocking I/O (like PHP/Apache)
const data1 = readFileSync('file1.txt'); // Blocks here
const data2 = readFileSync('file2.txt'); // Blocks here
console.log(data1, data2);

// Node.js non-blocking I/O
const fs = require('fs');

fs.readFile('file1.txt', 'utf8', (err, data1) => {
    if (err) throw err;
    console.log('File 1:', data1);
});

fs.readFile('file2.txt', 'utf8', (err, data2) => {
    if (err) throw err;
    console.log('File 2:', data2);
});

console.log('This executes first!');

// Modern async/await approach
async function readFiles() {
    try {
        const [data1, data2] = await Promise.all([
            fs.promises.readFile('file1.txt', 'utf8'),
            fs.promises.readFile('file2.txt', 'utf8')
        ]);
        console.log('File 1:', data1);
        console.log('File 2:', data2);
    } catch (error) {
        console.error('Error reading files:', error);
    }
}
```

### Q2. Explain the Node.js architecture and V8 engine
**Answer:** Node.js architecture consists of:
- **V8 Engine** - Executes JavaScript code
- **libuv** - Handles async I/O operations and event loop
- **C++ Bindings** - Connect JavaScript to system APIs
- **Node.js APIs** - Built-in modules

**Real-time Example:**
```javascript
// Understanding Node.js architecture
const EventEmitter = require('events');
const fs = require('fs');
const http = require('http');

// 1. Event-driven architecture
class OrderProcessor extends EventEmitter {
    processOrder(order) {
        console.log('Processing order:', order.id);
        
        // Emit events at different stages
        this.emit('orderReceived', order);
        
        setTimeout(() => {
            this.emit('orderProcessed', order);
        }, 1000);
        
        setTimeout(() => {
            this.emit('orderShipped', order);
        }, 2000);
    }
}

const processor = new OrderProcessor();

processor.on('orderReceived', (order) => {
    console.log(`Order ${order.id} received`);
});

processor.on('orderProcessed', (order) => {
    console.log(`Order ${order.id} processed`);
});

processor.on('orderShipped', (order) => {
    console.log(`Order ${order.id} shipped`);
});

processor.processOrder({ id: 12345, items: ['Book', 'Pen'] });

// 2. Understanding the event loop phases
console.log('1. Start');

setTimeout(() => console.log('2. Timeout 0'), 0);
setImmediate(() => console.log('3. Immediate'));

Promise.resolve().then(() => console.log('4. Promise'));

process.nextTick(() => console.log('5. NextTick'));

console.log('6. End');

// Output order: 1, 6, 5, 4, 2, 3
// Explanation:
// - Synchronous code first (1, 6)
// - process.nextTick() (5)
// - Promise microtasks (4)
// - Timer callbacks (2)
// - setImmediate (3)
```

### Q3. What are the latest features in Node.js (v18, v20, v21)?
**Answer:** 

**Node.js 18 LTS (2022):**
- Fetch API built-in
- Web Streams API
- Test runner (experimental)
- V8 10.1

**Node.js 20 LTS (2023):**
- Stable Test Runner
- Permission Model
- Single Executable Applications
- V8 11.3

**Node.js 21 (2023):**
- Stable Fetch and WebStreams
- Built-in WebSocket client
- Import attributes
- Improved diagnostic tools

**Real-time Example:**
```javascript
// Node.js 18+ Native Fetch API
async function fetchUserData(userId) {
    try {
        const response = await fetch(`https://api.example.com/users/${userId}`);
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const data = await response.json();
        return data;
    } catch (error) {
        console.error('Error fetching user:', error);
        throw error;
    }
}

// Node.js 20+ Native Test Runner
import { describe, it } from 'node:test';
import assert from 'node:assert';

describe('User Service', () => {
    it('should create a new user', async () => {
        const user = await createUser({ name: 'John', email: 'john@example.com' });
        assert.strictEqual(user.name, 'John');
        assert.ok(user.id);
    });
    
    it('should validate email format', () => {
        assert.throws(
            () => validateEmail('invalid-email'),
            { message: 'Invalid email format' }
        );
    });
});

// Node.js 20+ Permission Model
// Run with: node --experimental-permission --allow-fs-read=* --allow-fs-write=/tmp app.js
const fs = require('fs');

// This will work with permissions
fs.readFileSync('/etc/hosts');

// This will throw without --allow-fs-write permission
// fs.writeFileSync('/etc/malicious', 'data');

// Node.js 21+ Import Attributes
import data from './config.json' with { type: 'json' };
console.log(data);

// WebSocket Client (Node.js 21+)
const ws = new WebSocket('wss://echo.websocket.org');

ws.addEventListener('open', () => {
    console.log('Connected to WebSocket');
    ws.send('Hello Server!');
});

ws.addEventListener('message', (event) => {
    console.log('Message from server:', event.data);
});
```

---

## Event Loop & Asynchronous Programming

### Q4. Explain the Event Loop in detail
**Answer:** The Event Loop is what allows Node.js to perform non-blocking I/O operations despite JavaScript being single-threaded.

**Event Loop Phases:**
1. **Timers** - setTimeout(), setInterval()
2. **Pending Callbacks** - I/O callbacks deferred to next iteration
3. **Idle, Prepare** - Internal use only
4. **Poll** - Retrieve new I/O events
5. **Check** - setImmediate() callbacks
6. **Close Callbacks** - socket.on('close')

**Real-time Example:**
```javascript
// Understanding Event Loop Phases
console.log('Script start');

// Microtasks (Process.nextTick has highest priority)
process.nextTick(() => {
    console.log('1. nextTick');
});

// Microtasks (Promises)
Promise.resolve()
    .then(() => console.log('2. Promise 1'))
    .then(() => console.log('3. Promise 2'));

// Timer Phase
setTimeout(() => {
    console.log('4. setTimeout 0');
    
    process.nextTick(() => {
        console.log('5. nextTick inside setTimeout');
    });
    
    Promise.resolve().then(() => {
        console.log('6. Promise inside setTimeout');
    });
}, 0);

// Check Phase
setImmediate(() => {
    console.log('7. setImmediate');
    
    process.nextTick(() => {
        console.log('8. nextTick inside setImmediate');
    });
});

// I/O Phase
const fs = require('fs');
fs.readFile(__filename, () => {
    console.log('9. File read callback');
    
    setTimeout(() => console.log('10. setTimeout inside I/O'), 0);
    setImmediate(() => console.log('11. setImmediate inside I/O'));
});

console.log('Script end');

// Real-world example: API call orchestration
class APIOrchestrator {
    constructor() {
        this.cache = new Map();
    }
    
    async fetchWithCache(url) {
        // Check cache first (synchronous)
        if (this.cache.has(url)) {
            return this.cache.get(url);
        }
        
        // Fetch data (asynchronous)
        const data = await fetch(url).then(res => res.json());
        
        // Store in cache
        this.cache.set(url, data);
        
        // Clear cache after 5 minutes
        setTimeout(() => {
            this.cache.delete(url);
        }, 5 * 60 * 1000);
        
        return data;
    }
    
    async fetchMultiple(urls) {
        // Parallel execution using Promise.all
        const results = await Promise.all(
            urls.map(url => this.fetchWithCache(url))
        );
        
        return results;
    }
    
    async fetchSequential(urls) {
        // Sequential execution
        const results = [];
        
        for (const url of urls) {
            const data = await this.fetchWithCache(url);
            results.push(data);
        }
        
        return results;
    }
    
    async fetchWithTimeout(url, timeout = 5000) {
        return Promise.race([
            this.fetchWithCache(url),
            new Promise((_, reject) => 
                setTimeout(() => reject(new Error('Timeout')), timeout)
            )
        ]);
    }
}
```

### Q5. What are Promises, async/await, and how do they work?
**Answer:** Promises represent the eventual completion (or failure) of an asynchronous operation. Async/await is syntactic sugar built on top of Promises.

**Real-time Example:**
```javascript
// 1. Callback Hell (Old way - avoid this)
function getUserData(userId, callback) {
    db.findUser(userId, (err, user) => {
        if (err) return callback(err);
        
        db.findOrders(user.id, (err, orders) => {
            if (err) return callback(err);
            
            db.findProducts(orders, (err, products) => {
                if (err) return callback(err);
                
                callback(null, { user, orders, products });
            });
        });
    });
}

// 2. Promises (Better)
function getUserDataPromise(userId) {
    return db.findUser(userId)
        .then(user => {
            return db.findOrders(user.id)
                .then(orders => {
                    return db.findProducts(orders)
                        .then(products => {
                            return { user, orders, products };
                        });
                });
        });
}

// 3. Async/Await (Best - Most readable)
async function getUserDataAsync(userId) {
    try {
        const user = await db.findUser(userId);
        const orders = await db.findOrders(user.id);
        const products = await db.findProducts(orders);
        
        return { user, orders, products };
    } catch (error) {
        console.error('Error fetching user data:', error);
        throw error;
    }
}

// Real-world example: E-commerce Order Processing
class OrderService {
    async processOrder(orderId) {
        try {
            // 1. Validate order
            const order = await this.validateOrder(orderId);
            
            // 2. Check inventory (parallel)
            const inventoryChecks = await Promise.all(
                order.items.map(item => this.checkInventory(item.productId, item.quantity))
            );
            
            if (inventoryChecks.some(check => !check.available)) {
                throw new Error('Some items are out of stock');
            }
            
            // 3. Process payment
            const payment = await this.processPayment(order);
            
            // 4. Reserve inventory
            await Promise.all(
                order.items.map(item => 
                    this.reserveInventory(item.productId, item.quantity)
                )
            );
            
            // 5. Create shipment
            const shipment = await this.createShipment(order);
            
            // 6. Send notifications (parallel, don't wait)
            Promise.all([
                this.sendEmailNotification(order),
                this.sendSMSNotification(order),
                this.updateAnalytics(order)
            ]).catch(err => console.error('Notification error:', err));
            
            return {
                success: true,
                orderId: order.id,
                paymentId: payment.id,
                shipmentId: shipment.id
            };
            
        } catch (error) {
            // Rollback logic
            await this.rollbackOrder(orderId);
            throw error;
        }
    }
    
    async validateOrder(orderId) {
        const order = await Order.findById(orderId);
        
        if (!order) {
            throw new Error('Order not found');
        }
        
        if (order.status !== 'pending') {
            throw new Error('Order already processed');
        }
        
        return order;
    }
    
    async checkInventory(productId, quantity) {
        const product = await Product.findById(productId);
        
        return {
            productId,
            available: product.stock >= quantity,
            stock: product.stock
        };
    }
    
    async processPayment(order) {
        // Simulate payment processing
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                if (Math.random() > 0.1) { // 90% success rate
                    resolve({
                        id: `pay_${Date.now()}`,
                        amount: order.total,
                        status: 'completed'
                    });
                } else {
                    reject(new Error('Payment failed'));
                }
            }, 1000);
        });
    }
    
    async reserveInventory(productId, quantity) {
        return Product.updateOne(
            { _id: productId },
            { $inc: { stock: -quantity } }
        );
    }
    
    async rollbackOrder(orderId) {
        console.log(`Rolling back order ${orderId}`);
        // Implement rollback logic
    }
}

// Advanced Promise Patterns
class AdvancedPromises {
    // Promise.allSettled - Wait for all, regardless of success/failure
    async fetchAllData() {
        const results = await Promise.allSettled([
            fetch('https://api1.example.com/data'),
            fetch('https://api2.example.com/data'),
            fetch('https://api3.example.com/data')
        ]);
        
        results.forEach((result, index) => {
            if (result.status === 'fulfilled') {
                console.log(`API ${index + 1} succeeded:`, result.value);
            } else {
                console.error(`API ${index + 1} failed:`, result.reason);
            }
        });
        
        return results;
    }
    
    // Promise.race - First one to complete wins
    async fetchWithFastestServer(data) {
        const servers = [
            'https://server1.example.com',
            'https://server2.example.com',
            'https://server3.example.com'
        ];
        
        const result = await Promise.race(
            servers.map(server => 
                fetch(`${server}/api/data`, {
                    method: 'POST',
                    body: JSON.stringify(data)
                }).then(res => res.json())
            )
        );
        
        return result;
    }
    
    // Promise.any - First successful promise
    async fetchFromMirrors(url) {
        const mirrors = [
            `https://mirror1.example.com${url}`,
            `https://mirror2.example.com${url}`,
            `https://mirror3.example.com${url}`
        ];
        
        try {
            const result = await Promise.any(
                mirrors.map(mirror => fetch(mirror))
            );
            return result;
        } catch (error) {
            console.error('All mirrors failed');
            throw error;
        }
    }
    
    // Retry logic with exponential backoff
    async fetchWithRetry(url, maxRetries = 3) {
        for (let i = 0; i < maxRetries; i++) {
            try {
                const response = await fetch(url);
                
                if (response.ok) {
                    return await response.json();
                }
                
                throw new Error(`HTTP ${response.status}`);
            } catch (error) {
                if (i === maxRetries - 1) throw error;
                
                const delay = Math.pow(2, i) * 1000; // Exponential backoff
                console.log(`Retry ${i + 1} after ${delay}ms`);
                await new Promise(resolve => setTimeout(resolve, delay));
            }
        }
    }
}
```

### Q6. What is the difference between process.nextTick() and setImmediate()?
**Answer:** 
- **process.nextTick()** - Executes before any I/O operations, highest priority
- **setImmediate()** - Executes in the Check phase, after I/O operations

**Real-time Example:**
```javascript
// Understanding the difference
console.log('Start');

setImmediate(() => {
    console.log('setImmediate 1');
});

process.nextTick(() => {
    console.log('nextTick 1');
});

Promise.resolve().then(() => {
    console.log('Promise 1');
});

setTimeout(() => {
    console.log('setTimeout 1');
}, 0);

console.log('End');

// Output: Start -> End -> nextTick 1 -> Promise 1 -> setTimeout 1 -> setImmediate 1

// Real-world example: Database transaction handling
class DatabaseTransaction {
    constructor() {
        this.operations = [];
        this.committed = false;
    }
    
    addOperation(operation) {
        if (this.committed) {
            throw new Error('Transaction already committed');
        }
        
        this.operations.push(operation);
        
        // Use nextTick to ensure operation is added before commit
        process.nextTick(() => {
            console.log('Operation queued:', operation.type);
        });
    }
    
    async commit() {
        if (this.committed) {
            throw new Error('Transaction already committed');
        }
        
        this.committed = true;
        
        // Use setImmediate to ensure all nextTick operations complete first
        return new Promise((resolve, reject) => {
            setImmediate(async () => {
                try {
                    for (const operation of this.operations) {
                        await operation.execute();
                    }
                    resolve({ success: true, operations: this.operations.length });
                } catch (error) {
                    reject(error);
                }
            });
        });
    }
}

// Recursive nextTick warning (can block event loop)
let count = 0;

function recursiveNextTick() {
    if (count < 1000000) {
        count++;
        process.nextTick(recursiveNextTick); // This will block the event loop!
    }
}

// Better approach with setImmediate
function recursiveSetImmediate() {
    if (count < 1000000) {
        count++;
        setImmediate(recursiveSetImmediate); // This allows I/O operations
    }
}
```

---

## Modules & NPM

### Q7. Explain CommonJS vs ES6 Modules
**Answer:**

**CommonJS (Traditional Node.js):**
- `require()` for imports
- `module.exports` for exports
- Synchronous loading
- Dynamic imports supported

**ES6 Modules (Modern):**
- `import/export` syntax
- Static analysis possible
- Tree-shaking support
- Asynchronous loading

**Real-time Example:**
```javascript
// ============= CommonJS (users.js) =============
// Exporting
class UserService {
    async getUser(id) {
        return { id, name: 'John' };
    }
}

function validateEmail(email) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

module.exports = {
    UserService,
    validateEmail
};

// Alternative syntax
module.exports.hashPassword = (password) => {
    return crypto.createHash('sha256').update(password).digest('hex');
};

// Importing
const { UserService, validateEmail } = require('./users');
const userService = new UserService();

// Dynamic import
function loadModule(moduleName) {
    if (process.env.NODE_ENV === 'production') {
        return require(`./modules/${moduleName}`);
    }
    return require(`./modules/${moduleName}.dev`);
}

// ============= ES6 Modules (users.mjs or with "type": "module") =============
// Exporting
export class UserService {
    async getUser(id) {
        return { id, name: 'John' };
    }
}

export function validateEmail(email) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

export const hashPassword = (password) => {
    return crypto.createHash('sha256').update(password).digest('hex');
};

// Default export
export default class AuthService {
    async login(email, password) {
        // Login logic
    }
}

// Importing
import AuthService, { UserService, validateEmail } from './users.mjs';
import * as UserModule from './users.mjs';

// Dynamic import (ES6)
async function loadModule(moduleName) {
    const module = await import(`./modules/${moduleName}.mjs`);
    return module;
}

// Real-world example: Modular application structure
// ============= config/database.js =============
import mongoose from 'mongoose';

class Database {
    constructor() {
        this.connection = null;
    }
    
    async connect() {
        try {
            this.connection = await mongoose.connect(process.env.MONGODB_URI, {
                useNewUrlParser: true,
                useUnifiedTopology: true
            });
            console.log('Database connected');
        } catch (error) {
            console.error('Database connection error:', error);
            process.exit(1);
        }
    }
    
    async disconnect() {
        await mongoose.disconnect();
    }
}

export default new Database();

// ============= models/User.js =============
import mongoose from 'mongoose';
import bcrypt from 'bcrypt';

const userSchema = new mongoose.Schema({
    username: { type: String, required: true, unique: true },
    email: { type: String, required: true, unique: true },
    password: { type: String, required: true },
    role: { type: String, enum: ['user', 'admin'], default: 'user' },
    createdAt: { type: Date, default: Date.now }
});

userSchema.pre('save', async function(next) {
    if (!this.isModified('password')) return next();
    this.password = await bcrypt.hash(this.password, 10);
    next();
});

userSchema.methods.comparePassword = async function(password) {
    return await bcrypt.compare(password, this.password);
};

export default mongoose.model('User', userSchema);

// ============= services/UserService.js =============
import User from '../models/User.js';
import { sendEmail } from '../utils/email.js';

export class UserService {
    async createUser(userData) {
        const user = new User(userData);
        await user.save();
        
        // Send welcome email (don't await)
        sendEmail(user.email, 'Welcome!', 'Thanks for joining').catch(console.error);
        
        return user;
    }
    
    async findUserById(id) {
        return await User.findById(id).select('-password');
    }
    
    async updateUser(id, updates) {
        return await User.findByIdAndUpdate(id, updates, { new: true });
    }
    
    async deleteUser(id) {
        return await User.findByIdAndDelete(id);
    }
}

export default new UserService();

// ============= app.js (Main application) =============
import express from 'express';
import database from './config/database.js';
import userService from './services/UserService.js';

const app = express();

// Connect to database
await database.connect();

// Routes
app.post('/api/users', async (req, res) => {
    try {
        const user = await userService.createUser(req.body);
        res.status(201).json(user);
    } catch (error) {
        res.status(400).json({ error: error.message });
    }
});

export default app;
```

This is the first part of the Node.js README. Would you like me to continue with more sections and then create the MongoDB README as well?
