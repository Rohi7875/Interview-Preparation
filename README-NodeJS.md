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

### Q8. Event Emitters
```javascript
const EventEmitter = require('events');
class MyEmitter extends EventEmitter {}
const myEmitter = new MyEmitter();
myEmitter.on('event', () => console.log('Event fired!'));
myEmitter.emit('event');
```

### Q9. Streams
```javascript
const readStream = fs.createReadStream('input.txt');
const writeStream = fs.createWriteStream('output.txt');
readStream.pipe(writeStream);
```

### Q10. Buffers
```javascript
const buf = Buffer.from('Hello World');
console.log(buf.toString('hex'));
console.log(buf.toString('base64'));
```

### Q11. Child Processes
```javascript
const { exec, spawn, fork } = require('child_process');
exec('ls -la', (error, stdout) => console.log(stdout));
```

### Q12. Cluster Module
```javascript
const cluster = require('cluster');
if (cluster.isMaster) {
    for (let i = 0; i < numCPUs; i++) cluster.fork();
} else {
    http.createServer(app).listen(3000);
}
```

### Q13. Error-First Callbacks
```javascript
fs.readFile('file.txt', (err, data) => {
    if (err) return console.error(err);
    console.log(data);
});
```

### Q14. Callback vs Promises
```javascript
// Callback
function getData(callback) {
    setTimeout(() => callback(null, 'data'), 1000);
}
// Promise
function getData() {
    return new Promise(resolve => setTimeout(() => resolve('data'), 1000));
}
```

### Q15. Promise.all vs Promise.race
```javascript
Promise.all([promise1, promise2]); // All must complete
Promise.race([promise1, promise2]); // First to complete
```

### Q16. Async Error Handling
```javascript
async function example() {
    try {
        const result = await asyncOperation();
    } catch (error) {
        console.error(error);
    }
}
```

### Q17. Process Object
```javascript
process.env.NODE_ENV
process.argv
process.cwd()
process.exit(0)
```

### Q18. Global Objects
```javascript
__dirname  // Current directory
__filename // Current file
global     // Global namespace
```

### Q19. Timers
```javascript
setTimeout(fn, 1000);
setInterval(fn, 1000);
setImmediate(fn);
```

### Q20. File System Operations
```javascript
fs.readFile('file.txt', 'utf8', callback);
fs.writeFile('file.txt', data, callback);
fs.unlink('file.txt', callback);
```

### Q21. Path Module
```javascript
path.join('/foo', 'bar');
path.resolve('foo', 'bar');
path.extname('file.txt'); // .txt
```

### Q22. OS Module
```javascript
os.cpus();
os.totalmem();
os.freemem();
os.platform();
```

### Q23. HTTP Module
```javascript
http.createServer((req, res) => {
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('Hello World');
}).listen(3000);
```

### Q24. URL Parsing
```javascript
const url = require('url');
const myURL = new URL('https://example.com/path?query=value');
console.log(myURL.hostname, myURL.pathname);
```

### Q25. Query Strings
```javascript
const querystring = require('querystring');
querystring.parse('foo=bar&abc=xyz');
```

### Q26. Crypto Module
```javascript
const crypto = require('crypto');
const hash = crypto.createHash('sha256').update('password').digest('hex');
```

### Q27. Zlib Compression
```javascript
const zlib = require('zlib');
const gzip = zlib.createGzip();
input.pipe(gzip).pipe(output);
```

### Q28. Worker Threads
```javascript
const { Worker } = require('worker_threads');
const worker = new Worker('./worker.js');
```

### Q29. Debugging
```javascript
node --inspect app.js
console.log(), console.error(), console.trace()
```

### Q30. Memory Management
```javascript
process.memoryUsage();
// { rss, heapTotal, heapUsed, external }
```

### Q31. Performance Hooks
```javascript
const { performance } = require('perf_hooks');
const start = performance.now();
// code
const end = performance.now();
console.log(end - start);
```

### Q32. Assert Module
```javascript
const assert = require('assert');
assert.strictEqual(1, 1);
assert.deepStrictEqual(obj1, obj2);
```

### Q33. Util Module
```javascript
const util = require('util');
const readFile = util.promisify(fs.readFile);
```

### Q34. DNS Module
```javascript
const dns = require('dns');
dns.lookup('example.com', (err, address) => console.log(address));
```

### Q35. Net Module (TCP)
```javascript
const net = require('net');
const server = net.createServer(socket => {
    socket.write('Echo server\r\n');
});
```

### Q36. Readline
```javascript
const readline = require('readline');
const rl = readline.createInterface({input: process.stdin, output: process.stdout});
```

### Q37. Environment Variables
```javascript
require('dotenv').config();
const dbHost = process.env.DB_HOST;
```

### Q38. Package.json Scripts
```json
"scripts": {
  "start": "node app.js",
  "dev": "nodemon app.js",
  "test": "jest"
}
```

### Q39. NPM Commands
```bash
npm install package
npm install --save-dev package
npm update
npm audit fix
```

### Q40. Module Caching
```javascript
require('./module'); // Cached after first require
delete require.cache[require.resolve('./module')]; // Clear cache
```

### Q41. Custom Modules
```javascript
// math.js
module.exports.add = (a, b) => a + b;
// app.js
const math = require('./math');
```

### Q42. HTTP Status Codes
```javascript
res.status(200).json({success: true}); // OK
res.status(201).json({created: true}); // Created
res.status(400).json({error: 'Bad Request'});
res.status(404).json({error: 'Not Found'});
res.status(500).json({error: 'Server Error'});
```

### Q43. Middleware Pattern
```javascript
function logger(req, res, next) {
    console.log(`${req.method} ${req.url}`);
    next();
}
app.use(logger);
```

### Q44. Error Middleware
```javascript
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).send('Something broke!');
});
```

### Q45. CORS
```javascript
const cors = require('cors');
app.use(cors({ origin: 'https://example.com' }));
```

### Q46. Body Parser
```javascript
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
```

### Q47. Static Files
```javascript
app.use(express.static('public'));
app.use('/static', express.static('files'));
```

### Q48. Request Object
```javascript
req.params.id
req.query.search
req.body.username
req.headers['content-type']
req.cookies.session
```

### Q49. Response Methods
```javascript
res.send('text');
res.json({data: 'value'});
res.sendFile('/path/to/file');
res.download('/path/to/file');
res.redirect('/other-page');
```

### Q50. JWT Authentication
```javascript
const jwt = require('jsonwebtoken');
const token = jwt.sign({userId: 123}, SECRET_KEY, {expiresIn: '1h'});
const decoded = jwt.verify(token, SECRET_KEY);
```

### Q51. Password Hashing
```javascript
const bcrypt = require('bcrypt');
const hashed = await bcrypt.hash(password, 10);
const isValid = await bcrypt.compare(password, hashed);
```

### Q52. Session Management
```javascript
const session = require('express-session');
app.use(session({
    secret: 'secret',
    resave: false,
    saveUninitialized: true
}));
```

### Q53. Cookie Parser
```javascript
const cookieParser = require('cookie-parser');
app.use(cookieParser());
res.cookie('name', 'value', {maxAge: 900000, httpOnly: true});
```

### Q54. File Upload (Multer)
```javascript
const multer = require('multer');
const upload = multer({dest: 'uploads/'});
app.post('/upload', upload.single('file'), (req, res) => {
    res.json({file: req.file});
});
```

### Q55. Websockets
```javascript
const io = require('socket.io')(server);
io.on('connection', socket => {
    socket.emit('message', 'Hello');
    socket.on('chat', data => io.emit('chat', data));
});
```

### Q56. Request Validation
```javascript
const {body, validationResult} = require('express-validator');
app.post('/user', [
    body('email').isEmail(),
    body('password').isLength({min: 8})
], (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(400).json({errors: errors.array()});
});
```

### Q57. Rate Limiting
```javascript
const rateLimit = require('express-rate-limit');
const limiter = rateLimit({windowMs: 15*60*1000, max: 100});
app.use('/api/', limiter);
```

### Q58. Helmet (Security)
```javascript
const helmet = require('helmet');
app.use(helmet());
```

### Q59. Compression
```javascript
const compression = require('compression');
app.use(compression());
```

### Q60. Morgan (Logging)
```javascript
const morgan = require('morgan');
app.use(morgan('combined'));
```

### Q61. Dotenv
```javascript
require('dotenv').config();
const port = process.env.PORT || 3000;
```

### Q62. Nodemon (Development)
```json
"scripts": {
  "dev": "nodemon app.js"
}
```

### Q63. Jest Testing
```javascript
test('adds 1 + 2 to equal 3', () => {
    expect(sum(1, 2)).toBe(3);
});
```

### Q64. Supertest (API Testing)
```javascript
const request = require('supertest');
test('GET /users', async () => {
    const response = await request(app).get('/users');
    expect(response.statusCode).toBe(200);
});
```

### Q65. MongoDB Connection
```javascript
const mongoose = require('mongoose');
await mongoose.connect('mongodb://localhost:27017/mydb');
```

### Q66. Mongoose Schema
```javascript
const userSchema = new mongoose.Schema({
    name: String,
    email: {type: String, required: true, unique: true}
});
```

### Q67. Mongoose CRUD
```javascript
await User.create({name: 'John'});
const user = await User.findById(id);
await User.updateOne({_id: id}, {name: 'Jane'});
await User.deleteOne({_id: id});
```

### Q68. Mongoose Populate
```javascript
const user = await User.findById(id).populate('posts');
```

### Q69. Async Iteration
```javascript
for await (const chunk of stream) {
    console.log(chunk);
}
```

### Q70. AbortController
```javascript
const controller = new AbortController();
fetch(url, {signal: controller.signal});
setTimeout(() => controller.abort(), 5000);
```

### Q71. TextEncoder/TextDecoder
```javascript
const encoder = new TextEncoder();
const uint8array = encoder.encode('Hello');
```

### Q72. URL API
```javascript
const myURL = new URL('https://example.com/path?query=value');
myURL.searchParams.get('query');
```

### Q73. FormData
```javascript
const formData = new FormData();
formData.append('file', fileBlob);
```

### Q74. Headers API
```javascript
const headers = new Headers();
headers.append('Content-Type', 'application/json');
```

### Q75. Request/Response Streams
```javascript
request.on('data', chunk => {});
request.on('end', () => {});
```

### Q76. Graceful Shutdown
```javascript
process.on('SIGTERM', () => {
    server.close(() => {
        db.disconnect();
        process.exit(0);
    });
});
```

### Q77. Uncaught Exceptions
```javascript
process.on('uncaughtException', (error) => {
    console.error('Uncaught Exception:', error);
    process.exit(1);
});
```

### Q78. Unhandled Rejections
```javascript
process.on('unhandledRejection', (reason, promise) => {
    console.error('Unhandled Rejection:', reason);
});
```

### Q79. Package-lock.json
```javascript
// Ensures exact dependency versions
// Committed to version control
npm ci // Use in CI/CD
```

### Q80. ESLint Configuration
```json
{
  "extends": "eslint:recommended",
  "env": {"node": true, "es6": true},
  "rules": {"no-console": "warn"}
}
```

---

## Express.js Framework

### Q81. What is Express.js and how does it work?
**Answer:** Express.js is a minimal and flexible Node.js web application framework that provides a robust set of features for web and mobile applications.

**Key Features:**
- **Routing** - Define routes for different HTTP methods
- **Middleware** - Functions that execute during request-response cycle
- **Template Engines** - Pug, EJS, Handlebars
- **Static Files** - Serve static assets
- **Error Handling** - Centralized error handling

**Real-time Example:**
```javascript
const express = require('express');
const app = express();

// Middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Routes
app.get('/api/users', (req, res) => {
    res.json({ users: [] });
});

app.post('/api/users', (req, res) => {
    const { name, email } = req.body;
    // Create user logic
    res.status(201).json({ message: 'User created' });
});

// Error handling middleware
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).json({ error: 'Something went wrong!' });
});

app.listen(3000, () => {
    console.log('Server running on port 3000');
});
```

### Q82. Explain Express.js Middleware
**Answer:** Middleware functions are functions that have access to the request object (req), response object (res), and the next middleware function in the application's request-response cycle.

**Types of Middleware:**
- **Application-level** - `app.use()`
- **Router-level** - `router.use()`
- **Error-handling** - `app.use((err, req, res, next) => {})`
- **Built-in** - `express.json()`, `express.static()`
- **Third-party** - `cors`, `helmet`, `morgan`

**Real-time Example:**
```javascript
// Custom middleware
function logger(req, res, next) {
    console.log(`${new Date().toISOString()} - ${req.method} ${req.url}`);
    next();
}

function auth(req, res, next) {
    const token = req.headers.authorization;
    
    if (!token) {
        return res.status(401).json({ error: 'No token provided' });
    }
    
    try {
        const decoded = jwt.verify(token, process.env.JWT_SECRET);
        req.user = decoded;
        next();
    } catch (error) {
        res.status(401).json({ error: 'Invalid token' });
    }
}

// Rate limiting middleware
function rateLimit(maxRequests = 100, windowMs = 15 * 60 * 1000) {
    const requests = new Map();
    
    return (req, res, next) => {
        const ip = req.ip;
        const now = Date.now();
        const windowStart = now - windowMs;
        
        // Clean old requests
        if (requests.has(ip)) {
            requests.set(ip, requests.get(ip).filter(time => time > windowStart));
        } else {
            requests.set(ip, []);
        }
        
        const userRequests = requests.get(ip);
        
        if (userRequests.length >= maxRequests) {
            return res.status(429).json({ error: 'Too many requests' });
        }
        
        userRequests.push(now);
        next();
    };
}

// Usage
app.use(logger);
app.use(rateLimit(100, 15 * 60 * 1000)); // 100 requests per 15 minutes
app.use('/api/protected', auth);
```

### Q83. Express.js Routing and Route Parameters
**Answer:** Express routing refers to how an application's endpoints respond to client requests.

**Real-time Example:**
```javascript
const express = require('express');
const router = express.Router();

// Route parameters
router.get('/users/:id', (req, res) => {
    const { id } = req.params;
    res.json({ userId: id });
});

// Query parameters
router.get('/search', (req, res) => {
    const { q, page = 1, limit = 10 } = req.query;
    res.json({ 
        query: q, 
        page: parseInt(page), 
        limit: parseInt(limit) 
    });
});

// Multiple route parameters
router.get('/users/:userId/posts/:postId', (req, res) => {
    const { userId, postId } = req.params;
    res.json({ userId, postId });
});

// Route with validation
router.get('/users/:id', (req, res, next) => {
    const { id } = req.params;
    
    if (!/^\d+$/.test(id)) {
        return res.status(400).json({ error: 'Invalid user ID' });
    }
    
    next();
}, (req, res) => {
    res.json({ userId: req.params.id });
});

// Nested routing
const userRouter = express.Router();
const postRouter = express.Router();

userRouter.get('/', (req, res) => {
    res.json({ message: 'Get all users' });
});

postRouter.get('/:id', (req, res) => {
    res.json({ message: `Get post ${req.params.id}` });
});

app.use('/api/users', userRouter);
app.use('/api/posts', postRouter);
```

---

## RESTful APIs

### Q84. What are RESTful APIs and how to design them?
**Answer:** REST (Representational State Transfer) is an architectural style for designing networked applications.

**REST Principles:**
- **Stateless** - Each request contains all necessary information
- **Client-Server** - Separation of concerns
- **Cacheable** - Responses can be cached
- **Uniform Interface** - Consistent resource identification
- **Layered System** - Hierarchical layers

**Real-time Example:**
```javascript
// RESTful API Design
const express = require('express');
const app = express();

// Resource: Users
// GET /api/users - Get all users
app.get('/api/users', async (req, res) => {
    try {
        const users = await User.find();
        res.json({
            success: true,
            data: users,
            count: users.length
        });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// GET /api/users/:id - Get specific user
app.get('/api/users/:id', async (req, res) => {
    try {
        const user = await User.findById(req.params.id);
        if (!user) {
            return res.status(404).json({ error: 'User not found' });
        }
        res.json({ success: true, data: user });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// POST /api/users - Create new user
app.post('/api/users', async (req, res) => {
    try {
        const user = new User(req.body);
        await user.save();
        res.status(201).json({ success: true, data: user });
    } catch (error) {
        res.status(400).json({ error: error.message });
    }
});

// PUT /api/users/:id - Update user (full update)
app.put('/api/users/:id', async (req, res) => {
    try {
        const user = await User.findByIdAndUpdate(
            req.params.id, 
            req.body, 
            { new: true, runValidators: true }
        );
        if (!user) {
            return res.status(404).json({ error: 'User not found' });
        }
        res.json({ success: true, data: user });
    } catch (error) {
        res.status(400).json({ error: error.message });
    }
});

// PATCH /api/users/:id - Update user (partial update)
app.patch('/api/users/:id', async (req, res) => {
    try {
        const user = await User.findByIdAndUpdate(
            req.params.id, 
            { $set: req.body }, 
            { new: true, runValidators: true }
        );
        if (!user) {
            return res.status(404).json({ error: 'User not found' });
        }
        res.json({ success: true, data: user });
    } catch (error) {
        res.status(400).json({ error: error.message });
    }
});

// DELETE /api/users/:id - Delete user
app.delete('/api/users/:id', async (req, res) => {
    try {
        const user = await User.findByIdAndDelete(req.params.id);
        if (!user) {
            return res.status(404).json({ error: 'User not found' });
        }
        res.json({ success: true, message: 'User deleted' });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

### Q85. API Versioning and Documentation
**Answer:** API versioning allows you to maintain backward compatibility while introducing new features.

**Real-time Example:**
```javascript
// API Versioning
const v1Router = express.Router();
const v2Router = express.Router();

// Version 1 API
v1Router.get('/users', (req, res) => {
    res.json({ version: 'v1', users: [] });
});

// Version 2 API with enhanced features
v2Router.get('/users', (req, res) => {
    res.json({ 
        version: 'v2', 
        users: [],
        metadata: {
            pagination: true,
            filtering: true
        }
    });
});

app.use('/api/v1', v1Router);
app.use('/api/v2', v2Router);

// API Documentation with Swagger
const swaggerJSDoc = require('swagger-jsdoc');
const swaggerUi = require('swagger-ui-express');

const swaggerOptions = {
    definition: {
        openapi: '3.0.0',
        info: {
            title: 'User API',
            version: '1.0.0',
            description: 'A simple User API'
        },
        servers: [
            {
                url: 'http://localhost:3000',
                description: 'Development server'
            }
        ]
    },
    apis: ['./routes/*.js']
};

const specs = swaggerJSDoc(swaggerOptions);
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(specs));

// Swagger documentation example
/**
 * @swagger
 * /api/users:
 *   get:
 *     summary: Get all users
 *     tags: [Users]
 *     responses:
 *       200:
 *         description: List of users
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 data:
 *                   type: array
 *                   items:
 *                     $ref: '#/components/schemas/User'
 */
```

---

## Authentication & Security

### Q86. JWT Authentication Implementation
**Answer:** JSON Web Tokens (JWT) are a compact, URL-safe means of representing claims to be transferred between two parties.

**Real-time Example:**
```javascript
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');

// JWT Configuration
const JWT_SECRET = process.env.JWT_SECRET || 'your-secret-key';
const JWT_EXPIRES_IN = '24h';
const REFRESH_TOKEN_EXPIRES_IN = '7d';

class AuthService {
    // Generate JWT token
    generateToken(payload) {
        return jwt.sign(payload, JWT_SECRET, { 
            expiresIn: JWT_EXPIRES_IN 
        });
    }
    
    // Generate refresh token
    generateRefreshToken(payload) {
        return jwt.sign(payload, JWT_SECRET, { 
            expiresIn: REFRESH_TOKEN_EXPIRES_IN 
        });
    }
    
    // Verify JWT token
    verifyToken(token) {
        try {
            return jwt.verify(token, JWT_SECRET);
        } catch (error) {
            throw new Error('Invalid token');
        }
    }
    
    // Hash password
    async hashPassword(password) {
        const saltRounds = 12;
        return await bcrypt.hash(password, saltRounds);
    }
    
    // Compare password
    async comparePassword(password, hashedPassword) {
        return await bcrypt.compare(password, hashedPassword);
    }
}

// Authentication middleware
const authMiddleware = (req, res, next) => {
    try {
        const authHeader = req.headers.authorization;
        
        if (!authHeader || !authHeader.startsWith('Bearer ')) {
            return res.status(401).json({ error: 'No token provided' });
        }
        
        const token = authHeader.substring(7);
        const decoded = jwt.verify(token, JWT_SECRET);
        
        req.user = decoded;
        next();
    } catch (error) {
        res.status(401).json({ error: 'Invalid token' });
    }
};

// Role-based authorization
const authorize = (...roles) => {
    return (req, res, next) => {
        if (!req.user) {
            return res.status(401).json({ error: 'Authentication required' });
        }
        
        if (!roles.includes(req.user.role)) {
            return res.status(403).json({ error: 'Insufficient permissions' });
        }
        
        next();
    };
};

// Login endpoint
app.post('/api/auth/login', async (req, res) => {
    try {
        const { email, password } = req.body;
        
        // Find user
        const user = await User.findOne({ email });
        if (!user) {
            return res.status(401).json({ error: 'Invalid credentials' });
        }
        
        // Check password
        const isValidPassword = await bcrypt.compare(password, user.password);
        if (!isValidPassword) {
            return res.status(401).json({ error: 'Invalid credentials' });
        }
        
        // Generate tokens
        const token = jwt.sign(
            { userId: user._id, email: user.email, role: user.role },
            JWT_SECRET,
            { expiresIn: JWT_EXPIRES_IN }
        );
        
        const refreshToken = jwt.sign(
            { userId: user._id },
            JWT_SECRET,
            { expiresIn: REFRESH_TOKEN_EXPIRES_IN }
        );
        
        // Store refresh token
        await RefreshToken.create({
            token: refreshToken,
            userId: user._id,
            expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)
        });
        
        res.json({
            success: true,
            token,
            refreshToken,
            user: {
                id: user._id,
                email: user.email,
                role: user.role
            }
        });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// Protected route
app.get('/api/profile', authMiddleware, (req, res) => {
    res.json({ user: req.user });
});

// Admin only route
app.delete('/api/users/:id', authMiddleware, authorize('admin'), async (req, res) => {
    // Delete user logic
});
```

### Q87. Security Best Practices
**Answer:** Security is crucial for Node.js applications to prevent common vulnerabilities.

**Real-time Example:**
```javascript
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const cors = require('cors');
const validator = require('validator');

// Security middleware setup
app.use(helmet({
    contentSecurityPolicy: {
        directives: {
            defaultSrc: ["'self'"],
            styleSrc: ["'self'", "'unsafe-inline'"],
            scriptSrc: ["'self'"],
            imgSrc: ["'self'", "data:", "https:"]
        }
    }
}));

// CORS configuration
app.use(cors({
    origin: process.env.ALLOWED_ORIGINS?.split(',') || ['http://localhost:3000'],
    credentials: true,
    methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
    allowedHeaders: ['Content-Type', 'Authorization']
}));

// Rate limiting
const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // limit each IP to 100 requests per windowMs
    message: 'Too many requests from this IP',
    standardHeaders: true,
    legacyHeaders: false
});

app.use('/api/', limiter);

// Input validation and sanitization
const validateUser = (req, res, next) => {
    const { email, password, name } = req.body;
    
    // Email validation
    if (!validator.isEmail(email)) {
        return res.status(400).json({ error: 'Invalid email format' });
    }
    
    // Password strength validation
    if (!validator.isLength(password, { min: 8 })) {
        return res.status(400).json({ error: 'Password must be at least 8 characters' });
    }
    
    // Sanitize inputs
    req.body.email = validator.normalizeEmail(email);
    req.body.name = validator.escape(name);
    
    next();
};

// SQL Injection prevention (using parameterized queries)
app.get('/api/users', async (req, res) => {
    try {
        const { search } = req.query;
        
        // Safe query with parameterized values
        const users = await User.find({
            $or: [
                { name: { $regex: search, $options: 'i' } },
                { email: { $regex: search, $options: 'i' } }
            ]
        });
        
        res.json({ users });
    } catch (error) {
        res.status(500).json({ error: 'Internal server error' });
    }
});

// XSS Protection
app.use((req, res, next) => {
    // Remove script tags from all inputs
    const sanitizeInput = (obj) => {
        for (let key in obj) {
            if (typeof obj[key] === 'string') {
                obj[key] = obj[key].replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '');
            } else if (typeof obj[key] === 'object' && obj[key] !== null) {
                sanitizeInput(obj[key]);
            }
        }
    };
    
    if (req.body) sanitizeInput(req.body);
    if (req.query) sanitizeInput(req.query);
    
    next();
});

// CSRF Protection
const csrf = require('csurf');
const csrfProtection = csrf({ cookie: true });

app.use(csrfProtection);

// Session security
app.use(session({
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    cookie: {
        secure: process.env.NODE_ENV === 'production',
        httpOnly: true,
        maxAge: 24 * 60 * 60 * 1000 // 24 hours
    }
}));
```

---

## Database Integration

### Q88. MongoDB Integration with Mongoose
**Answer:** Mongoose is an Object Data Modeling (ODM) library for MongoDB and Node.js.

**Real-time Example:**
```javascript
const mongoose = require('mongoose');

// Database connection
const connectDB = async () => {
    try {
        await mongoose.connect(process.env.MONGODB_URI, {
            useNewUrlParser: true,
            useUnifiedTopology: true,
            maxPoolSize: 10,
            serverSelectionTimeoutMS: 5000,
            socketTimeoutMS: 45000,
        });
        console.log('MongoDB connected');
    } catch (error) {
        console.error('Database connection error:', error);
        process.exit(1);
    }
};

// User Schema
const userSchema = new mongoose.Schema({
    username: {
        type: String,
        required: true,
        unique: true,
        trim: true,
        minlength: 3,
        maxlength: 30
    },
    email: {
        type: String,
        required: true,
        unique: true,
        lowercase: true,
        match: [/^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/, 'Invalid email']
    },
    password: {
        type: String,
        required: true,
        minlength: 8
    },
    role: {
        type: String,
        enum: ['user', 'admin', 'moderator'],
        default: 'user'
    },
    profile: {
        firstName: String,
        lastName: String,
        avatar: String,
        bio: String
    },
    isActive: {
        type: Boolean,
        default: true
    },
    lastLogin: Date,
    createdAt: {
        type: Date,
        default: Date.now
    }
}, {
    timestamps: true,
    toJSON: { virtuals: true },
    toObject: { virtuals: true }
});

// Virtual fields
userSchema.virtual('fullName').get(function() {
    return `${this.profile.firstName} ${this.profile.lastName}`;
});

// Indexes
userSchema.index({ email: 1 });
userSchema.index({ username: 1 });
userSchema.index({ createdAt: -1 });

// Pre-save middleware
userSchema.pre('save', async function(next) {
    if (!this.isModified('password')) return next();
    
    try {
        const salt = await bcrypt.genSalt(12);
        this.password = await bcrypt.hash(this.password, salt);
        next();
    } catch (error) {
        next(error);
    }
});

// Instance methods
userSchema.methods.comparePassword = async function(candidatePassword) {
    return await bcrypt.compare(candidatePassword, this.password);
};

userSchema.methods.toSafeObject = function() {
    const userObject = this.toObject();
    delete userObject.password;
    return userObject;
};

// Static methods
userSchema.statics.findByEmail = function(email) {
    return this.findOne({ email: email.toLowerCase() });
};

userSchema.statics.findActiveUsers = function() {
    return this.find({ isActive: true });
};

// User Model
const User = mongoose.model('User', userSchema);

// Advanced queries
class UserService {
    async createUser(userData) {
        try {
            const user = new User(userData);
            await user.save();
            return user.toSafeObject();
        } catch (error) {
            if (error.code === 11000) {
                throw new Error('Email or username already exists');
            }
            throw error;
        }
    }
    
    async findUsersWithPagination(page = 1, limit = 10, filters = {}) {
        const skip = (page - 1) * limit;
        
        const query = User.find(filters)
            .select('-password')
            .sort({ createdAt: -1 })
            .skip(skip)
            .limit(limit);
        
        const [users, total] = await Promise.all([
            query.exec(),
            User.countDocuments(filters)
        ]);
        
        return {
            users,
            pagination: {
                page,
                limit,
                total,
                pages: Math.ceil(total / limit)
            }
        };
    }
    
    async searchUsers(searchTerm, page = 1, limit = 10) {
        const regex = new RegExp(searchTerm, 'i');
        
        const query = User.find({
            $or: [
                { username: regex },
                { email: regex },
                { 'profile.firstName': regex },
                { 'profile.lastName': regex }
            ]
        }).select('-password');
        
        const users = await query
            .sort({ createdAt: -1 })
            .skip((page - 1) * limit)
            .limit(limit)
            .exec();
        
        return users;
    }
    
    async updateUserProfile(userId, updateData) {
        const allowedFields = ['profile.firstName', 'profile.lastName', 'profile.bio', 'profile.avatar'];
        const update = {};
        
        Object.keys(updateData).forEach(key => {
            if (allowedFields.includes(key)) {
                update[key] = updateData[key];
            }
        });
        
        return await User.findByIdAndUpdate(
            userId,
            { $set: update },
            { new: true, runValidators: true }
        ).select('-password');
    }
}
```

### Q89. MySQL Integration with Sequelize
**Answer:** Sequelize is a promise-based Node.js ORM for Postgres, MySQL, MariaDB, SQLite, and Microsoft SQL Server.

**Real-time Example:**
```javascript
const { Sequelize, DataTypes } = require('sequelize');

// Database connection
const sequelize = new Sequelize(process.env.DATABASE_URL, {
    dialect: 'mysql',
    logging: process.env.NODE_ENV === 'development' ? console.log : false,
    pool: {
        max: 5,
        min: 0,
        acquire: 30000,
        idle: 10000
    }
});

// User Model
const User = sequelize.define('User', {
    id: {
        type: DataTypes.INTEGER,
        primaryKey: true,
        autoIncrement: true
    },
    username: {
        type: DataTypes.STRING(50),
        allowNull: false,
        unique: true,
        validate: {
            len: [3, 50]
        }
    },
    email: {
        type: DataTypes.STRING(100),
        allowNull: false,
        unique: true,
        validate: {
            isEmail: true
        }
    },
    password: {
        type: DataTypes.STRING(255),
        allowNull: false
    },
    role: {
        type: DataTypes.ENUM('user', 'admin', 'moderator'),
        defaultValue: 'user'
    },
    isActive: {
        type: DataTypes.BOOLEAN,
        defaultValue: true
    }
}, {
    tableName: 'users',
    timestamps: true,
    paranoid: true // Soft delete
});

// Post Model
const Post = sequelize.define('Post', {
    id: {
        type: DataTypes.INTEGER,
        primaryKey: true,
        autoIncrement: true
    },
    title: {
        type: DataTypes.STRING(200),
        allowNull: false
    },
    content: {
        type: DataTypes.TEXT,
        allowNull: false
    },
    published: {
        type: DataTypes.BOOLEAN,
        defaultValue: false
    }
}, {
    tableName: 'posts'
});

// Define associations
User.hasMany(Post, { foreignKey: 'authorId', as: 'posts' });
Post.belongsTo(User, { foreignKey: 'authorId', as: 'author' });

// User Service
class UserService {
    async createUser(userData) {
        try {
            const user = await User.create(userData);
            return user.toJSON();
        } catch (error) {
            if (error.name === 'SequelizeUniqueConstraintError') {
                throw new Error('Email or username already exists');
            }
            throw error;
        }
    }
    
    async findUserById(id) {
        const user = await User.findByPk(id, {
            attributes: { exclude: ['password'] },
            include: [{
                model: Post,
                as: 'posts',
                where: { published: true },
                required: false
            }]
        });
        
        return user;
    }
    
    async findUsersWithPosts(page = 1, limit = 10) {
        const offset = (page - 1) * limit;
        
        const { count, rows } = await User.findAndCountAll({
            attributes: { exclude: ['password'] },
            include: [{
                model: Post,
                as: 'posts',
                where: { published: true },
                required: false
            }],
            limit,
            offset,
            order: [['createdAt', 'DESC']]
        });
        
        return {
            users: rows,
            pagination: {
                page,
                limit,
                total: count,
                pages: Math.ceil(count / limit)
            }
        };
    }
    
    async updateUser(id, updateData) {
        const [affectedRows] = await User.update(updateData, {
            where: { id },
            returning: true
        });
        
        if (affectedRows === 0) {
            throw new Error('User not found');
        }
        
        return await User.findByPk(id, {
            attributes: { exclude: ['password'] }
        });
    }
    
    async deleteUser(id) {
        const user = await User.findByPk(id);
        if (!user) {
            throw new Error('User not found');
        }
        
        await user.destroy(); // Soft delete
        return { message: 'User deleted successfully' };
    }
}
```

---

## Streams & Buffers

### Q90. Understanding Node.js Streams
**Answer:** Streams are objects that let you read data from a source or write data to a destination in a continuous fashion.

**Types of Streams:**
- **Readable** - Can read data from
- **Writable** - Can write data to
- **Duplex** - Both readable and writable
- **Transform** - Duplex stream that can modify data

**Real-time Example:**
```javascript
const fs = require('fs');
const { Transform, PassThrough } = require('stream');
const zlib = require('zlib');

// File streaming with error handling
function streamFile(inputPath, outputPath) {
    const readStream = fs.createReadStream(inputPath, { encoding: 'utf8' });
    const writeStream = fs.createWriteStream(outputPath);
    
    readStream.on('error', (error) => {
        console.error('Read error:', error);
    });
    
    writeStream.on('error', (error) => {
        console.error('Write error:', error);
    });
    
    readStream.pipe(writeStream);
    
    writeStream.on('finish', () => {
        console.log('File streaming completed');
    });
}

// Custom Transform stream for data processing
class DataProcessor extends Transform {
    constructor(options = {}) {
        super({ objectMode: true });
        this.batchSize = options.batchSize || 100;
        this.batch = [];
    }
    
    _transform(chunk, encoding, callback) {
        // Process each chunk
        const processedChunk = this.processChunk(chunk);
        this.batch.push(processedChunk);
        
        // Emit batch when it reaches the size limit
        if (this.batch.length >= this.batchSize) {
            this.push(this.batch);
            this.batch = [];
        }
        
        callback();
    }
    
    _flush(callback) {
        // Emit remaining items
        if (this.batch.length > 0) {
            this.push(this.batch);
        }
        callback();
    }
    
    processChunk(chunk) {
        // Add processing logic here
        return {
            ...chunk,
            processedAt: new Date(),
            processed: true
        };
    }
}

// Real-time data processing pipeline
function createDataPipeline() {
    const processor = new DataProcessor({ batchSize: 50 });
    const compressor = zlib.createGzip();
    const output = fs.createWriteStream('processed-data.gz');
    
    // Create pipeline
    processor
        .pipe(compressor)
        .pipe(output)
        .on('error', (error) => {
            console.error('Pipeline error:', error);
        });
    
    return processor;
}

// HTTP streaming response
app.get('/api/large-data', (req, res) => {
    res.setHeader('Content-Type', 'application/json');
    res.setHeader('Transfer-Encoding', 'chunked');
    
    const dataStream = fs.createReadStream('large-data.json');
    
    dataStream.on('data', (chunk) => {
        res.write(chunk);
    });
    
    dataStream.on('end', () => {
        res.end();
    });
    
    dataStream.on('error', (error) => {
        res.status(500).json({ error: 'Stream error' });
    });
});

// WebSocket streaming
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', (ws) => {
    const dataStream = fs.createReadStream('realtime-data.json');
    
    dataStream.on('data', (chunk) => {
        if (ws.readyState === WebSocket.OPEN) {
            ws.send(JSON.stringify({
                type: 'data',
                payload: chunk.toString()
            }));
        }
    });
    
    ws.on('close', () => {
        dataStream.destroy();
    });
});
```

### Q91. Buffer Operations and Memory Management
**Answer:** Buffers are used to handle binary data in Node.js.

**Real-time Example:**
```javascript
// Buffer creation and manipulation
function bufferOperations() {
    // Create buffers
    const buf1 = Buffer.alloc(10); // 10 bytes filled with zeros
    const buf2 = Buffer.allocUnsafe(10); // 10 bytes, may contain old data
    const buf3 = Buffer.from('Hello World', 'utf8');
    const buf4 = Buffer.from([1, 2, 3, 4, 5]);
    
    // Buffer operations
    console.log('Buffer length:', buf3.length);
    console.log('Buffer as string:', buf3.toString());
    console.log('Buffer as hex:', buf3.toString('hex'));
    console.log('Buffer as base64:', buf3.toString('base64'));
    
    // Buffer slicing (creates new buffer, shares memory)
    const slice = buf3.slice(0, 5);
    console.log('Slice:', slice.toString());
    
    // Buffer copying
    const copy = Buffer.alloc(5);
    buf3.copy(copy, 0, 0, 5);
    console.log('Copy:', copy.toString());
    
    // Buffer comparison
    const buf5 = Buffer.from('Hello');
    const buf6 = Buffer.from('Hello');
    console.log('Buffers equal:', buf5.equals(buf6));
    console.log('Buffer compare:', buf5.compare(buf6));
}

// Memory-efficient file processing
function processLargeFile(filePath) {
    const CHUNK_SIZE = 64 * 1024; // 64KB chunks
    const readStream = fs.createReadStream(filePath, { 
        highWaterMark: CHUNK_SIZE 
    });
    
    let totalBytes = 0;
    let chunkCount = 0;
    
    readStream.on('data', (chunk) => {
        totalBytes += chunk.length;
        chunkCount++;
        
        // Process chunk
        processChunk(chunk);
        
        // Log progress
        if (chunkCount % 100 === 0) {
            console.log(`Processed ${chunkCount} chunks, ${totalBytes} bytes`);
        }
    });
    
    readStream.on('end', () => {
        console.log(`File processing complete: ${totalBytes} bytes in ${chunkCount} chunks`);
    });
    
    readStream.on('error', (error) => {
        console.error('Stream error:', error);
    });
}

function processChunk(chunk) {
    // Example: Find specific patterns in binary data
    const pattern = Buffer.from('TODO');
    let index = 0;
    
    while ((index = chunk.indexOf(pattern, index)) !== -1) {
        console.log(`Found pattern at position: ${index}`);
        index += pattern.length;
    }
}

// Base64 encoding/decoding
function base64Operations() {
    const text = 'Hello World!';
    
    // Encode to base64
    const encoded = Buffer.from(text, 'utf8').toString('base64');
    console.log('Encoded:', encoded);
    
    // Decode from base64
    const decoded = Buffer.from(encoded, 'base64').toString('utf8');
    console.log('Decoded:', decoded);
}

// Binary data manipulation
function binaryOperations() {
    // Create a buffer with specific values
    const buffer = Buffer.alloc(8);
    
    // Write different data types
    buffer.writeUInt32BE(123456789, 0); // Big-endian 32-bit integer
    buffer.writeUInt32LE(987654321, 4); // Little-endian 32-bit integer
    
    console.log('Buffer as hex:', buffer.toString('hex'));
    
    // Read back the values
    const value1 = buffer.readUInt32BE(0);
    const value2 = buffer.readUInt32LE(4);
    
    console.log('Value 1 (BE):', value1);
    console.log('Value 2 (LE):', value2);
}
```

---

## Error Handling

### Q92. Comprehensive Error Handling Strategies
**Answer:** Proper error handling is crucial for robust Node.js applications.

**Real-time Example:**
```javascript
// Custom Error Classes
class AppError extends Error {
    constructor(message, statusCode) {
        super(message);
        this.statusCode = statusCode;
        this.status = `${statusCode}`.startsWith('4') ? 'fail' : 'error';
        this.isOperational = true;
        
        Error.captureStackTrace(this, this.constructor);
    }
}

class ValidationError extends AppError {
    constructor(message) {
        super(message, 400);
        this.name = 'ValidationError';
    }
}

class NotFoundError extends AppError {
    constructor(resource) {
        super(`${resource} not found`, 404);
        this.name = 'NotFoundError';
    }
}

// Global Error Handler
const globalErrorHandler = (err, req, res, next) => {
    err.statusCode = err.statusCode || 500;
    err.status = err.status || 'error';
    
    // Log error
    console.error('Error:', {
        message: err.message,
        stack: err.stack,
        url: req.url,
        method: req.method,
        ip: req.ip,
        userAgent: req.get('User-Agent')
    });
    
    // Send error response
    if (process.env.NODE_ENV === 'development') {
        res.status(err.statusCode).json({
            status: err.status,
            error: err,
            message: err.message,
            stack: err.stack
        });
    } else {
        // Production error response
        if (err.isOperational) {
            res.status(err.statusCode).json({
                status: err.status,
                message: err.message
            });
        } else {
            // Programming or unknown error
            res.status(500).json({
                status: 'error',
                message: 'Something went wrong!'
            });
        }
    }
};

// Async Error Wrapper
const catchAsync = (fn) => {
    return (req, res, next) => {
        fn(req, res, next).catch(next);
    };
};

// Usage examples
app.get('/api/users/:id', catchAsync(async (req, res, next) => {
    const user = await User.findById(req.params.id);
    
    if (!user) {
        return next(new NotFoundError('User'));
    }
    
    res.json({ success: true, data: user });
}));

// Validation middleware
const validateUser = (req, res, next) => {
    const { email, password } = req.body;
    const errors = [];
    
    if (!email || !validator.isEmail(email)) {
        errors.push('Valid email is required');
    }
    
    if (!password || password.length < 8) {
        errors.push('Password must be at least 8 characters');
    }
    
    if (errors.length > 0) {
        return next(new ValidationError(errors.join(', ')));
    }
    
    next();
};

// Database error handling
const handleDatabaseError = (error) => {
    if (error.code === 11000) {
        return new AppError('Duplicate field value', 400);
    }
    
    if (error.name === 'ValidationError') {
        const message = Object.values(error.errors).map(err => err.message).join(', ');
        return new AppError(message, 400);
    }
    
    if (error.name === 'CastError') {
        return new AppError('Invalid ID format', 400);
    }
    
    return error;
};

// Process error handlers
process.on('uncaughtException', (err) => {
    console.error('Uncaught Exception:', err);
    process.exit(1);
});

process.on('unhandledRejection', (reason, promise) => {
    console.error('Unhandled Rejection at:', promise, 'reason:', reason);
    process.exit(1);
});

// Graceful shutdown
const gracefulShutdown = (server) => {
    return (signal) => {
        console.log(`Received ${signal}. Shutting down gracefully...`);
        
        server.close(() => {
            console.log('Process terminated');
            process.exit(0);
        });
        
        // Force close after 30 seconds
        setTimeout(() => {
            console.error('Could not close connections in time, forcefully shutting down');
            process.exit(1);
        }, 30000);
    };
};

// Apply error handling
app.use(globalErrorHandler);
```

---

## Testing & Debugging

### Q93. Comprehensive Testing with Jest
**Answer:** Testing ensures code quality and prevents regressions.

**Real-time Example:**
```javascript
// User Service Tests
describe('UserService', () => {
    let userService;
    let mockUser;
    
    beforeEach(() => {
        userService = new UserService();
        mockUser = {
            id: 1,
            username: 'testuser',
            email: 'test@example.com',
            password: 'hashedpassword'
        };
    });
    
    afterEach(() => {
        jest.clearAllMocks();
    });
    
    describe('createUser', () => {
        it('should create a new user successfully', async () => {
            // Arrange
            const userData = {
                username: 'newuser',
                email: 'newuser@example.com',
                password: 'password123'
            };
            
            User.create = jest.fn().mockResolvedValue(mockUser);
            
            // Act
            const result = await userService.createUser(userData);
            
            // Assert
            expect(User.create).toHaveBeenCalledWith(userData);
            expect(result).toEqual(mockUser);
        });
        
        it('should throw error for duplicate email', async () => {
            // Arrange
            const userData = {
                username: 'newuser',
                email: 'test@example.com',
                password: 'password123'
            };
            
            const duplicateError = new Error('Duplicate email');
            duplicateError.code = 11000;
            User.create = jest.fn().mockRejectedValue(duplicateError);
            
            // Act & Assert
            await expect(userService.createUser(userData))
                .rejects
                .toThrow('Email or username already exists');
        });
    });
    
    describe('findUserById', () => {
        it('should return user when found', async () => {
            // Arrange
            User.findById = jest.fn().mockResolvedValue(mockUser);
            
            // Act
            const result = await userService.findUserById(1);
            
            // Assert
            expect(User.findById).toHaveBeenCalledWith(1);
            expect(result).toEqual(mockUser);
        });
        
        it('should return null when user not found', async () => {
            // Arrange
            User.findById = jest.fn().mockResolvedValue(null);
            
            // Act
            const result = await userService.findUserById(999);
            
            // Assert
            expect(result).toBeNull();
        });
    });
});

// API Integration Tests
describe('User API', () => {
    let app;
    let server;
    
    beforeAll(async () => {
        app = require('../app');
        server = app.listen(0);
    });
    
    afterAll(async () => {
        await server.close();
    });
    
    describe('POST /api/users', () => {
        it('should create a new user', async () => {
            const userData = {
                username: 'testuser',
                email: 'test@example.com',
                password: 'password123'
            };
            
            const response = await request(app)
                .post('/api/users')
                .send(userData)
                .expect(201);
            
            expect(response.body.success).toBe(true);
            expect(response.body.data.username).toBe(userData.username);
        });
        
        it('should return 400 for invalid data', async () => {
            const invalidData = {
                username: 'ab', // Too short
                email: 'invalid-email',
                password: '123' // Too short
            };
            
            const response = await request(app)
                .post('/api/users')
                .send(invalidData)
                .expect(400);
            
            expect(response.body.error).toBeDefined();
        });
    });
    
    describe('GET /api/users/:id', () => {
        it('should return user by id', async () => {
            const response = await request(app)
                .get('/api/users/1')
                .expect(200);
            
            expect(response.body.success).toBe(true);
            expect(response.body.data).toBeDefined();
        });
        
        it('should return 404 for non-existent user', async () => {
            await request(app)
                .get('/api/users/999')
                .expect(404);
        });
    });
});

// Mock and Stub Examples
describe('External API Integration', () => {
    let userService;
    
    beforeEach(() => {
        userService = new UserService();
    });
    
    it('should handle external API failures gracefully', async () => {
        // Mock external API to return error
        const mockFetch = jest.fn().mockRejectedValue(new Error('API Error'));
        global.fetch = mockFetch;
        
        // Mock console.error to avoid test output
        const consoleSpy = jest.spyOn(console, 'error').mockImplementation();
        
        // Act
        await userService.fetchExternalData();
        
        // Assert
        expect(mockFetch).toHaveBeenCalled();
        expect(consoleSpy).toHaveBeenCalledWith('External API error:', expect.any(Error));
        
        // Cleanup
        consoleSpy.mockRestore();
    });
    
    it('should retry failed requests', async () => {
        let callCount = 0;
        const mockFetch = jest.fn().mockImplementation(() => {
            callCount++;
            if (callCount < 3) {
                return Promise.reject(new Error('Network error'));
            }
            return Promise.resolve({ ok: true, json: () => Promise.resolve({ data: 'success' }) });
        });
        global.fetch = mockFetch;
        
        const result = await userService.fetchWithRetry('https://api.example.com/data');
        
        expect(callCount).toBe(3);
        expect(result).toEqual({ data: 'success' });
    });
});
```

### Q94. Debugging Techniques and Tools
**Answer:** Effective debugging is essential for development and production issues.

**Real-time Example:**
```javascript
// Debugging with console methods
function debugExample() {
    const data = { user: 'john', items: [1, 2, 3] };
    
    // Basic logging
    console.log('Basic log:', data);
    
    // Structured logging
    console.log('User data:', JSON.stringify(data, null, 2));
    
    // Debug with timing
    console.time('operation');
    // Some operation
    console.timeEnd('operation');
    
    // Debug with stack trace
    console.trace('Function call trace');
    
    // Conditional logging
    if (process.env.NODE_ENV === 'development') {
        console.debug('Debug info:', data);
    }
}

// Advanced debugging with debug module
const debug = require('debug');
const dbDebug = debug('app:database');
const apiDebug = debug('app:api');

function databaseOperation() {
    dbDebug('Starting database operation');
    
    // Simulate database work
    setTimeout(() => {
        dbDebug('Database operation completed');
    }, 1000);
}

function apiCall() {
    apiDebug('Making API call to external service');
    
    // Simulate API call
    setTimeout(() => {
        apiDebug('API call completed');
    }, 500);
}

// Performance debugging
const { performance } = require('perf_hooks');

function performanceDebugging() {
    const start = performance.now();
    
    // Simulate work
    setTimeout(() => {
        const end = performance.now();
        console.log(`Operation took ${end - start} milliseconds`);
    }, 100);
}

// Memory debugging
function memoryDebugging() {
    const memBefore = process.memoryUsage();
    
    // Create some objects
    const largeArray = new Array(1000000).fill('data');
    
    const memAfter = process.memoryUsage();
    
    console.log('Memory before:', memBefore);
    console.log('Memory after:', memAfter);
    console.log('Memory difference:', {
        rss: memAfter.rss - memBefore.rss,
        heapUsed: memAfter.heapUsed - memBefore.heapUsed
    });
}

// Error debugging with stack traces
function errorDebugging() {
    try {
        throw new Error('Something went wrong');
    } catch (error) {
        console.error('Error occurred:', {
            message: error.message,
            stack: error.stack,
            name: error.name
        });
    }
}

// Async debugging
async function asyncDebugging() {
    console.log('Starting async operation');
    
    try {
        const result = await someAsyncOperation();
        console.log('Async operation completed:', result);
    } catch (error) {
        console.error('Async operation failed:', error);
    }
}

// Debugging with breakpoints (using node --inspect)
function debuggingWithBreakpoints() {
    debugger; // This will pause execution when using --inspect
    
    const data = processData();
    return data;
}
```

---

## Performance & Scalability

### Q95. Performance Optimization Techniques
**Answer:** Performance optimization is crucial for scalable Node.js applications.

**Real-time Example:**
```javascript
// Caching strategies
const NodeCache = require('node-cache');
const cache = new NodeCache({ stdTTL: 600 }); // 10 minutes

class CacheService {
    async get(key) {
        const cached = cache.get(key);
        if (cached) {
            console.log('Cache hit for:', key);
            return cached;
        }
        return null;
    }
    
    set(key, value, ttl = 600) {
        cache.set(key, value, ttl);
    }
    
    async getOrSet(key, fetchFunction, ttl = 600) {
        let value = await this.get(key);
        
        if (!value) {
            console.log('Cache miss for:', key);
            value = await fetchFunction();
            this.set(key, value, ttl);
        }
        
        return value;
    }
}

// Database query optimization
class OptimizedUserService {
    async findUsersWithOptimization(filters = {}, page = 1, limit = 10) {
        const skip = (page - 1) * limit;
        
        // Use projection to limit fields
        const projection = {
            username: 1,
            email: 1,
            createdAt: 1,
            _id: 1
        };
        
        // Use indexes for sorting
        const sort = { createdAt: -1 };
        
        // Build efficient query
        const query = {};
        if (filters.search) {
            query.$or = [
                { username: { $regex: filters.search, $options: 'i' } },
                { email: { $regex: filters.search, $options: 'i' } }
            ];
        }
        
        const [users, total] = await Promise.all([
            User.find(query, projection)
                .sort(sort)
                .skip(skip)
                .limit(limit)
                .lean() // Return plain objects instead of Mongoose documents
                .exec(),
            User.countDocuments(query)
        ]);
        
        return { users, total, page, limit };
    }
}

// Connection pooling
const mysql = require('mysql2/promise');

const pool = mysql.createPool({
    host: process.env.DB_HOST,
    user: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
    waitForConnections: true,
    connectionLimit: 10,
    queueLimit: 0,
    acquireTimeout: 60000,
    timeout: 60000
});

// Memory optimization
class MemoryOptimizedService {
    constructor() {
        this.cache = new Map();
        this.maxCacheSize = 1000;
    }
    
    set(key, value) {
        // Implement LRU cache
        if (this.cache.size >= this.maxCacheSize) {
            const firstKey = this.cache.keys().next().value;
            this.cache.delete(firstKey);
        }
        
        this.cache.set(key, {
            value,
            timestamp: Date.now()
        });
    }
    
    get(key) {
        const item = this.cache.get(key);
        if (!item) return null;
        
        // Check if expired (1 hour)
        if (Date.now() - item.timestamp > 3600000) {
            this.cache.delete(key);
            return null;
        }
        
        return item.value;
    }
}

// Compression middleware
const compression = require('compression');

app.use(compression({
    level: 6,
    threshold: 1024,
    filter: (req, res) => {
        if (req.headers['x-no-compression']) {
            return false;
        }
        return compression.filter(req, res);
    }
}));

// Response optimization
app.get('/api/optimized-data', async (req, res) => {
    // Set appropriate headers
    res.set({
        'Content-Type': 'application/json',
        'Cache-Control': 'public, max-age=300', // 5 minutes
        'ETag': `"${Date.now()}"`
    });
    
    // Check if client has cached version
    if (req.headers['if-none-match'] === res.get('ETag')) {
        return res.status(304).end();
    }
    
    const data = await fetchOptimizedData();
    res.json(data);
});
```

### Q96. Clustering and Load Balancing
**Answer:** Clustering allows Node.js to utilize multiple CPU cores for better performance.

**Real-time Example:**
```javascript
const cluster = require('cluster');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
    console.log(`Master ${process.pid} is running`);
    
    // Fork workers
    for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
    }
    
    cluster.on('exit', (worker, code, signal) => {
        console.log(`Worker ${worker.process.pid} died`);
        console.log('Starting a new worker');
        cluster.fork();
    });
    
    // Graceful shutdown
    process.on('SIGTERM', () => {
        console.log('Master received SIGTERM');
        for (const id in cluster.workers) {
            cluster.workers[id].kill();
        }
    });
} else {
    // Worker process
    const express = require('express');
    const app = express();
    
    app.get('/', (req, res) => {
        res.json({
            message: 'Hello from worker',
            pid: process.pid,
            timestamp: new Date().toISOString()
        });
    });
    
    const server = app.listen(3000, () => {
        console.log(`Worker ${process.pid} started`);
    });
    
    // Graceful shutdown for workers
    process.on('SIGTERM', () => {
        console.log(`Worker ${process.pid} received SIGTERM`);
        server.close(() => {
            process.exit(0);
        });
    });
}

// Load balancing with Nginx
const nginxConfig = `
upstream backend {
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
    server 127.0.0.1:3003;
}

server {
    listen 80;
    server_name example.com;
    
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
`;

// PM2 configuration for production
const pm2Config = {
    apps: [{
        name: 'api-server',
        script: './app.js',
        instances: 'max',
        exec_mode: 'cluster',
        env: {
            NODE_ENV: 'production',
            PORT: 3000
        },
        env_production: {
            NODE_ENV: 'production',
            PORT: 3000
        }
    }]
};
```

---

## Microservices & Advanced Patterns

### Q97. Microservices Architecture
**Answer:** Microservices break down applications into small, independent services.

**Real-time Example:**
```javascript
// User Service
const express = require('express');
const app = express();

class UserService {
    constructor() {
        this.users = new Map();
        this.orderServiceUrl = process.env.ORDER_SERVICE_URL;
    }
    
    async createUser(userData) {
        const user = {
            id: Date.now(),
            ...userData,
            createdAt: new Date()
        };
        
        this.users.set(user.id, user);
        return user;
    }
    
    async getUserById(id) {
        return this.users.get(parseInt(id));
    }
    
    async getUserOrders(userId) {
        // Call Order Service
        const response = await fetch(`${this.orderServiceUrl}/orders?userId=${userId}`);
        return await response.json();
    }
}

const userService = new UserService();

// Routes
app.get('/users/:id', async (req, res) => {
    const user = await userService.getUserById(req.params.id);
    if (!user) {
        return res.status(404).json({ error: 'User not found' });
    }
    res.json(user);
});

app.get('/users/:id/orders', async (req, res) => {
    try {
        const orders = await userService.getUserOrders(req.params.id);
        res.json(orders);
    } catch (error) {
        res.status(500).json({ error: 'Failed to fetch orders' });
    }
});

// Service Discovery
class ServiceRegistry {
    constructor() {
        this.services = new Map();
    }
    
    register(serviceName, serviceUrl) {
        this.services.set(serviceName, serviceUrl);
        console.log(`Registered service: ${serviceName} at ${serviceUrl}`);
    }
    
    getService(serviceName) {
        return this.services.get(serviceName);
    }
    
    getAllServices() {
        return Array.from(this.services.entries());
    }
}

// API Gateway
class APIGateway {
    constructor() {
        this.serviceRegistry = new ServiceRegistry();
        this.routes = new Map();
    }
    
    addRoute(path, serviceName) {
        this.routes.set(path, serviceName);
    }
    
    async routeRequest(req, res) {
        const serviceName = this.routes.get(req.path);
        
        if (!serviceName) {
            return res.status(404).json({ error: 'Service not found' });
        }
        
        const serviceUrl = this.serviceRegistry.getService(serviceName);
        
        if (!serviceUrl) {
            return res.status(503).json({ error: 'Service unavailable' });
        }
        
        try {
            const response = await fetch(`${serviceUrl}${req.path}`, {
                method: req.method,
                headers: req.headers,
                body: req.method !== 'GET' ? JSON.stringify(req.body) : undefined
            });
            
            const data = await response.json();
            res.status(response.status).json(data);
        } catch (error) {
            res.status(500).json({ error: 'Internal server error' });
        }
    }
}

// Circuit Breaker Pattern
class CircuitBreaker {
    constructor(threshold = 5, timeout = 60000) {
        this.threshold = threshold;
        this.timeout = timeout;
        this.failureCount = 0;
        this.lastFailureTime = null;
        this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
    }
    
    async execute(operation) {
        if (this.state === 'OPEN') {
            if (Date.now() - this.lastFailureTime > this.timeout) {
                this.state = 'HALF_OPEN';
            } else {
                throw new Error('Circuit breaker is OPEN');
            }
        }
        
        try {
            const result = await operation();
            this.onSuccess();
            return result;
        } catch (error) {
            this.onFailure();
            throw error;
        }
    }
    
    onSuccess() {
        this.failureCount = 0;
        this.state = 'CLOSED';
    }
    
    onFailure() {
        this.failureCount++;
        this.lastFailureTime = Date.now();
        
        if (this.failureCount >= this.threshold) {
            this.state = 'OPEN';
        }
    }
}

// Event Sourcing
class EventStore {
    constructor() {
        this.events = [];
    }
    
    appendEvent(aggregateId, event) {
        const eventRecord = {
            id: Date.now(),
            aggregateId,
            event,
            timestamp: new Date(),
            version: this.getNextVersion(aggregateId)
        };
        
        this.events.push(eventRecord);
        return eventRecord;
    }
    
    getEvents(aggregateId) {
        return this.events.filter(e => e.aggregateId === aggregateId);
    }
    
    getNextVersion(aggregateId) {
        const events = this.getEvents(aggregateId);
        return events.length + 1;
    }
}

// CQRS (Command Query Responsibility Segregation)
class CommandHandler {
    constructor(eventStore) {
        this.eventStore = eventStore;
    }
    
    async handle(command) {
        switch (command.type) {
            case 'CREATE_USER':
                return await this.createUser(command);
            case 'UPDATE_USER':
                return await this.updateUser(command);
            default:
                throw new Error('Unknown command type');
        }
    }
    
    async createUser(command) {
        const event = {
            type: 'USER_CREATED',
            data: command.data
        };
        
        return this.eventStore.appendEvent(command.aggregateId, event);
    }
}

class QueryHandler {
    constructor(readModel) {
        this.readModel = readModel;
    }
    
    async handle(query) {
        switch (query.type) {
            case 'GET_USER':
                return await this.getUser(query);
            case 'LIST_USERS':
                return await this.listUsers(query);
            default:
                throw new Error('Unknown query type');
        }
    }
    
    async getUser(query) {
        return this.readModel.getUser(query.userId);
    }
}
```

### Q98. Most Commonly Asked Interview Questions

**Q: How does Node.js handle concurrent requests?**
**A:** Node.js uses a single-threaded event loop with non-blocking I/O operations. When a request comes in, it's added to the event queue. The event loop processes these requests one by one, but when an I/O operation is encountered, it's delegated to the system, and the event loop continues processing other requests.

**Q: What is the difference between process.nextTick() and setImmediate()?**
**A:** 
- `process.nextTick()` has the highest priority and executes before any I/O operations
- `setImmediate()` executes in the Check phase, after I/O operations
- `process.nextTick()` can starve the event loop if used recursively

**Q: How do you handle memory leaks in Node.js?**
**A:** 
- Use `process.memoryUsage()` to monitor memory
- Avoid global variables
- Clear timers and intervals
- Remove event listeners
- Use streams for large data processing
- Implement proper garbage collection

**Q: What is the difference between cluster and worker_threads?**
**A:**
- **Cluster**: Multiple processes, each with its own memory space, good for CPU-intensive tasks
- **Worker Threads**: Multiple threads within the same process, share memory, good for I/O-intensive tasks

**Q: How do you implement authentication in Node.js?**
**A:** Use JWT tokens, bcrypt for password hashing, middleware for route protection, and session management for stateful authentication.

### Q99. Practical Coding Questions

**Question 1: Implement a rate limiter**
```javascript
class RateLimiter {
    constructor(maxRequests, windowMs) {
        this.maxRequests = maxRequests;
        this.windowMs = windowMs;
        this.requests = new Map();
    }
    
    isAllowed(identifier) {
        const now = Date.now();
        const windowStart = now - this.windowMs;
        
        if (!this.requests.has(identifier)) {
            this.requests.set(identifier, []);
        }
        
        const userRequests = this.requests.get(identifier);
        const validRequests = userRequests.filter(time => time > windowStart);
        
        if (validRequests.length >= this.maxRequests) {
            return false;
        }
        
        validRequests.push(now);
        this.requests.set(identifier, validRequests);
        return true;
    }
}
```

**Question 2: Implement a caching middleware**
```javascript
const cache = new Map();

function cacheMiddleware(ttl = 300000) { // 5 minutes default
    return (req, res, next) => {
        const key = req.originalUrl;
        const cached = cache.get(key);
        
        if (cached && (Date.now() - cached.timestamp) < ttl) {
            return res.json(cached.data);
        }
        
        const originalSend = res.json;
        res.json = function(data) {
            cache.set(key, {
                data,
                timestamp: Date.now()
            });
            originalSend.call(this, data);
        };
        
        next();
    };
}
```

**Question 3: Implement a file upload with validation**
```javascript
const multer = require('multer');
const path = require('path');

const storage = multer.diskStorage({
    destination: (req, file, cb) => {
        cb(null, 'uploads/');
    },
    filename: (req, file, cb) => {
        const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
        cb(null, file.fieldname + '-' + uniqueSuffix + path.extname(file.originalname));
    }
});

const fileFilter = (req, file, cb) => {
    const allowedTypes = /jpeg|jpg|png|gif/;
    const extname = allowedTypes.test(path.extname(file.originalname).toLowerCase());
    const mimetype = allowedTypes.test(file.mimetype);
    
    if (mimetype && extname) {
        return cb(null, true);
    } else {
        cb(new Error('Only image files are allowed'));
    }
};

const upload = multer({
    storage,
    limits: {
        fileSize: 5 * 1024 * 1024 // 5MB
    },
    fileFilter
});

app.post('/upload', upload.single('image'), (req, res) => {
    if (!req.file) {
        return res.status(400).json({ error: 'No file uploaded' });
    }
    
    res.json({
        message: 'File uploaded successfully',
        file: req.file
    });
});
```

**Total Node.js Questions: 99+ covering all major topics with practical examples!**
