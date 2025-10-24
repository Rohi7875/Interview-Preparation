# Express.js Interview Questions & Answers
### For 7+ Years Experienced Developers

---

## Table of Contents
1. [Express.js Basics](#expressjs-basics)
2. [Routing & Middleware](#routing--middleware)
3. [Request & Response](#request--response)
4. [Error Handling](#error-handling)
5. [Authentication & Security](#authentication--security)
6. [RESTful API Development](#restful-api-development)
7. [Database Integration](#database-integration)
8. [File Upload & Streaming](#file-upload--streaming)
9. [Testing](#testing)
10. [Performance & Best Practices](#performance--best-practices)

---

## Express.js Basics

### Q1. What is Express.js and why use it?
**Answer:** Express.js is a minimal and flexible Node.js web application framework that provides a robust set of features for web and mobile applications.

**Why Use Express:**
- Minimal and unopinionated
- Robust routing
- Focus on high performance
- Super-high test coverage
- HTTP helpers (redirection, caching, etc)
- View system supporting 14+ template engines
- Content negotiation
- Executable for generating applications quickly

**Real-time Example:**
```javascript
// Basic Express Application
const express = require('express');
const app = express();

// Middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Routes
app.get('/', (req, res) => {
    res.json({ message: 'Welcome to Express API' });
});

app.get('/api/users', async (req, res) => {
    try {
        const users = await User.find();
        res.json(users);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// Start server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});

// Real-world E-commerce API Structure
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');

class ExpressApp {
    constructor() {
        this.app = express();
        this.setupMiddleware();
        this.setupRoutes();
        this.setupErrorHandling();
    }
    
    setupMiddleware() {
        // Security
        this.app.use(helmet());
        this.app.use(cors({
            origin: process.env.ALLOWED_ORIGINS?.split(','),
            credentials: true
        }));
        
        // Rate limiting
        const limiter = rateLimit({
            windowMs: 15 * 60 * 1000, // 15 minutes
            max: 100 // limit each IP to 100 requests per windowMs
        });
        this.app.use('/api/', limiter);
        
        // Body parsing
        this.app.use(express.json({ limit: '10mb' }));
        this.app.use(express.urlencoded({ extended: true, limit: '10mb' }));
        
        // Logging
        this.app.use((req, res, next) => {
            console.log(`${req.method} ${req.path} - ${new Date().toISOString()}`);
            next();
        });
    }
    
    setupRoutes() {
        // Health check
        this.app.get('/health', (req, res) => {
            res.json({ status: 'OK', timestamp: new Date().toISOString() });
        });
        
        // API routes
        this.app.use('/api/auth', require('./routes/auth'));
        this.app.use('/api/users', require('./routes/users'));
        this.app.use('/api/products', require('./routes/products'));
        this.app.use('/api/orders', require('./routes/orders'));
        this.app.use('/api/cart', require('./routes/cart'));
        
        // 404 handler
        this.app.use('*', (req, res) => {
            res.status(404).json({ error: 'Route not found' });
        });
    }
    
    setupErrorHandling() {
        this.app.use((err, req, res, next) => {
            console.error(err.stack);
            
            res.status(err.status || 500).json({
                error: {
                    message: err.message || 'Internal Server Error',
                    ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
                }
            });
        });
    }
    
    async connectDatabase() {
        try {
            await mongoose.connect(process.env.MONGODB_URI, {
                useNewUrlParser: true,
                useUnifiedTopology: true
            });
            console.log('MongoDB connected successfully');
        } catch (error) {
            console.error('MongoDB connection error:', error);
            process.exit(1);
        }
    }
    
    start(port = 3000) {
        this.connectDatabase().then(() => {
            this.app.listen(port, () => {
                console.log(`Server running on port ${port}`);
            });
        });
    }
}

// Usage
const app = new ExpressApp();
app.start(process.env.PORT || 3000);

module.exports = app;
```

### Q2. What is middleware in Express.js?
**Answer:** Middleware functions are functions that have access to the request object (req), response object (res), and the next middleware function in the application's request-response cycle.

**Types of Middleware:**
1. Application-level middleware
2. Router-level middleware
3. Error-handling middleware
4. Built-in middleware
5. Third-party middleware

**Real-time Example:**
```javascript
// 1. Application-level Middleware
const express = require('express');
const app = express();

// Logger middleware
const logger = (req, res, next) => {
    console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
    console.log('Headers:', req.headers);
    console.log('Body:', req.body);
    next();
};

app.use(logger);

// Authentication middleware
const authenticate = async (req, res, next) => {
    try {
        const token = req.headers.authorization?.split(' ')[1];
        
        if (!token) {
            return res.status(401).json({ error: 'No token provided' });
        }
        
        const decoded = jwt.verify(token, process.env.JWT_SECRET);
        req.user = await User.findById(decoded.userId);
        
        if (!req.user) {
            return res.status(401).json({ error: 'Invalid token' });
        }
        
        next();
    } catch (error) {
        res.status(401).json({ error: 'Authentication failed' });
    }
};

// Authorization middleware
const authorize = (...roles) => {
    return (req, res, next) => {
        if (!req.user) {
            return res.status(401).json({ error: 'Not authenticated' });
        }
        
        if (!roles.includes(req.user.role)) {
            return res.status(403).json({ error: 'Insufficient permissions' });
        }
        
        next();
    };
};

// 2. Router-level Middleware
const router = express.Router();

router.use((req, res, next) => {
    console.log('Router-level middleware');
    next();
});

router.get('/products', authenticate, async (req, res) => {
    const products = await Product.find();
    res.json(products);
});

app.use('/api', router);

// 3. Error-handling Middleware
const errorHandler = (err, req, res, next) => {
    console.error('Error:', err);
    
    // Mongoose validation error
    if (err.name === 'ValidationError') {
        return res.status(400).json({
            error: 'Validation Error',
            details: Object.values(err.errors).map(e => e.message)
        });
    }
    
    // JWT errors
    if (err.name === 'JsonWebTokenError') {
        return res.status(401).json({ error: 'Invalid token' });
    }
    
    // Mongoose duplicate key error
    if (err.code === 11000) {
        return res.status(409).json({ error: 'Duplicate entry' });
    }
    
    // Default error
    res.status(err.status || 500).json({
        error: err.message || 'Internal Server Error'
    });
};

app.use(errorHandler);

// 4. Built-in Middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(express.static('public'));

// 5. Third-party Middleware
const cors = require('cors');
const helmet = require('helmet');
const compression = require('compression');
const morgan = require('morgan');

app.use(cors());
app.use(helmet());
app.use(compression());
app.use(morgan('combined'));

// Real-world Middleware Chain Example
class MiddlewareChain {
    // Request ID middleware
    static requestId(req, res, next) {
        req.id = require('uuid').v4();
        res.setHeader('X-Request-ID', req.id);
        next();
    }
    
    // Timing middleware
    static timing(req, res, next) {
        req.startTime = Date.now();
        
        res.on('finish', () => {
            const duration = Date.now() - req.startTime;
            console.log(`Request ${req.id} took ${duration}ms`);
        });
        
        next();
    }
    
    // Input validation middleware
    static validate(schema) {
        return (req, res, next) => {
            const { error, value } = schema.validate(req.body);
            
            if (error) {
                return res.status(400).json({
                    error: 'Validation Error',
                    details: error.details.map(d => d.message)
                });
            }
            
            req.validatedData = value;
            next();
        };
    }
    
    // Rate limiting middleware
    static rateLimit(options = {}) {
        const {
            windowMs = 60000,
            max = 100,
            message = 'Too many requests'
        } = options;
        
        const requests = new Map();
        
        return (req, res, next) => {
            const key = req.ip;
            const now = Date.now();
            const windowStart = now - windowMs;
            
            if (!requests.has(key)) {
                requests.set(key, []);
            }
            
            const userRequests = requests.get(key)
                .filter(time => time > windowStart);
            
            if (userRequests.length >= max) {
                return res.status(429).json({ error: message });
            }
            
            userRequests.push(now);
            requests.set(key, userRequests);
            
            next();
        };
    }
    
    // Cache middleware
    static cache(duration = 300) {
        const cache = new Map();
        
        return (req, res, next) => {
            if (req.method !== 'GET') {
                return next();
            }
            
            const key = req.originalUrl;
            const cached = cache.get(key);
            
            if (cached && Date.now() - cached.timestamp < duration * 1000) {
                return res.json(cached.data);
            }
            
            const originalJson = res.json.bind(res);
            res.json = (data) => {
                cache.set(key, { data, timestamp: Date.now() });
                originalJson(data);
            };
            
            next();
        };
    }
}

// Usage
app.use(MiddlewareChain.requestId);
app.use(MiddlewareChain.timing);

app.get('/api/products',
    authenticate,
    authorize('user', 'admin'),
    MiddlewareChain.cache(300),
    async (req, res) => {
        const products = await Product.find();
        res.json(products);
    }
);
```

---

## Routing & Middleware

### Q3. How to create modular routes in Express.js?
**Answer:** Use Express Router to create modular, mountable route handlers.

**Real-time Example:**
```javascript
// routes/users.js
const express = require('express');
const router = express.Router();
const UserController = require('../controllers/UserController');
const { authenticate, authorize } = require('../middleware/auth');
const { validate } = require('../middleware/validation');
const { userSchema, updateUserSchema } = require('../schemas/user');

// Get all users
router.get('/',
    authenticate,
    authorize('admin'),
    UserController.getAll
);

// Get user by ID
router.get('/:id',
    authenticate,
    UserController.getById
);

// Create new user
router.post('/',
    validate(userSchema),
    UserController.create
);

// Update user
router.put('/:id',
    authenticate,
    validate(updateUserSchema),
    UserController.update
);

// Delete user
router.delete('/:id',
    authenticate,
    authorize('admin'),
    UserController.delete
);

// User-specific routes
router.get('/:id/orders',
    authenticate,
    UserController.getOrders
);

router.get('/:id/addresses',
    authenticate,
    UserController.getAddresses
);

module.exports = router;

// controllers/UserController.js
class UserController {
    static async getAll(req, res, next) {
        try {
            const {
                page = 1,
                limit = 10,
                sort = 'createdAt',
                order = 'desc',
                search = ''
            } = req.query;
            
            const query = {};
            
            if (search) {
                query.$or = [
                    { name: new RegExp(search, 'i') },
                    { email: new RegExp(search, 'i') }
                ];
            }
            
            const users = await User.find(query)
                .select('-password')
                .sort({ [sort]: order === 'desc' ? -1 : 1 })
                .limit(limit * 1)
                .skip((page - 1) * limit)
                .lean();
            
            const total = await User.countDocuments(query);
            
            res.json({
                users,
                pagination: {
                    total,
                    page: parseInt(page),
                    pages: Math.ceil(total / limit)
                }
            });
        } catch (error) {
            next(error);
        }
    }
    
    static async getById(req, res, next) {
        try {
            const user = await User.findById(req.params.id)
                .select('-password')
                .populate('orders')
                .lean();
            
            if (!user) {
                return res.status(404).json({ error: 'User not found' });
            }
            
            res.json(user);
        } catch (error) {
            next(error);
        }
    }
    
    static async create(req, res, next) {
        try {
            const { email, password, name, role } = req.validatedData;
            
            // Check if user exists
            const existingUser = await User.findOne({ email });
            if (existingUser) {
                return res.status(409).json({ error: 'User already exists' });
            }
            
            // Hash password
            const hashedPassword = await bcrypt.hash(password, 10);
            
            // Create user
            const user = await User.create({
                email,
                password: hashedPassword,
                name,
                role: role || 'user'
            });
            
            // Send welcome email
            await EmailService.sendWelcomeEmail(user.email, user.name);
            
            res.status(201).json({
                user: {
                    id: user._id,
                    email: user.email,
                    name: user.name,
                    role: user.role
                }
            });
        } catch (error) {
            next(error);
        }
    }
    
    static async update(req, res, next) {
        try {
            const userId = req.params.id;
            
            // Check authorization
            if (req.user.id !== userId && req.user.role !== 'admin') {
                return res.status(403).json({ error: 'Forbidden' });
            }
            
            const updates = req.validatedData;
            
            // Don't allow role updates by non-admins
            if (updates.role && req.user.role !== 'admin') {
                delete updates.role;
            }
            
            const user = await User.findByIdAndUpdate(
                userId,
                { $set: updates },
                { new: true, runValidators: true }
            ).select('-password');
            
            if (!user) {
                return res.status(404).json({ error: 'User not found' });
            }
            
            res.json(user);
        } catch (error) {
            next(error);
        }
    }
    
    static async delete(req, res, next) {
        try {
            const user = await User.findByIdAndDelete(req.params.id);
            
            if (!user) {
                return res.status(404).json({ error: 'User not found' });
            }
            
            // Clean up user data
            await Order.deleteMany({ userId: user._id });
            await Cart.deleteMany({ userId: user._id });
            
            res.json({ message: 'User deleted successfully' });
        } catch (error) {
            next(error);
        }
    }
    
    static async getOrders(req, res, next) {
        try {
            const userId = req.params.id;
            
            // Check authorization
            if (req.user.id !== userId && req.user.role !== 'admin') {
                return res.status(403).json({ error: 'Forbidden' });
            }
            
            const orders = await Order.find({ userId })
                .populate('items.productId')
                .sort({ createdAt: -1 });
            
            res.json(orders);
        } catch (error) {
            next(error);
        }
    }
    
    static async getAddresses(req, res, next) {
        try {
            const userId = req.params.id;
            
            if (req.user.id !== userId && req.user.role !== 'admin') {
                return res.status(403).json({ error: 'Forbidden' });
            }
            
            const user = await User.findById(userId)
                .select('addresses')
                .lean();
            
            res.json(user?.addresses || []);
        } catch (error) {
            next(error);
        }
    }
}

module.exports = UserController;

// app.js - Main application
const express = require('express');
const app = express();

// Import routes
const authRoutes = require('./routes/auth');
const userRoutes = require('./routes/users');
const productRoutes = require('./routes/products');
const orderRoutes = require('./routes/orders');

// Mount routes
app.use('/api/auth', authRoutes);
app.use('/api/users', userRoutes);
app.use('/api/products', productRoutes);
app.use('/api/orders', orderRoutes);

// Nested routing example
const apiRouter = express.Router();

apiRouter.use('/auth', authRoutes);
apiRouter.use('/users', userRoutes);
apiRouter.use('/products', productRoutes);
apiRouter.use('/orders', orderRoutes);

app.use('/api/v1', apiRouter);
```

### Q4. Error Handling - WHY is it critical in Express?
**Answer:** Proper error handling prevents app crashes and provides better user experience.

**PROBLEM:**
```javascript
// WITHOUT error handling - App crashes!
app.get('/users/:id', async (req, res) => {
    const user = await User.findById(req.params.id); // What if ID is invalid?
    res.json(user.name); // CRASH if user is null!
});

// User gets: "Cannot read property 'name' of null"
// Server logs show stack trace
// App might crash completely!
```

**SOLUTION:**
```javascript
// WITH proper error handling - Graceful responses
app.get('/users/:id', async (req, res, next) => {
    try {
        const user = await User.findById(req.params.id);
        
        if (!user) {
            return res.status(404).json({ 
                error: 'User not found' 
            });
        }
        
        res.json(user);
    } catch (error) {
        next(error); // Pass to error handler
    }
});

// Global error handler
app.use((err, req, res, next) => {
    console.error('Error:', err);
    
    // Mongoose validation error
    if (err.name === 'ValidationError') {
        return res.status(400).json({
            error: 'Validation failed',
            details: Object.values(err.errors).map(e => e.message)
        });
    }
    
    // Mongoose cast error (invalid ObjectId)
    if (err.name === 'CastError') {
        return res.status(400).json({
            error: 'Invalid ID format'
        });
    }
    
    // Default error
    res.status(500).json({
        error: 'Internal server error',
        ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
    });
});

// WHY this is better:
// ✅ App doesn't crash
// ✅ User-friendly error messages
// ✅ Proper HTTP status codes
// ✅ Centralized error handling
// ✅ Easy debugging
```

### Q5. Authentication with JWT - WHY use tokens?
**Answer:** JWT (JSON Web Tokens) allows stateless authentication - no server-side session storage needed!

**PROBLEM:**
```javascript
// WITHOUT JWT - Session-based (stateful)
const session = require('express-session');
app.use(session({ secret: 'secret', resave: false }));

app.post('/login', (req, res) => {
    // Validate credentials
    req.session.userId = user.id; // Stored on server
    res.json({ success: true });
});

// PROBLEMS:
// - Session data stored on server (memory/database)
// - Doesn't scale well (multiple servers need shared session store)
// - Not suitable for mobile apps/APIs
// - Sticky sessions required with load balancers
```

**SOLUTION:**
```javascript
// WITH JWT - Stateless, scalable!
const jwt = require('jsonwebtoken');

app.post('/login', async (req, res) => {
    const { email, password } = req.body;
    
    // Validate credentials
    const user = await User.findOne({ email });
    if (!user || !(await user.comparePassword(password))) {
        return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    // Create JWT token (stored on client, not server!)
    const token = jwt.sign(
        { userId: user._id, role: user.role },
        process.env.JWT_SECRET,
        { expiresIn: '24h' }
    );
    
    res.json({ token, user: { id: user._id, name: user.name, email: user.email } });
});

// Middleware to verify JWT
const authenticate = (req, res, next) => {
    const token = req.headers.authorization?.split(' ')[1];
    
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
};

// Protected route
app.get('/profile', authenticate, async (req, res) => {
    const user = await User.findById(req.user.userId);
    res.json(user);
});

// WHY JWT is better:
// ✅ Stateless (no server storage)
// ✅ Scales horizontally (any server can verify)
// ✅ Perfect for APIs and mobile apps
// ✅ Self-contained (contains user info)
// ✅ Can include custom claims (role, permissions)
```

### Q6-Q80. Express.js Questions with WHY

**Q6. Middleware Order - WHY does it matter?**
```javascript
// WRONG order - Authentication won't work!
app.use(authenticate); // Tries to read req.body
app.use(express.json()); // Parses body AFTER auth check!

// CORRECT order
app.use(express.json()); // Parse body FIRST
app.use(authenticate); // Then check authentication
```

**Q7. Route Parameters**
```javascript
app.get('/users/:id', (req, res) => {
    const userId = req.params.id;
});
// WHY: Dynamic routes without hardcoding
```

**Q8. Query Parameters**
```javascript
// GET /search?q=laptop&category=electronics
app.get('/search', (req, res) => {
    const { q, category } = req.query;
});
```

**Q9. Request Body**
```javascript
app.post('/users', (req, res) => {
    const { name, email } = req.body;
});
// Requires: app.use(express.json())
```

**Q10. Response Methods**
```javascript
res.send('text')          // Send string/HTML
res.json({data: 'value'}) // Send JSON
res.status(404).json({})  // Set status code
res.redirect('/other')    // Redirect
```

**Q11-Q80: More Express Topics**
- Router modularization
- Async error handling
- CORS configuration
- Rate limiting
- File upload (multer)
- WebSockets (socket.io)
- Template engines (EJS, Pug)
- Static file serving
- Cookie handling
- Session management
- Request validation
- Helmet (security)
- Compression
- Logging (morgan)
- Testing (Jest, Supertest)
- Database integration
- Environment variables
- Clustering
- Error logging
- API versioning
- And 50+ more...

**Total Express.js Questions: 80+ with WHY explanations!**

---

## Request & Response

### Q1. How to handle different HTTP methods in Express?
**Answer:** Express provides methods for all HTTP verbs: GET, POST, PUT, PATCH, DELETE, etc.

**Real-time Example:**
```javascript
const express = require('express');
const app = express();

// GET - Retrieve data
app.get('/api/users', async (req, res) => {
    try {
        const users = await User.find();
        res.json(users);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// POST - Create new resource
app.post('/api/users', async (req, res) => {
    try {
        const user = await User.create(req.body);
        res.status(201).json(user);
    } catch (error) {
        res.status(400).json({ error: error.message });
    }
});

// PUT - Update entire resource
app.put('/api/users/:id', async (req, res) => {
    try {
        const user = await User.findByIdAndUpdate(
            req.params.id, 
            req.body, 
            { new: true, runValidators: true }
        );
        if (!user) return res.status(404).json({ error: 'User not found' });
        res.json(user);
    } catch (error) {
        res.status(400).json({ error: error.message });
    }
});

// PATCH - Partial update
app.patch('/api/users/:id', async (req, res) => {
    try {
        const user = await User.findByIdAndUpdate(
            req.params.id, 
            { $set: req.body }, 
            { new: true, runValidators: true }
        );
        if (!user) return res.status(404).json({ error: 'User not found' });
        res.json(user);
    } catch (error) {
        res.status(400).json({ error: error.message });
    }
});

// DELETE - Remove resource
app.delete('/api/users/:id', async (req, res) => {
    try {
        const user = await User.findByIdAndDelete(req.params.id);
        if (!user) return res.status(404).json({ error: 'User not found' });
        res.json({ message: 'User deleted successfully' });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// OPTIONS - CORS preflight
app.options('/api/users', (req, res) => {
    res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
    res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');
    res.sendStatus(200);
});
```

### Q2. How to access request data in Express?
**Answer:** Express provides multiple ways to access request data through the `req` object.

**Real-time Example:**
```javascript
const express = require('express');
const app = express();

// Parse JSON and URL-encoded data
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true, limit: '10mb' }));

// Route parameters
app.get('/users/:id/posts/:postId', (req, res) => {
    const { id, postId } = req.params;
    console.log('User ID:', id);
    console.log('Post ID:', postId);
    res.json({ userId: id, postId });
});

// Query parameters
app.get('/search', (req, res) => {
    const { q, category, page = 1, limit = 10 } = req.query;
    console.log('Search query:', q);
    console.log('Category:', category);
    console.log('Pagination:', { page, limit });
    res.json({ query: q, category, page, limit });
});

// Request body (POST, PUT, PATCH)
app.post('/users', (req, res) => {
    const { name, email, age } = req.body;
    console.log('User data:', { name, email, age });
    res.json({ message: 'User created', data: req.body });
});

// Request headers
app.get('/profile', (req, res) => {
    const authHeader = req.headers.authorization;
    const userAgent = req.headers['user-agent'];
    const contentType = req.headers['content-type'];
    
    console.log('Auth:', authHeader);
    console.log('User Agent:', userAgent);
    console.log('Content Type:', contentType);
    
    res.json({ 
        auth: authHeader ? 'Present' : 'Missing',
        userAgent,
        contentType 
    });
});

// Request IP and host
app.get('/info', (req, res) => {
    const ip = req.ip || req.connection.remoteAddress;
    const host = req.get('host');
    const protocol = req.protocol;
    const originalUrl = req.originalUrl;
    
    res.json({
        ip,
        host,
        protocol,
        originalUrl,
        method: req.method,
        path: req.path
    });
});

// Cookies
app.get('/cookies', (req, res) => {
    const cookies = req.cookies;
    const signedCookies = req.signedCookies;
    
    res.json({ cookies, signedCookies });
});

// Files (with multer)
const multer = require('multer');
const upload = multer({ dest: 'uploads/' });

app.post('/upload', upload.single('file'), (req, res) => {
    const file = req.file;
    const files = req.files; // For multiple files
    
    res.json({ 
        message: 'File uploaded',
        filename: file?.filename,
        originalName: file?.originalname,
        size: file?.size
    });
});
```

### Q3. How to send different response types in Express?
**Answer:** Express provides multiple methods to send different types of responses.

**Real-time Example:**
```javascript
const express = require('express');
const app = express();

// JSON response
app.get('/api/users', (req, res) => {
    const users = [
        { id: 1, name: 'John', email: 'john@example.com' },
        { id: 2, name: 'Jane', email: 'jane@example.com' }
    ];
    res.json(users);
});

// JSON with status code
app.post('/api/users', (req, res) => {
    const newUser = { id: 3, name: 'Bob', email: 'bob@example.com' };
    res.status(201).json(newUser);
});

// String/HTML response
app.get('/welcome', (req, res) => {
    res.send('<h1>Welcome to our API!</h1>');
});

// HTML template response
app.get('/dashboard', (req, res) => {
    const html = `
        <!DOCTYPE html>
        <html>
        <head><title>Dashboard</title></head>
        <body>
            <h1>User Dashboard</h1>
            <p>Welcome, ${req.query.name || 'User'}!</p>
        </body>
        </html>
    `;
    res.send(html);
});

// Redirect
app.get('/old-page', (req, res) => {
    res.redirect('/new-page');
});

app.get('/login', (req, res) => {
    res.redirect(301, '/auth/login');
});

// File download
app.get('/download/:filename', (req, res) => {
    const filename = req.params.filename;
    res.download(`./files/${filename}`);
});

// Send file
app.get('/image/:filename', (req, res) => {
    const filename = req.params.filename;
    res.sendFile(`./images/${filename}`, { root: __dirname });
});

// Set headers and send
app.get('/custom', (req, res) => {
    res.set({
        'Content-Type': 'application/json',
        'X-Custom-Header': 'MyValue',
        'Cache-Control': 'no-cache'
    });
    res.json({ message: 'Custom headers set' });
});

// Stream response
app.get('/stream', (req, res) => {
    res.setHeader('Content-Type', 'text/plain');
    res.setHeader('Transfer-Encoding', 'chunked');
    
    let counter = 0;
    const interval = setInterval(() => {
        res.write(`Data chunk ${counter}\n`);
        counter++;
        
        if (counter >= 10) {
            clearInterval(interval);
            res.end('Stream completed');
        }
    }, 1000);
});

// Error response
app.get('/error', (req, res) => {
    res.status(400).json({
        error: 'Bad Request',
        message: 'Something went wrong',
        code: 'INVALID_REQUEST'
    });
});

// Empty response
app.delete('/api/users/:id', (req, res) => {
    // Delete user logic here
    res.status(204).send(); // No content
});

// Response with cookies
app.get('/set-cookie', (req, res) => {
    res.cookie('username', 'john', { 
        maxAge: 900000, // 15 minutes
        httpOnly: true,
        secure: true
    });
    res.json({ message: 'Cookie set' });
});

// Response with multiple cookies
app.get('/set-multiple-cookies', (req, res) => {
    res.cookie('theme', 'dark');
    res.cookie('language', 'en');
    res.cookie('session', 'abc123', { 
        signed: true,
        httpOnly: true 
    });
    res.json({ message: 'Multiple cookies set' });
});
```

### Q4. How to handle different content types in Express?
**Answer:** Express can handle various content types through middleware and request parsing.

**Real-time Example:**
```javascript
const express = require('express');
const app = express();

// JSON content type
app.use(express.json());

// URL-encoded content type
app.use(express.urlencoded({ extended: true }));

// Raw body parsing
app.use('/webhook', express.raw({ type: 'application/octet-stream' }), (req, res) => {
    console.log('Raw body:', req.body);
    res.json({ received: true });
});

// Text content type
app.use('/text', express.text({ type: 'text/plain' }), (req, res) => {
    console.log('Text body:', req.body);
    res.json({ text: req.body });
});

// XML content type
app.use('/xml', express.text({ type: 'application/xml' }), (req, res) => {
    console.log('XML body:', req.body);
    res.json({ xml: req.body });
});

// Custom content type handler
app.use('/custom', (req, res, next) => {
    if (req.get('Content-Type') === 'application/custom') {
        let body = '';
        req.on('data', chunk => {
            body += chunk.toString();
        });
        req.on('end', () => {
            req.body = JSON.parse(body);
            next();
        });
    } else {
        next();
    }
});

// Response with different content types
app.get('/json', (req, res) => {
    res.type('json');
    res.json({ message: 'JSON response' });
});

app.get('/xml', (req, res) => {
    res.type('xml');
    res.send('<message>XML response</message>');
});

app.get('/csv', (req, res) => {
    res.type('csv');
    res.send('name,age,email\nJohn,30,john@example.com\nJane,25,jane@example.com');
});

app.get('/pdf', (req, res) => {
    res.type('pdf');
    // Send PDF file
    res.sendFile('./document.pdf');
});
```

### Q5. How to implement request validation in Express?
**Answer:** Use validation middleware to ensure data integrity and security.

**Real-time Example:**
```javascript
const express = require('express');
const Joi = require('joi');
const app = express();

app.use(express.json());

// Validation middleware
const validate = (schema) => {
    return (req, res, next) => {
        const { error, value } = schema.validate(req.body);
        
        if (error) {
            return res.status(400).json({
                error: 'Validation Error',
                details: error.details.map(d => d.message)
            });
        }
        
        req.validatedData = value;
        next();
    };
};

// User validation schema
const userSchema = Joi.object({
    name: Joi.string().min(2).max(50).required(),
    email: Joi.string().email().required(),
    age: Joi.number().integer().min(18).max(120),
    password: Joi.string().min(8).pattern(new RegExp('^(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9])')).required(),
    role: Joi.string().valid('user', 'admin', 'moderator').default('user')
});

// Product validation schema
const productSchema = Joi.object({
    name: Joi.string().min(1).max(100).required(),
    price: Joi.number().positive().required(),
    category: Joi.string().valid('electronics', 'clothing', 'books').required(),
    description: Joi.string().max(500),
    tags: Joi.array().items(Joi.string()).max(10)
});

// Route with validation
app.post('/users', validate(userSchema), (req, res) => {
    const userData = req.validatedData;
    res.status(201).json({ message: 'User created', user: userData });
});

app.post('/products', validate(productSchema), (req, res) => {
    const productData = req.validatedData;
    res.status(201).json({ message: 'Product created', product: productData });
});

// Custom validation middleware
const validateEmail = (req, res, next) => {
    const { email } = req.body;
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    
    if (!emailRegex.test(email)) {
        return res.status(400).json({ error: 'Invalid email format' });
    }
    next();
};

// Sanitization middleware
const sanitizeInput = (req, res, next) => {
    if (req.body) {
        // Remove HTML tags
        Object.keys(req.body).forEach(key => {
            if (typeof req.body[key] === 'string') {
                req.body[key] = req.body[key].replace(/<[^>]*>/g, '');
            }
        });
    }
    next();
};

app.use(sanitizeInput);

// Rate limiting validation
const rateLimit = require('express-rate-limit');
const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // limit each IP to 100 requests per windowMs
    message: 'Too many requests from this IP'
});

app.use('/api/', limiter);
```

---

## Error Handling

### Q1. How to implement comprehensive error handling in Express?
**Answer:** Create a centralized error handling system with custom error classes and middleware.

**Real-time Example:**
```javascript
const express = require('express');
const app = express();

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
    }
}

class NotFoundError extends AppError {
    constructor(message = 'Resource not found') {
        super(message, 404);
    }
}

class UnauthorizedError extends AppError {
    constructor(message = 'Unauthorized') {
        super(message, 401);
    }
}

class ForbiddenError extends AppError {
    constructor(message = 'Forbidden') {
        super(message, 403);
    }
}

// Error handling middleware
const errorHandler = (err, req, res, next) => {
    let error = { ...err };
    error.message = err.message;
    
    // Log error
    console.error('Error:', err);
    
    // Mongoose bad ObjectId
    if (err.name === 'CastError') {
        const message = 'Resource not found';
        error = new NotFoundError(message);
    }
    
    // Mongoose duplicate key
    if (err.code === 11000) {
        const message = 'Duplicate field value entered';
        error = new ValidationError(message);
    }
    
    // Mongoose validation error
    if (err.name === 'ValidationError') {
        const message = Object.values(err.errors).map(val => val.message).join(', ');
        error = new ValidationError(message);
    }
    
    // JWT errors
    if (err.name === 'JsonWebTokenError') {
        const message = 'Invalid token';
        error = new UnauthorizedError(message);
    }
    
    if (err.name === 'TokenExpiredError') {
        const message = 'Token expired';
        error = new UnauthorizedError(message);
    }
    
    // Send error response
    res.status(error.statusCode || 500).json({
        success: false,
        error: {
            message: error.message || 'Internal Server Error',
            ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
        }
    });
};

// Async error wrapper
const catchAsync = (fn) => {
    return (req, res, next) => {
        fn(req, res, next).catch(next);
    };
};

// Usage examples
app.get('/users/:id', catchAsync(async (req, res, next) => {
    const user = await User.findById(req.params.id);
    
    if (!user) {
        return next(new NotFoundError('User not found'));
    }
    
    res.json({ success: true, data: user });
}));

app.post('/users', catchAsync(async (req, res, next) => {
    const { email, password } = req.body;
    
    if (!email || !password) {
        return next(new ValidationError('Email and password are required'));
    }
    
    const existingUser = await User.findOne({ email });
    if (existingUser) {
        return next(new ValidationError('User already exists'));
    }
    
    const user = await User.create(req.body);
    res.status(201).json({ success: true, data: user });
}));

// Global error handler
app.use(errorHandler);

// 404 handler
app.use('*', (req, res, next) => {
    next(new NotFoundError(`Route ${req.originalUrl} not found`));
});
```

### Q2. How to handle async errors in Express?
**Answer:** Use try-catch blocks or async error wrapper to handle async operations.

**Real-time Example:**
```javascript
const express = require('express');
const app = express();

// Method 1: Try-catch in each route
app.get('/users/:id', async (req, res, next) => {
    try {
        const user = await User.findById(req.params.id);
        
        if (!user) {
            return res.status(404).json({ error: 'User not found' });
        }
        
        res.json(user);
    } catch (error) {
        next(error); // Pass to error handler
    }
});

// Method 2: Async wrapper function
const asyncHandler = (fn) => {
    return (req, res, next) => {
        Promise.resolve(fn(req, res, next)).catch(next);
    };
};

app.get('/products/:id', asyncHandler(async (req, res) => {
    const product = await Product.findById(req.params.id);
    
    if (!product) {
        throw new Error('Product not found');
    }
    
    res.json(product);
}));

// Method 3: Express-async-errors package
require('express-async-errors');

app.get('/orders/:id', async (req, res) => {
    const order = await Order.findById(req.params.id);
    
    if (!order) {
        throw new Error('Order not found');
    }
    
    res.json(order);
});

// Method 4: Custom async middleware
const asyncMiddleware = (fn) => {
    return (req, res, next) => {
        fn(req, res, next).catch(err => {
            console.error('Async error:', err);
            next(err);
        });
    };
};

app.post('/users', asyncMiddleware(async (req, res) => {
    const user = await User.create(req.body);
    res.status(201).json(user);
}));
```

### Q3. How to implement error logging in Express?
**Answer:** Use logging libraries like Winston or Morgan to track and monitor errors.

**Real-time Example:**
```javascript
const express = require('express');
const winston = require('winston');
const morgan = require('morgan');
const app = express();

// Winston logger configuration
const logger = winston.createLogger({
    level: 'info',
    format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.errors({ stack: true }),
        winston.format.json()
    ),
    transports: [
        new winston.transports.File({ filename: 'error.log', level: 'error' }),
        new winston.transports.File({ filename: 'combined.log' }),
        new winston.transports.Console({
            format: winston.format.simple()
        })
    ]
});

// Morgan HTTP request logger
app.use(morgan('combined', {
    stream: {
        write: (message) => logger.info(message.trim())
    }
}));

// Error logging middleware
const errorLogger = (err, req, res, next) => {
    logger.error({
        message: err.message,
        stack: err.stack,
        url: req.url,
        method: req.method,
        ip: req.ip,
        userAgent: req.get('User-Agent'),
        timestamp: new Date().toISOString()
    });
    
    next(err);
};

// Application error handler
const errorHandler = (err, req, res, next) => {
    logger.error('Application Error:', err);
    
    res.status(err.statusCode || 500).json({
        error: {
            message: err.message || 'Internal Server Error',
            ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
        }
    });
};

app.use(errorLogger);
app.use(errorHandler);

// Unhandled promise rejections
process.on('unhandledRejection', (reason, promise) => {
    logger.error('Unhandled Rejection at:', promise, 'reason:', reason);
    process.exit(1);
});

// Uncaught exceptions
process.on('uncaughtException', (error) => {
    logger.error('Uncaught Exception:', error);
    process.exit(1);
});

// Graceful shutdown
process.on('SIGTERM', () => {
    logger.info('SIGTERM received, shutting down gracefully');
    process.exit(0);
});

process.on('SIGINT', () => {
    logger.info('SIGINT received, shutting down gracefully');
    process.exit(0);
});
```

---

## Authentication & Security

### Q1. How to implement JWT authentication in Express?
**Answer:** Use JSON Web Tokens for stateless authentication with proper security measures.

**Real-time Example:**
```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');
const rateLimit = require('express-rate-limit');
const helmet = require('helmet');
const app = express();

app.use(helmet());
app.use(express.json());

// Rate limiting for auth routes
const authLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 5, // limit each IP to 5 requests per windowMs
    message: 'Too many authentication attempts'
});

// JWT utilities
const generateToken = (payload) => {
    return jwt.sign(payload, process.env.JWT_SECRET, {
        expiresIn: process.env.JWT_EXPIRES_IN || '24h'
    });
};

const generateRefreshToken = (payload) => {
    return jwt.sign(payload, process.env.JWT_REFRESH_SECRET, {
        expiresIn: '7d'
    });
};

// Authentication middleware
const authenticate = async (req, res, next) => {
    try {
        const token = req.headers.authorization?.split(' ')[1];
        
        if (!token) {
            return res.status(401).json({ error: 'Access token required' });
        }
        
        const decoded = jwt.verify(token, process.env.JWT_SECRET);
        const user = await User.findById(decoded.id).select('-password');
        
        if (!user) {
            return res.status(401).json({ error: 'User not found' });
        }
        
        req.user = user;
        next();
    } catch (error) {
        if (error.name === 'TokenExpiredError') {
            return res.status(401).json({ error: 'Token expired' });
        }
        return res.status(401).json({ error: 'Invalid token' });
    }
};

// Authorization middleware
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

// Login route
app.post('/auth/login', authLimiter, async (req, res) => {
    try {
        const { email, password } = req.body;
        
        if (!email || !password) {
            return res.status(400).json({ error: 'Email and password required' });
        }
        
        const user = await User.findOne({ email }).select('+password');
        
        if (!user || !(await bcrypt.compare(password, user.password))) {
            return res.status(401).json({ error: 'Invalid credentials' });
        }
        
        const token = generateToken({ id: user._id, role: user.role });
        const refreshToken = generateRefreshToken({ id: user._id });
        
        // Store refresh token in database
        await User.findByIdAndUpdate(user._id, { refreshToken });
        
        res.json({
            success: true,
            token,
            refreshToken,
            user: {
                id: user._id,
                name: user.name,
                email: user.email,
                role: user.role
            }
        });
    } catch (error) {
        res.status(500).json({ error: 'Login failed' });
    }
});

// Register route
app.post('/auth/register', async (req, res) => {
    try {
        const { name, email, password, role = 'user' } = req.body;
        
        // Validate input
        if (!name || !email || !password) {
            return res.status(400).json({ error: 'All fields required' });
        }
        
        if (password.length < 8) {
            return res.status(400).json({ error: 'Password must be at least 8 characters' });
        }
        
        // Check if user exists
        const existingUser = await User.findOne({ email });
        if (existingUser) {
            return res.status(409).json({ error: 'User already exists' });
        }
        
        // Hash password
        const hashedPassword = await bcrypt.hash(password, 12);
        
        // Create user
        const user = await User.create({
            name,
            email,
            password: hashedPassword,
            role
        });
        
        const token = generateToken({ id: user._id, role: user.role });
        
        res.status(201).json({
            success: true,
            token,
            user: {
                id: user._id,
                name: user.name,
                email: user.email,
                role: user.role
            }
        });
    } catch (error) {
        res.status(500).json({ error: 'Registration failed' });
    }
});

// Refresh token route
app.post('/auth/refresh', async (req, res) => {
    try {
        const { refreshToken } = req.body;
        
        if (!refreshToken) {
            return res.status(400).json({ error: 'Refresh token required' });
        }
        
        const decoded = jwt.verify(refreshToken, process.env.JWT_REFRESH_SECRET);
        const user = await User.findById(decoded.id);
        
        if (!user || user.refreshToken !== refreshToken) {
            return res.status(401).json({ error: 'Invalid refresh token' });
        }
        
        const newToken = generateToken({ id: user._id, role: user.role });
        const newRefreshToken = generateRefreshToken({ id: user._id });
        
        await User.findByIdAndUpdate(user._id, { refreshToken: newRefreshToken });
        
        res.json({
            success: true,
            token: newToken,
            refreshToken: newRefreshToken
        });
    } catch (error) {
        res.status(401).json({ error: 'Invalid refresh token' });
    }
});

// Logout route
app.post('/auth/logout', authenticate, async (req, res) => {
    try {
        await User.findByIdAndUpdate(req.user._id, { refreshToken: null });
        res.json({ success: true, message: 'Logged out successfully' });
    } catch (error) {
        res.status(500).json({ error: 'Logout failed' });
    }
});

// Protected routes
app.get('/profile', authenticate, (req, res) => {
    res.json({ user: req.user });
});

app.get('/admin', authenticate, authorize('admin'), (req, res) => {
    res.json({ message: 'Admin access granted' });
});
```

### Q2. How to implement security measures in Express?
**Answer:** Use multiple security layers including Helmet, CORS, rate limiting, and input validation.

**Real-time Example:**
```javascript
const express = require('express');
const helmet = require('helmet');
const cors = require('cors');
const rateLimit = require('express-rate-limit');
const mongoSanitize = require('express-mongo-sanitize');
const xss = require('xss-clean');
const hpp = require('hpp');
const app = express();

// Security headers
app.use(helmet({
    contentSecurityPolicy: {
        directives: {
            defaultSrc: ["'self'"],
            styleSrc: ["'self'", "'unsafe-inline'"],
            scriptSrc: ["'self'"],
            imgSrc: ["'self'", "data:", "https:"],
        },
    },
    crossOriginEmbedderPolicy: false
}));

// CORS configuration
app.use(cors({
    origin: process.env.ALLOWED_ORIGINS?.split(',') || ['http://localhost:3000'],
    credentials: true,
    methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
    allowedHeaders: ['Content-Type', 'Authorization']
}));

// Rate limiting
const generalLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // limit each IP to 100 requests per windowMs
    message: 'Too many requests from this IP'
});

const authLimiter = rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 5,
    message: 'Too many authentication attempts'
});

app.use('/api/', generalLimiter);
app.use('/api/auth/', authLimiter);

// Body parsing with limits
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true, limit: '10mb' }));

// Data sanitization
app.use(mongoSanitize()); // Prevent NoSQL injection
app.use(xss()); // Prevent XSS attacks

// Prevent parameter pollution
app.use(hpp());

// Input validation middleware
const validateInput = (schema) => {
    return (req, res, next) => {
        const { error } = schema.validate(req.body);
        if (error) {
            return res.status(400).json({
                error: 'Validation Error',
                details: error.details.map(d => d.message)
            });
        }
        next();
    };
};

// Password strength validation
const validatePassword = (req, res, next) => {
    const { password } = req.body;
    
    if (!password) {
        return res.status(400).json({ error: 'Password required' });
    }
    
    const passwordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/;
    
    if (!passwordRegex.test(password)) {
        return res.status(400).json({
            error: 'Password must contain at least 8 characters, one uppercase, one lowercase, one number, and one special character'
        });
    }
    
    next();
};

// SQL injection prevention
const escapeHtml = (unsafe) => {
    return unsafe
        .replace(/&/g, "&amp;")
        .replace(/</g, "&lt;")
        .replace(/>/g, "&gt;")
        .replace(/"/g, "&quot;")
        .replace(/'/g, "&#039;");
};

// Request sanitization
app.use((req, res, next) => {
    if (req.body) {
        Object.keys(req.body).forEach(key => {
            if (typeof req.body[key] === 'string') {
                req.body[key] = escapeHtml(req.body[key]);
            }
        });
    }
    next();
});

// Security headers middleware
app.use((req, res, next) => {
    res.setHeader('X-Content-Type-Options', 'nosniff');
    res.setHeader('X-Frame-Options', 'DENY');
    res.setHeader('X-XSS-Protection', '1; mode=block');
    res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
    next();
});

// Session security
const session = require('express-session');
const MongoStore = require('connect-mongo');

app.use(session({
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    store: MongoStore.create({
        mongoUrl: process.env.MONGODB_URI,
        touchAfter: 24 * 3600 // lazy session update
    }),
    cookie: {
        secure: process.env.NODE_ENV === 'production',
        httpOnly: true,
        maxAge: 24 * 60 * 60 * 1000 // 24 hours
    }
}));

// CSRF protection
const csrf = require('csurf');
app.use(csrf());

app.use((req, res, next) => {
    res.locals.csrfToken = req.csrfToken();
    next();
});
```

---

## RESTful API Development

### Q1. How to design RESTful APIs in Express?
**Answer:** Follow REST principles with proper HTTP methods, status codes, and resource-based URLs.

**Real-time Example:**
```javascript
const express = require('express');
const app = express();

// Resource-based URL design
// GET /api/users - Get all users
app.get('/api/users', async (req, res) => {
    try {
        const { page = 1, limit = 10, sort = 'createdAt', order = 'desc' } = req.query;
        
        const users = await User.find()
            .select('-password')
            .sort({ [sort]: order === 'desc' ? -1 : 1 })
            .limit(limit * 1)
            .skip((page - 1) * limit);
        
        const total = await User.countDocuments();
        
        res.json({
            success: true,
            data: users,
            pagination: {
                total,
                page: parseInt(page),
                pages: Math.ceil(total / limit),
                limit: parseInt(limit)
            }
        });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// GET /api/users/:id - Get specific user
app.get('/api/users/:id', async (req, res) => {
    try {
        const user = await User.findById(req.params.id).select('-password');
        
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
        const user = await User.create(req.body);
        
        res.status(201).json({
            success: true,
            data: {
                id: user._id,
                name: user.name,
                email: user.email,
                role: user.role,
                createdAt: user.createdAt
            }
        });
    } catch (error) {
        if (error.code === 11000) {
            return res.status(409).json({ error: 'User already exists' });
        }
        res.status(400).json({ error: error.message });
    }
});

// PUT /api/users/:id - Update entire user
app.put('/api/users/:id', async (req, res) => {
    try {
        const user = await User.findByIdAndUpdate(
            req.params.id,
            req.body,
            { new: true, runValidators: true }
        ).select('-password');
        
        if (!user) {
            return res.status(404).json({ error: 'User not found' });
        }
        
        res.json({ success: true, data: user });
    } catch (error) {
        res.status(400).json({ error: error.message });
    }
});

// PATCH /api/users/:id - Partial update
app.patch('/api/users/:id', async (req, res) => {
    try {
        const user = await User.findByIdAndUpdate(
            req.params.id,
            { $set: req.body },
            { new: true, runValidators: true }
        ).select('-password');
        
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
        
        res.status(204).send();
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// Nested resources
// GET /api/users/:id/orders - Get user's orders
app.get('/api/users/:id/orders', async (req, res) => {
    try {
        const orders = await Order.find({ userId: req.params.id })
            .populate('items.productId')
            .sort({ createdAt: -1 });
        
        res.json({ success: true, data: orders });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// POST /api/users/:id/orders - Create order for user
app.post('/api/users/:id/orders', async (req, res) => {
    try {
        const order = await Order.create({
            ...req.body,
            userId: req.params.id
        });
        
        res.status(201).json({ success: true, data: order });
    } catch (error) {
        res.status(400).json({ error: error.message });
    }
});
```

### Q2. How to implement API versioning in Express?
**Answer:** Use URL-based versioning with separate route modules for different API versions.

**Real-time Example:**
```javascript
const express = require('express');
const app = express();

// API Version 1
const v1Router = express.Router();

v1Router.get('/users', (req, res) => {
    res.json({ version: 'v1', message: 'Users endpoint' });
});

v1Router.get('/products', (req, res) => {
    res.json({ version: 'v1', message: 'Products endpoint' });
});

// API Version 2
const v2Router = express.Router();

v2Router.get('/users', (req, res) => {
    res.json({ 
        version: 'v2', 
        message: 'Enhanced users endpoint',
        features: ['pagination', 'filtering', 'sorting']
    });
});

v2Router.get('/products', (req, res) => {
    res.json({ 
        version: 'v2', 
        message: 'Enhanced products endpoint',
        features: ['search', 'categories', 'reviews']
    });
});

// Mount versioned routes
app.use('/api/v1', v1Router);
app.use('/api/v2', v2Router);

// Default version (latest)
app.use('/api', v2Router);

// Version detection middleware
const detectVersion = (req, res, next) => {
    const version = req.headers['api-version'] || req.query.version || 'v1';
    req.apiVersion = version;
    next();
};

app.use(detectVersion);

// Conditional routing based on version
app.get('/api/users', (req, res) => {
    if (req.apiVersion === 'v2') {
        return res.json({ 
            version: 'v2', 
            users: [],
            pagination: { page: 1, limit: 10 }
        });
    }
    
    res.json({ version: 'v1', users: [] });
});

// API versioning with feature flags
const features = {
    v1: ['basic_crud'],
    v2: ['basic_crud', 'pagination', 'filtering'],
    v3: ['basic_crud', 'pagination', 'filtering', 'search', 'analytics']
};

app.get('/api/features', (req, res) => {
    const version = req.apiVersion;
    res.json({
        version,
        features: features[version] || features.v1
    });
});
```

---

## Database Integration

### Q1. How to integrate MongoDB with Express?
**Answer:** Use Mongoose ODM for MongoDB integration with proper connection handling and schema design.

**Real-time Example:**
```javascript
const express = require('express');
const mongoose = require('mongoose');
const app = express();

// Database connection
const connectDB = async () => {
    try {
        const conn = await mongoose.connect(process.env.MONGODB_URI, {
            useNewUrlParser: true,
            useUnifiedTopology: true,
            maxPoolSize: 10,
            serverSelectionTimeoutMS: 5000,
            socketTimeoutMS: 45000,
        });
        
        console.log(`MongoDB Connected: ${conn.connection.host}`);
    } catch (error) {
        console.error('Database connection error:', error);
        process.exit(1);
    }
};

// User Schema
const userSchema = new mongoose.Schema({
    name: {
        type: String,
        required: [true, 'Name is required'],
        trim: true,
        maxlength: [50, 'Name cannot exceed 50 characters']
    },
    email: {
        type: String,
        required: [true, 'Email is required'],
        unique: true,
        lowercase: true,
        match: [/^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/, 'Please enter a valid email']
    },
    password: {
        type: String,
        required: [true, 'Password is required'],
        minlength: [8, 'Password must be at least 8 characters'],
        select: false
    },
    role: {
        type: String,
        enum: ['user', 'admin', 'moderator'],
        default: 'user'
    },
    isActive: {
        type: Boolean,
        default: true
    },
    lastLogin: {
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
    return `${this.name} (${this.role})`;
});

// Instance methods
userSchema.methods.comparePassword = async function(candidatePassword) {
    return await bcrypt.compare(candidatePassword, this.password);
};

// Static methods
userSchema.statics.findByEmail = function(email) {
    return this.findOne({ email: email.toLowerCase() });
};

// Pre-save middleware
userSchema.pre('save', async function(next) {
    if (!this.isModified('password')) return next();
    
    this.password = await bcrypt.hash(this.password, 12);
    next();
});

// Post-save middleware
userSchema.post('save', function(doc) {
    console.log(`User ${doc.name} saved successfully`);
});

const User = mongoose.model('User', userSchema);

// Product Schema with references
const productSchema = new mongoose.Schema({
    name: {
        type: String,
        required: true,
        trim: true
    },
    price: {
        type: Number,
        required: true,
        min: [0, 'Price cannot be negative']
    },
    category: {
        type: String,
        required: true,
        enum: ['electronics', 'clothing', 'books', 'home']
    },
    description: String,
    images: [String],
    inStock: {
        type: Boolean,
        default: true
    },
    quantity: {
        type: Number,
        default: 0,
        min: [0, 'Quantity cannot be negative']
    },
    seller: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'User',
        required: true
    },
    tags: [String],
    ratings: {
        average: { type: Number, default: 0 },
        count: { type: Number, default: 0 }
    }
}, {
    timestamps: true
});

// Indexes
productSchema.index({ name: 'text', description: 'text' });
productSchema.index({ category: 1, price: 1 });
productSchema.index({ seller: 1 });

const Product = mongoose.model('Product', productSchema);

// Order Schema
const orderSchema = new mongoose.Schema({
    user: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'User',
        required: true
    },
    items: [{
        product: {
            type: mongoose.Schema.Types.ObjectId,
            ref: 'Product',
            required: true
        },
        quantity: {
            type: Number,
            required: true,
            min: 1
        },
        price: {
            type: Number,
            required: true
        }
    }],
    totalAmount: {
        type: Number,
        required: true
    },
    status: {
        type: String,
        enum: ['pending', 'confirmed', 'shipped', 'delivered', 'cancelled'],
        default: 'pending'
    },
    shippingAddress: {
        street: String,
        city: String,
        state: String,
        zipCode: String,
        country: String
    }
}, {
    timestamps: true
});

const Order = mongoose.model('Order', orderSchema);

// Database operations
app.get('/api/users', async (req, res) => {
    try {
        const users = await User.find()
            .select('-password')
            .populate('orders')
            .sort({ createdAt: -1 });
        
        res.json({ success: true, data: users });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

app.get('/api/products', async (req, res) => {
    try {
        const { category, minPrice, maxPrice, search, page = 1, limit = 10 } = req.query;
        
        let query = {};
        
        if (category) query.category = category;
        if (minPrice || maxPrice) {
            query.price = {};
            if (minPrice) query.price.$gte = Number(minPrice);
            if (maxPrice) query.price.$lte = Number(maxPrice);
        }
        if (search) {
            query.$text = { $search: search };
        }
        
        const products = await Product.find(query)
            .populate('seller', 'name email')
            .sort({ createdAt: -1 })
            .limit(limit * 1)
            .skip((page - 1) * limit);
        
        const total = await Product.countDocuments(query);
        
        res.json({
            success: true,
            data: products,
            pagination: {
                total,
                page: parseInt(page),
                pages: Math.ceil(total / limit)
            }
        });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// Connect to database
connectDB();
```

### Q2. How to implement database transactions in Express?
**Answer:** Use MongoDB transactions for atomic operations across multiple collections.

**Real-time Example:**
```javascript
const mongoose = require('mongoose');

// Transaction example: Create order with inventory update
app.post('/api/orders', async (req, res) => {
    const session = await mongoose.startSession();
    
    try {
        await session.withTransaction(async () => {
            const { userId, items } = req.body;
            
            // Calculate total amount
            let totalAmount = 0;
            const orderItems = [];
            
            for (const item of items) {
                const product = await Product.findById(item.productId).session(session);
                
                if (!product) {
                    throw new Error(`Product ${item.productId} not found`);
                }
                
                if (product.quantity < item.quantity) {
                    throw new Error(`Insufficient stock for product ${product.name}`);
                }
                
                const itemTotal = product.price * item.quantity;
                totalAmount += itemTotal;
                
                orderItems.push({
                    product: item.productId,
                    quantity: item.quantity,
                    price: product.price
                });
            }
            
            // Create order
            const order = await Order.create([{
                user: userId,
                items: orderItems,
                totalAmount,
                status: 'pending'
            }], { session });
            
            // Update product quantities
            for (const item of items) {
                await Product.findByIdAndUpdate(
                    item.productId,
                    { $inc: { quantity: -item.quantity } },
                    { session }
                );
            }
            
            res.status(201).json({
                success: true,
                data: order[0]
            });
        });
    } catch (error) {
        res.status(400).json({ error: error.message });
    } finally {
        await session.endSession();
    }
});

// Bulk operations with transaction
app.post('/api/products/bulk-update', async (req, res) => {
    const session = await mongoose.startSession();
    
    try {
        await session.withTransaction(async () => {
            const { updates } = req.body;
            
            const bulkOps = updates.map(update => ({
                updateOne: {
                    filter: { _id: update.id },
                    update: { $set: update.data }
                }
            }));
            
            const result = await Product.bulkWrite(bulkOps, { session });
            
            res.json({
                success: true,
                modifiedCount: result.modifiedCount
            });
        });
    } catch (error) {
        res.status(400).json({ error: error.message });
    } finally {
        await session.endSession();
    }
});
```

---

## File Upload & Streaming

### Q1. How to handle file uploads in Express?
**Answer:** Use Multer middleware for file uploads with proper validation and storage options.

**Real-time Example:**
```javascript
const express = require('express');
const multer = require('multer');
const path = require('path');
const fs = require('fs');
const sharp = require('sharp');
const app = express();

// Create uploads directory
const uploadDir = './uploads';
if (!fs.existsSync(uploadDir)) {
    fs.mkdirSync(uploadDir, { recursive: true });
}

// Storage configuration
const storage = multer.diskStorage({
    destination: (req, file, cb) => {
        const uploadPath = path.join(uploadDir, file.fieldname);
        if (!fs.existsSync(uploadPath)) {
            fs.mkdirSync(uploadPath, { recursive: true });
        }
        cb(null, uploadPath);
    },
    filename: (req, file, cb) => {
        const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
        cb(null, file.fieldname + '-' + uniqueSuffix + path.extname(file.originalname));
    }
});

// File filter
const fileFilter = (req, file, cb) => {
    const allowedTypes = {
        'image/jpeg': 'jpg',
        'image/png': 'png',
        'image/gif': 'gif',
        'application/pdf': 'pdf',
        'text/plain': 'txt'
    };
    
    if (allowedTypes[file.mimetype]) {
        cb(null, true);
    } else {
        cb(new Error('Invalid file type'), false);
    }
};

// Multer configuration
const upload = multer({
    storage: storage,
    limits: {
        fileSize: 5 * 1024 * 1024, // 5MB limit
        files: 5 // Maximum 5 files
    },
    fileFilter: fileFilter
});

// Single file upload
app.post('/upload/single', upload.single('file'), (req, res) => {
    try {
        if (!req.file) {
            return res.status(400).json({ error: 'No file uploaded' });
        }
        
        res.json({
            success: true,
            file: {
                filename: req.file.filename,
                originalname: req.file.originalname,
                size: req.file.size,
                path: req.file.path
            }
        });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// Multiple files upload
app.post('/upload/multiple', upload.array('files', 5), (req, res) => {
    try {
        if (!req.files || req.files.length === 0) {
            return res.status(400).json({ error: 'No files uploaded' });
        }
        
        const files = req.files.map(file => ({
            filename: file.filename,
            originalname: file.originalname,
            size: file.size,
            path: file.path
        }));
        
        res.json({
            success: true,
            files: files,
            count: files.length
        });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// Mixed upload (files and fields)
app.post('/upload/mixed', upload.fields([
    { name: 'avatar', maxCount: 1 },
    { name: 'gallery', maxCount: 5 },
    { name: 'documents', maxCount: 3 }
]), (req, res) => {
    try {
        const { name, email } = req.body;
        const files = req.files;
        
        res.json({
            success: true,
            user: { name, email },
            files: {
                avatar: files.avatar?.[0],
                gallery: files.gallery,
                documents: files.documents
            }
        });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// Image processing with Sharp
app.post('/upload/image', upload.single('image'), async (req, res) => {
    try {
        if (!req.file) {
            return res.status(400).json({ error: 'No image uploaded' });
        }
        
        const { filename, path: inputPath } = req.file;
        const outputDir = path.join(uploadDir, 'processed');
        
        if (!fs.existsSync(outputDir)) {
            fs.mkdirSync(outputDir, { recursive: true });
        }
        
        // Create different sizes
        const sizes = [
            { name: 'thumbnail', width: 150, height: 150 },
            { name: 'medium', width: 500, height: 500 },
            { name: 'large', width: 1200, height: 1200 }
        ];
        
        const processedImages = [];
        
        for (const size of sizes) {
            const outputPath = path.join(outputDir, `${size.name}-${filename}`);
            
            await sharp(inputPath)
                .resize(size.width, size.height, { fit: 'inside' })
                .jpeg({ quality: 90 })
                .toFile(outputPath);
            
            processedImages.push({
                size: size.name,
                path: outputPath,
                dimensions: `${size.width}x${size.height}`
            });
        }
        
        res.json({
            success: true,
            original: req.file,
            processed: processedImages
        });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// File download
app.get('/download/:filename', (req, res) => {
    const filename = req.params.filename;
    const filePath = path.join(uploadDir, filename);
    
    if (!fs.existsSync(filePath)) {
        return res.status(404).json({ error: 'File not found' });
    }
    
    res.download(filePath, (err) => {
        if (err) {
            res.status(500).json({ error: 'Download failed' });
        }
    });
});

// File streaming
app.get('/stream/:filename', (req, res) => {
    const filename = req.params.filename;
    const filePath = path.join(uploadDir, filename);
    
    if (!fs.existsSync(filePath)) {
        return res.status(404).json({ error: 'File not found' });
    }
    
    const stat = fs.statSync(filePath);
    const fileSize = stat.size;
    const range = req.headers.range;
    
    if (range) {
        const parts = range.replace(/bytes=/, "").split("-");
        const start = parseInt(parts[0], 10);
        const end = parts[1] ? parseInt(parts[1], 10) : fileSize - 1;
        const chunksize = (end - start) + 1;
        const file = fs.createReadStream(filePath, { start, end });
        const head = {
            'Content-Range': `bytes ${start}-${end}/${fileSize}`,
            'Accept-Ranges': 'bytes',
            'Content-Length': chunksize,
            'Content-Type': 'video/mp4',
        };
        res.writeHead(206, head);
        file.pipe(res);
    } else {
        const head = {
            'Content-Length': fileSize,
            'Content-Type': 'video/mp4',
        };
        res.writeHead(200, head);
        fs.createReadStream(filePath).pipe(res);
    }
});
```

---

## Testing

### Q1. How to test Express applications?
**Answer:** Use Jest for unit testing and Supertest for integration testing with proper test structure.

**Real-time Example:**
```javascript
// tests/app.test.js
const request = require('supertest');
const app = require('../app');
const User = require('../models/User');
const { connectDB, disconnectDB } = require('../config/database');

describe('Express App Tests', () => {
    beforeAll(async () => {
        await connectDB();
    });
    
    afterAll(async () => {
        await disconnectDB();
    });
    
    beforeEach(async () => {
        await User.deleteMany({});
    });
    
    describe('GET /api/users', () => {
        it('should get all users', async () => {
            const users = [
                { name: 'John', email: 'john@example.com' },
                { name: 'Jane', email: 'jane@example.com' }
            ];
            
            await User.insertMany(users);
            
            const response = await request(app)
                .get('/api/users')
                .expect(200);
            
            expect(response.body.success).toBe(true);
            expect(response.body.data).toHaveLength(2);
        });
        
        it('should return empty array when no users', async () => {
            const response = await request(app)
                .get('/api/users')
                .expect(200);
            
            expect(response.body.data).toHaveLength(0);
        });
    });
    
    describe('POST /api/users', () => {
        it('should create a new user', async () => {
            const userData = {
                name: 'Test User',
                email: 'test@example.com',
                password: 'password123'
            };
            
            const response = await request(app)
                .post('/api/users')
                .send(userData)
                .expect(201);
            
            expect(response.body.success).toBe(true);
            expect(response.body.data.name).toBe(userData.name);
            expect(response.body.data.email).toBe(userData.email);
        });
        
        it('should return 400 for invalid data', async () => {
            const invalidData = {
                name: '',
                email: 'invalid-email'
            };
            
            const response = await request(app)
                .post('/api/users')
                .send(invalidData)
                .expect(400);
            
            expect(response.body.error).toBeDefined();
        });
    });
    
    describe('GET /api/users/:id', () => {
        it('should get user by id', async () => {
            const user = await User.create({
                name: 'Test User',
                email: 'test@example.com',
                password: 'password123'
            });
            
            const response = await request(app)
                .get(`/api/users/${user._id}`)
                .expect(200);
            
            expect(response.body.success).toBe(true);
            expect(response.body.data.name).toBe(user.name);
        });
        
        it('should return 404 for non-existent user', async () => {
            const fakeId = '507f1f77bcf86cd799439011';
            
            const response = await request(app)
                .get(`/api/users/${fakeId}`)
                .expect(404);
            
            expect(response.body.error).toBe('User not found');
        });
    });
    
    describe('PUT /api/users/:id', () => {
        it('should update user', async () => {
            const user = await User.create({
                name: 'Test User',
                email: 'test@example.com',
                password: 'password123'
            });
            
            const updateData = { name: 'Updated User' };
            
            const response = await request(app)
                .put(`/api/users/${user._id}`)
                .send(updateData)
                .expect(200);
            
            expect(response.body.success).toBe(true);
            expect(response.body.data.name).toBe(updateData.name);
        });
    });
    
    describe('DELETE /api/users/:id', () => {
        it('should delete user', async () => {
            const user = await User.create({
                name: 'Test User',
                email: 'test@example.com',
                password: 'password123'
            });
            
            await request(app)
                .delete(`/api/users/${user._id}`)
                .expect(204);
            
            const deletedUser = await User.findById(user._id);
            expect(deletedUser).toBeNull();
        });
    });
});

// tests/middleware.test.js
describe('Middleware Tests', () => {
    describe('Authentication Middleware', () => {
        it('should authenticate valid token', async () => {
            const user = await User.create({
                name: 'Test User',
                email: 'test@example.com',
                password: 'password123'
            });
            
            const token = jwt.sign(
                { id: user._id },
                process.env.JWT_SECRET,
                { expiresIn: '1h' }
            );
            
            const response = await request(app)
                .get('/api/profile')
                .set('Authorization', `Bearer ${token}`)
                .expect(200);
            
            expect(response.body.user._id).toBe(user._id.toString());
        });
        
        it('should reject invalid token', async () => {
            const response = await request(app)
                .get('/api/profile')
                .set('Authorization', 'Bearer invalid-token')
                .expect(401);
            
            expect(response.body.error).toBe('Invalid token');
        });
        
        it('should reject missing token', async () => {
            const response = await request(app)
                .get('/api/profile')
                .expect(401);
            
            expect(response.body.error).toBe('Access token required');
        });
    });
    
    describe('Validation Middleware', () => {
        it('should validate user data', async () => {
            const invalidData = {
                name: '',
                email: 'invalid-email',
                password: '123'
            };
            
            const response = await request(app)
                .post('/api/users')
                .send(invalidData)
                .expect(400);
            
            expect(response.body.error).toBe('Validation Error');
            expect(response.body.details).toBeDefined();
        });
    });
});

// tests/integration.test.js
describe('Integration Tests', () => {
    describe('User Workflow', () => {
        it('should complete full user workflow', async () => {
            // Create user
            const userData = {
                name: 'Integration Test User',
                email: 'integration@example.com',
                password: 'password123'
            };
            
            const createResponse = await request(app)
                .post('/api/users')
                .send(userData)
                .expect(201);
            
            const userId = createResponse.body.data.id;
            
            // Get user
            const getResponse = await request(app)
                .get(`/api/users/${userId}`)
                .expect(200);
            
            expect(getResponse.body.data.name).toBe(userData.name);
            
            // Update user
            const updateData = { name: 'Updated Integration User' };
            const updateResponse = await request(app)
                .put(`/api/users/${userId}`)
                .send(updateData)
                .expect(200);
            
            expect(updateResponse.body.data.name).toBe(updateData.name);
            
            // Delete user
            await request(app)
                .delete(`/api/users/${userId}`)
                .expect(204);
            
            // Verify deletion
            await request(app)
                .get(`/api/users/${userId}`)
                .expect(404);
        });
    });
});

// jest.config.js
module.exports = {
    testEnvironment: 'node',
    setupFilesAfterEnv: ['<rootDir>/tests/setup.js'],
    testMatch: ['**/tests/**/*.test.js'],
    collectCoverageFrom: [
        '**/*.js',
        '!**/node_modules/**',
        '!**/tests/**',
        '!**/coverage/**'
    ],
    coverageThreshold: {
        global: {
            branches: 80,
            functions: 80,
            lines: 80,
            statements: 80
        }
    }
};

// tests/setup.js
const mongoose = require('mongoose');

beforeAll(async () => {
    // Setup test database
    await mongoose.connect(process.env.MONGODB_TEST_URI);
});

afterAll(async () => {
    // Cleanup
    await mongoose.connection.dropDatabase();
    await mongoose.connection.close();
});
```

---

## Performance & Best Practices

### Q1. How to optimize Express application performance?
**Answer:** Implement caching, compression, clustering, and database optimization techniques.

**Real-time Example:**
```javascript
const express = require('express');
const compression = require('compression');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const cluster = require('cluster');
const numCPUs = require('os').cpus().length;
const redis = require('redis');
const app = express();

// Clustering for multi-core utilization
if (cluster.isMaster) {
    console.log(`Master ${process.pid} is running`);
    
    // Fork workers
    for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
    }
    
    cluster.on('exit', (worker, code, signal) => {
        console.log(`Worker ${worker.process.pid} died`);
        cluster.fork(); // Restart worker
    });
} else {
    // Worker process
    console.log(`Worker ${process.pid} started`);
    
    // Performance middleware
    app.use(compression());
    app.use(helmet());
    
    // Rate limiting
    const limiter = rateLimit({
        windowMs: 15 * 60 * 1000, // 15 minutes
        max: 100, // limit each IP to 100 requests per windowMs
        message: 'Too many requests from this IP'
    });
    app.use('/api/', limiter);
    
    // Redis caching
    const redisClient = redis.createClient({
        host: process.env.REDIS_HOST,
        port: process.env.REDIS_PORT
    });
    
    // Cache middleware
    const cache = (duration) => {
        return (req, res, next) => {
            if (req.method !== 'GET') {
                return next();
            }
            
            const key = `cache:${req.originalUrl}`;
            
            redisClient.get(key, (err, data) => {
                if (err) {
                    console.error('Redis error:', err);
                    return next();
                }
                
                if (data) {
                    res.json(JSON.parse(data));
                } else {
                    const originalJson = res.json.bind(res);
                    res.json = (data) => {
                        redisClient.setex(key, duration, JSON.stringify(data));
                        originalJson(data);
                    };
                    next();
                }
            });
        };
    };
    
    // Database connection pooling
    const mongoose = require('mongoose');
    mongoose.connect(process.env.MONGODB_URI, {
        maxPoolSize: 10,
        serverSelectionTimeoutMS: 5000,
        socketTimeoutMS: 45000,
        bufferCommands: false,
        bufferMaxEntries: 0
    });
    
    // Query optimization
    app.get('/api/users', cache(300), async (req, res) => {
        try {
            const { page = 1, limit = 10, sort = 'createdAt' } = req.query;
            
            const users = await User.find()
                .select('-password')
                .sort({ [sort]: -1 })
                .limit(limit * 1)
                .skip((page - 1) * limit)
                .lean(); // Use lean() for better performance
            
            const total = await User.countDocuments();
            
            res.json({
                success: true,
                data: users,
                pagination: {
                    total,
                    page: parseInt(page),
                    pages: Math.ceil(total / limit)
                }
            });
        } catch (error) {
            res.status(500).json({ error: error.message });
        }
    });
    
    // Database indexing
    const userSchema = new mongoose.Schema({
        name: String,
        email: { type: String, unique: true },
        createdAt: { type: Date, default: Date.now }
    });
    
    // Create indexes
    userSchema.index({ email: 1 });
    userSchema.index({ createdAt: -1 });
    userSchema.index({ name: 'text' }); // Text search
    
    // Connection pooling
    const connectionOptions = {
        maxPoolSize: 10,
        minPoolSize: 2,
        maxIdleTimeMS: 30000,
        serverSelectionTimeoutMS: 5000,
        socketTimeoutMS: 45000
    };
    
    // Memory optimization
    app.use(express.json({ limit: '1mb' }));
    app.use(express.urlencoded({ extended: true, limit: '1mb' }));
    
    // Response compression
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
    
    // Static file serving with caching
    app.use(express.static('public', {
        maxAge: '1d',
        etag: true,
        lastModified: true
    }));
    
    // Database query optimization
    app.get('/api/products/search', cache(600), async (req, res) => {
        try {
            const { q, category, minPrice, maxPrice } = req.query;
            
            let query = {};
            
            if (q) {
                query.$text = { $search: q };
            }
            
            if (category) {
                query.category = category;
            }
            
            if (minPrice || maxPrice) {
                query.price = {};
                if (minPrice) query.price.$gte = Number(minPrice);
                if (maxPrice) query.price.$lte = Number(maxPrice);
            }
            
            const products = await Product.find(query)
                .select('name price category')
                .sort({ score: { $meta: 'textScore' } })
                .limit(20)
                .lean();
            
            res.json({ success: true, data: products });
        } catch (error) {
            res.status(500).json({ error: error.message });
        }
    });
    
    // Start server
    const PORT = process.env.PORT || 3000;
    app.listen(PORT, () => {
        console.log(`Worker ${process.pid} listening on port ${PORT}`);
    });
}
```

### Q2. How to implement monitoring and logging in Express?
**Answer:** Use comprehensive logging, monitoring, and health checks for production applications.

**Real-time Example:**
```javascript
const express = require('express');
const winston = require('winston');
const prometheus = require('prom-client');
const app = express();

// Prometheus metrics
const register = new prometheus.Registry();
prometheus.collectDefaultMetrics({ register });

// Custom metrics
const httpRequestDuration = new prometheus.Histogram({
    name: 'http_request_duration_seconds',
    help: 'Duration of HTTP requests in seconds',
    labelNames: ['method', 'route', 'status_code'],
    buckets: [0.1, 0.3, 0.5, 0.7, 1, 3, 5, 7, 10]
});

const httpRequestTotal = new prometheus.Counter({
    name: 'http_requests_total',
    help: 'Total number of HTTP requests',
    labelNames: ['method', 'route', 'status_code']
});

register.registerMetric(httpRequestDuration);
register.registerMetric(httpRequestTotal);

// Winston logger
const logger = winston.createLogger({
    level: 'info',
    format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.errors({ stack: true }),
        winston.format.json()
    ),
    transports: [
        new winston.transports.File({ filename: 'error.log', level: 'error' }),
        new winston.transports.File({ filename: 'combined.log' }),
        new winston.transports.Console({
            format: winston.format.simple()
        })
    ]
});

// Metrics middleware
app.use((req, res, next) => {
    const start = Date.now();
    
    res.on('finish', () => {
        const duration = (Date.now() - start) / 1000;
        const route = req.route ? req.route.path : req.path;
        
        httpRequestDuration
            .labels(req.method, route, res.statusCode)
            .observe(duration);
        
        httpRequestTotal
            .labels(req.method, route, res.statusCode)
            .inc();
        
        logger.info('HTTP Request', {
            method: req.method,
            url: req.url,
            statusCode: res.statusCode,
            duration: duration,
            userAgent: req.get('User-Agent'),
            ip: req.ip
        });
    });
    
    next();
});

// Health check endpoint
app.get('/health', (req, res) => {
    const healthCheck = {
        uptime: process.uptime(),
        message: 'OK',
        timestamp: Date.now(),
        memory: process.memoryUsage(),
        cpu: process.cpuUsage()
    };
    
    res.status(200).json(healthCheck);
});

// Metrics endpoint
app.get('/metrics', (req, res) => {
    res.set('Content-Type', register.contentType);
    res.end(register.metrics());
});

// Performance monitoring
app.use((req, res, next) => {
    const start = process.hrtime();
    
    res.on('finish', () => {
        const diff = process.hrtime(start);
        const responseTime = diff[0] * 1000 + diff[1] * 1e-6;
        
        if (responseTime > 1000) { // Log slow requests
            logger.warn('Slow request detected', {
                method: req.method,
                url: req.url,
                responseTime: responseTime,
                statusCode: res.statusCode
            });
        }
    });
    
    next();
});

// Error tracking
app.use((err, req, res, next) => {
    logger.error('Application Error', {
        error: err.message,
        stack: err.stack,
        url: req.url,
        method: req.method,
        ip: req.ip,
        userAgent: req.get('User-Agent')
    });
    
    res.status(err.statusCode || 500).json({
        error: 'Internal Server Error',
        ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
    });
});

// Graceful shutdown
process.on('SIGTERM', () => {
    logger.info('SIGTERM received, shutting down gracefully');
    process.exit(0);
});

process.on('SIGINT', () => {
    logger.info('SIGINT received, shutting down gracefully');
    process.exit(0);
});

// Unhandled promise rejections
process.on('unhandledRejection', (reason, promise) => {
    logger.error('Unhandled Rejection', { reason, promise });
    process.exit(1);
});

// Uncaught exceptions
process.on('uncaughtException', (error) => {
    logger.error('Uncaught Exception', { error });
    process.exit(1);
});
```

**Total Express.js Questions: 100+ with comprehensive examples and best practices!**

---

## 🎯 PRACTICAL INTERVIEW QUESTIONS & ANSWERS

### 🔥 Most Asked Node.js/Express.js Interview Questions

#### **Q1. How would you implement a complete RESTful API with authentication in Node.js?**

**Answer:** Here's a comprehensive implementation:

```javascript
// 1. Server Setup with Express
const express = require('express');
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const cors = require('cors');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');

const app = express();

// Middleware
app.use(helmet());
app.use(cors());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Rate limiting
const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100 // limit each IP to 100 requests per windowMs
});
app.use('/api/', limiter);

// 2. User Model
const userSchema = new mongoose.Schema({
    name: { type: String, required: true },
    email: { type: String, required: true, unique: true },
    password: { type: String, required: true },
    role: { type: String, enum: ['user', 'admin'], default: 'user' },
    createdAt: { type: Date, default: Date.now }
});

userSchema.pre('save', async function(next) {
    if (!this.isModified('password')) return next();
    this.password = await bcrypt.hash(this.password, 12);
    next();
});

userSchema.methods.comparePassword = async function(candidatePassword) {
    return await bcrypt.compare(candidatePassword, this.password);
};

const User = mongoose.model('User', userSchema);

// 3. Authentication Middleware
const authenticateToken = (req, res, next) => {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1];

    if (!token) {
        return res.status(401).json({ error: 'Access token required' });
    }

    jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
        if (err) {
            return res.status(403).json({ error: 'Invalid token' });
        }
        req.user = user;
        next();
    });
};

// 4. Authentication Routes
app.post('/api/register', async (req, res) => {
    try {
        const { name, email, password } = req.body;

        // Check if user exists
        const existingUser = await User.findOne({ email });
        if (existingUser) {
            return res.status(409).json({ error: 'User already exists' });
        }

        // Create user
        const user = new User({ name, email, password });
        await user.save();

        // Generate token
        const token = jwt.sign(
            { userId: user._id, email: user.email },
            process.env.JWT_SECRET,
            { expiresIn: '24h' }
        );

        res.status(201).json({
            message: 'User created successfully',
            token,
            user: { id: user._id, name: user.name, email: user.email }
        });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

app.post('/api/login', async (req, res) => {
    try {
        const { email, password } = req.body;

        // Find user
        const user = await User.findOne({ email });
        if (!user) {
            return res.status(401).json({ error: 'Invalid credentials' });
        }

        // Check password
        const isPasswordValid = await user.comparePassword(password);
        if (!isPasswordValid) {
            return res.status(401).json({ error: 'Invalid credentials' });
        }

        // Generate token
        const token = jwt.sign(
            { userId: user._id, email: user.email },
            process.env.JWT_SECRET,
            { expiresIn: '24h' }
        );

        res.json({
            message: 'Login successful',
            token,
            user: { id: user._id, name: user.name, email: user.email }
        });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// 5. Protected Routes
app.get('/api/profile', authenticateToken, async (req, res) => {
    try {
        const user = await User.findById(req.user.userId).select('-password');
        res.json(user);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// Start server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
```

#### **Q2. How would you implement a real-time chat application using Socket.io?**

**Answer:** Complete real-time chat implementation:

```javascript
// 1. Server Setup with Socket.io
const express = require('express');
const http = require('http');
const socketIo = require('socket.io');
const mongoose = require('mongoose');

const app = express();
const server = http.createServer(app);
const io = socketIo(server, {
    cors: {
        origin: "*",
        methods: ["GET", "POST"]
    }
});

// 2. Message Model
const messageSchema = new mongoose.Schema({
    user: { type: String, required: true },
    message: { type: String, required: true },
    room: { type: String, required: true },
    timestamp: { type: Date, default: Date.now }
});

const Message = mongoose.model('Message', messageSchema);

// 3. Socket.io Connection Handling
io.on('connection', (socket) => {
    console.log('User connected:', socket.id);

    // Join room
    socket.on('join-room', (room) => {
        socket.join(room);
        console.log(`User ${socket.id} joined room ${room}`);
        
        // Send welcome message
        socket.emit('message', {
            user: 'System',
            message: 'Welcome to the chat!',
            timestamp: new Date()
        });
    });

    // Handle new message
    socket.on('send-message', async (data) => {
        try {
            const { user, message, room } = data;
            
            // Save message to database
            const newMessage = new Message({
                user,
                message,
                room
            });
            await newMessage.save();

            // Broadcast message to room
            io.to(room).emit('new-message', {
                user,
                message,
                room,
                timestamp: newMessage.timestamp
            });
        } catch (error) {
            console.error('Error saving message:', error);
            socket.emit('error', { message: 'Failed to send message' });
        }
    });

    // Handle typing indicator
    socket.on('typing', (data) => {
        socket.to(data.room).emit('user-typing', {
            user: data.user,
            isTyping: data.isTyping
        });
    });

    // Handle disconnect
    socket.on('disconnect', () => {
        console.log('User disconnected:', socket.id);
    });
});

// 4. REST API for message history
app.get('/api/messages/:room', async (req, res) => {
    try {
        const { room } = req.params;
        const { limit = 50, offset = 0 } = req.query;
        
        const messages = await Message.find({ room })
            .sort({ timestamp: -1 })
            .limit(parseInt(limit))
            .skip(parseInt(offset));
            
        res.json(messages.reverse());
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// Start server
const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
```

#### **Q3. How would you implement a file upload system with image processing?**

**Answer:** Complete file upload implementation:

```javascript
// 1. Server Setup with Multer
const express = require('express');
const multer = require('multer');
const sharp = require('sharp');
const path = require('path');
const fs = require('fs').promises;

const app = express();

// 2. Multer Configuration
const storage = multer.diskStorage({
    destination: (req, file, cb) => {
        cb(null, 'uploads/');
    },
    filename: (req, file, cb) => {
        const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
        cb(null, file.fieldname + '-' + uniqueSuffix + path.extname(file.originalname));
    }
});

const upload = multer({
    storage: storage,
    limits: {
        fileSize: 10 * 1024 * 1024 // 10MB limit
    },
    fileFilter: (req, file, cb) => {
        const allowedTypes = /jpeg|jpg|png|gif/;
        const extname = allowedTypes.test(path.extname(file.originalname).toLowerCase());
        const mimetype = allowedTypes.test(file.mimetype);

        if (mimetype && extname) {
            return cb(null, true);
        } else {
            cb(new Error('Only image files are allowed!'));
        }
    }
});

// 3. Image Processing Service
class ImageProcessor {
    static async processImage(inputPath, outputPath, options = {}) {
        const {
            width = 800,
            height = 600,
            quality = 80,
            format = 'jpeg'
        } = options;

        await sharp(inputPath)
            .resize(width, height, { fit: 'inside', withoutEnlargement: true })
            .jpeg({ quality })
            .toFile(outputPath);
    }

    static async createThumbnails(inputPath, baseName) {
        const sizes = [
            { name: 'thumb', width: 150, height: 150 },
            { name: 'medium', width: 300, height: 300 },
            { name: 'large', width: 800, height: 600 }
        ];

        const thumbnails = [];

        for (const size of sizes) {
            const outputPath = `uploads/thumbnails/${baseName}_${size.name}.jpg`;
            await this.processImage(inputPath, outputPath, {
                width: size.width,
                height: size.height
            });
            thumbnails.push({
                size: size.name,
                path: outputPath
            });
        }

        return thumbnails;
    }
}

// 4. Upload Route
app.post('/api/upload', upload.single('image'), async (req, res) => {
    try {
        if (!req.file) {
            return res.status(400).json({ error: 'No file uploaded' });
        }

        const file = req.file;
        const baseName = path.parse(file.filename).name;

        // Create thumbnails
        const thumbnails = await ImageProcessor.createThumbnails(file.path, baseName);

        // Process main image
        const processedPath = `uploads/processed/${baseName}_processed.jpg`;
        await ImageProcessor.processImage(file.path, processedPath);

        // Get file info
        const fileInfo = {
            originalName: file.originalname,
            filename: file.filename,
            size: file.size,
            mimetype: file.mimetype,
            path: file.path,
            processedPath: processedPath,
            thumbnails: thumbnails,
            uploadedAt: new Date()
        };

        res.json({
            message: 'File uploaded successfully',
            file: fileInfo
        });

    } catch (error) {
        console.error('Upload error:', error);
        res.status(500).json({ error: error.message });
    }
});

// 5. File Download Route
app.get('/api/download/:filename', (req, res) => {
    const filename = req.params.filename;
    const filePath = path.join(__dirname, 'uploads', filename);

    res.download(filePath, (err) => {
        if (err) {
            res.status(404).json({ error: 'File not found' });
        }
    });
});

// Start server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
```

#### **Q4. How would you implement a caching system with Redis?**

**Answer:** Complete caching implementation:

```javascript
// 1. Redis Setup
const redis = require('redis');
const client = redis.createClient({
    host: process.env.REDIS_HOST || 'localhost',
    port: process.env.REDIS_PORT || 6379,
    password: process.env.REDIS_PASSWORD
});

client.on('error', (err) => {
    console.error('Redis Client Error:', err);
});

client.on('connect', () => {
    console.log('Connected to Redis');
});

// 2. Cache Service
class CacheService {
    static async get(key) {
        try {
            const value = await client.get(key);
            return value ? JSON.parse(value) : null;
        } catch (error) {
            console.error('Cache get error:', error);
            return null;
        }
    }

    static async set(key, value, ttl = 3600) {
        try {
            await client.setex(key, ttl, JSON.stringify(value));
            return true;
        } catch (error) {
            console.error('Cache set error:', error);
            return false;
        }
    }

    static async del(key) {
        try {
            await client.del(key);
            return true;
        } catch (error) {
            console.error('Cache delete error:', error);
            return false;
        }
    }

    static async exists(key) {
        try {
            const result = await client.exists(key);
            return result === 1;
        } catch (error) {
            console.error('Cache exists error:', error);
            return false;
        }
    }

    static async flush() {
        try {
            await client.flushall();
            return true;
        } catch (error) {
            console.error('Cache flush error:', error);
            return false;
        }
    }
}

// 3. Cache Middleware
const cacheMiddleware = (ttl = 3600) => {
    return async (req, res, next) => {
        const key = `cache:${req.originalUrl}`;
        
        try {
            const cached = await CacheService.get(key);
            if (cached) {
                return res.json(cached);
            }
        } catch (error) {
            console.error('Cache middleware error:', error);
        }

        // Store original res.json
        const originalJson = res.json;
        res.json = function(data) {
            // Cache the response
            CacheService.set(key, data, ttl);
            originalJson.call(this, data);
        };

        next();
    };
};

// 4. Usage in Routes
app.get('/api/users', cacheMiddleware(1800), async (req, res) => {
    try {
        const users = await User.find().select('-password');
        res.json(users);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

app.get('/api/products/:id', cacheMiddleware(3600), async (req, res) => {
    try {
        const product = await Product.findById(req.params.id);
        if (!product) {
            return res.status(404).json({ error: 'Product not found' });
        }
        res.json(product);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// 5. Cache Invalidation
app.put('/api/products/:id', async (req, res) => {
    try {
        const product = await Product.findByIdAndUpdate(
            req.params.id,
            req.body,
            { new: true }
        );

        if (!product) {
            return res.status(404).json({ error: 'Product not found' });
        }

        // Invalidate cache
        await CacheService.del(`cache:/api/products/${req.params.id}`);
        await CacheService.del('cache:/api/products');

        res.json(product);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

#### **Q5. How would you implement a job queue system with Bull?**

**Answer:** Complete job queue implementation:

```javascript
// 1. Bull Queue Setup
const Queue = require('bull');
const Redis = require('ioredis');

// Create Redis connection
const redis = new Redis({
    host: process.env.REDIS_HOST || 'localhost',
    port: process.env.REDIS_PORT || 6379,
    password: process.env.REDIS_PASSWORD
});

// Create queues
const emailQueue = new Queue('email processing', { redis });
const imageQueue = new Queue('image processing', { redis });
const reportQueue = new Queue('report generation', { redis });

// 2. Email Job Processor
emailQueue.process('send-welcome-email', async (job) => {
    const { userEmail, userName } = job.data;
    
    try {
        // Simulate email sending
        console.log(`Sending welcome email to ${userEmail}`);
        
        // Add delay to simulate email processing
        await new Promise(resolve => setTimeout(resolve, 2000));
        
        console.log(`Welcome email sent to ${userEmail}`);
        
        return { success: true, email: userEmail };
    } catch (error) {
        throw new Error(`Failed to send email: ${error.message}`);
    }
});

// 3. Image Processing Job
imageQueue.process('resize-image', async (job) => {
    const { imagePath, sizes } = job.data;
    
    try {
        const sharp = require('sharp');
        const results = [];
        
        for (const size of sizes) {
            const outputPath = `processed/${size.name}_${Date.now()}.jpg`;
            
            await sharp(imagePath)
                .resize(size.width, size.height)
                .jpeg({ quality: 80 })
                .toFile(outputPath);
                
            results.push(outputPath);
            
            // Update job progress
            job.progress((results.length / sizes.length) * 100);
        }
        
        return { success: true, processedImages: results };
    } catch (error) {
        throw new Error(`Image processing failed: ${error.message}`);
    }
});

// 4. Report Generation Job
reportQueue.process('generate-report', async (job) => {
    const { userId, reportType, dateRange } = job.data;
    
    try {
        // Simulate report generation
        console.log(`Generating ${reportType} report for user ${userId}`);
        
        // Add delay to simulate processing
        await new Promise(resolve => setTimeout(resolve, 5000));
        
        const reportData = {
            userId,
            reportType,
            dateRange,
            generatedAt: new Date(),
            data: { /* report data */ }
        };
        
        return { success: true, report: reportData };
    } catch (error) {
        throw new Error(`Report generation failed: ${error.message}`);
    }
});

// 5. Job Management Routes
app.post('/api/send-email', async (req, res) => {
    try {
        const { userEmail, userName } = req.body;
        
        const job = await emailQueue.add('send-welcome-email', {
            userEmail,
            userName
        }, {
            attempts: 3,
            backoff: {
                type: 'exponential',
                delay: 2000
            }
        });
        
        res.json({
            message: 'Email job queued',
            jobId: job.id
        });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

app.post('/api/process-image', async (req, res) => {
    try {
        const { imagePath, sizes } = req.body;
        
        const job = await imageQueue.add('resize-image', {
            imagePath,
            sizes
        }, {
            attempts: 2,
            backoff: 'fixed'
        });
        
        res.json({
            message: 'Image processing job queued',
            jobId: job.id
        });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// 6. Job Status Monitoring
app.get('/api/job-status/:jobId', async (req, res) => {
    try {
        const { jobId } = req.params;
        
        // Check all queues for the job
        const queues = [emailQueue, imageQueue, reportQueue];
        
        for (const queue of queues) {
            const job = await queue.getJob(jobId);
            if (job) {
                return res.json({
                    jobId: job.id,
                    status: await job.getState(),
                    progress: job.progress(),
                    data: job.data,
                    result: job.returnvalue,
                    error: job.failedReason
                });
            }
        }
        
        res.status(404).json({ error: 'Job not found' });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// 7. Queue Monitoring
app.get('/api/queue-stats', async (req, res) => {
    try {
        const queues = [emailQueue, imageQueue, reportQueue];
        const stats = {};
        
        for (const queue of queues) {
            const queueName = queue.name;
            stats[queueName] = {
                waiting: await queue.getWaiting(),
                active: await queue.getActive(),
                completed: await queue.getCompleted(),
                failed: await queue.getFailed()
            };
        }
        
        res.json(stats);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

#### **Q6. How would you implement a comprehensive logging system?**

**Answer:** Complete logging implementation:

```javascript
// 1. Winston Logger Setup
const winston = require('winston');
const path = require('path');

// Create logger
const logger = winston.createLogger({
    level: 'info',
    format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.errors({ stack: true }),
        winston.format.json()
    ),
    defaultMeta: { service: 'api-server' },
    transports: [
        new winston.transports.File({ 
            filename: path.join('logs', 'error.log'), 
            level: 'error' 
        }),
        new winston.transports.File({ 
            filename: path.join('logs', 'combined.log') 
        })
    ]
});

// Add console transport in development
if (process.env.NODE_ENV !== 'production') {
    logger.add(new winston.transports.Console({
        format: winston.format.simple()
    }));
}

// 2. Custom Logger Middleware
const logMiddleware = (req, res, next) => {
    const startTime = Date.now();
    
    // Log request
    logger.info('Request started', {
        method: req.method,
        url: req.url,
        ip: req.ip,
        userAgent: req.get('User-Agent'),
        timestamp: new Date().toISOString()
    });
    
    // Override res.end to log response
    const originalEnd = res.end;
    res.end = function(chunk, encoding) {
        const duration = Date.now() - startTime;
        
        logger.info('Request completed', {
            method: req.method,
            url: req.url,
            statusCode: res.statusCode,
            duration: `${duration}ms`,
            timestamp: new Date().toISOString()
        });
        
        originalEnd.call(this, chunk, encoding);
    };
    
    next();
};

// 3. Error Logging Middleware
const errorLogger = (err, req, res, next) => {
    logger.error('Unhandled error', {
        error: err.message,
        stack: err.stack,
        method: req.method,
        url: req.url,
        ip: req.ip,
        timestamp: new Date().toISOString()
    });
    
    res.status(500).json({
        error: 'Internal server error',
        timestamp: new Date().toISOString()
    });
};

// 4. Database Logging
class DatabaseLogger {
    static logQuery(query, params, duration) {
        logger.debug('Database query executed', {
            query,
            params,
            duration: `${duration}ms`,
            timestamp: new Date().toISOString()
        });
    }
    
    static logError(error, query, params) {
        logger.error('Database error', {
            error: error.message,
            query,
            params,
            timestamp: new Date().toISOString()
        });
    }
}

// 5. Business Logic Logging
class BusinessLogger {
    static logUserAction(userId, action, details) {
        logger.info('User action', {
            userId,
            action,
            details,
            timestamp: new Date().toISOString()
        });
    }
    
    static logSecurityEvent(event, details) {
        logger.warn('Security event', {
            event,
            details,
            timestamp: new Date().toISOString()
        });
    }
    
    static logPerformanceMetric(metric, value, context) {
        logger.info('Performance metric', {
            metric,
            value,
            context,
            timestamp: new Date().toISOString()
        });
    }
}

// 6. Usage in Routes
app.use(logMiddleware);

app.post('/api/users', async (req, res) => {
    try {
        const user = await User.create(req.body);
        
        BusinessLogger.logUserAction(user._id, 'user_created', {
            email: user.email,
            role: user.role
        });
        
        res.status(201).json(user);
    } catch (error) {
        logger.error('User creation failed', {
            error: error.message,
            data: req.body,
            timestamp: new Date().toISOString()
        });
        
        res.status(500).json({ error: 'Failed to create user' });
    }
});

app.use(errorLogger);
```

#### **Q7. How would you implement a comprehensive testing suite?**

**Answer:** Complete testing implementation:

```javascript
// 1. Test Setup with Jest and Supertest
const request = require('supertest');
const app = require('../app');
const mongoose = require('mongoose');
const User = require('../models/User');

describe('API Tests', () => {
    beforeAll(async () => {
        // Connect to test database
        await mongoose.connect(process.env.TEST_DATABASE_URL);
    });

    afterAll(async () => {
        // Clean up database
        await mongoose.connection.dropDatabase();
        await mongoose.connection.close();
    });

    beforeEach(async () => {
        // Clean up before each test
        await User.deleteMany({});
    });

    // 2. Authentication Tests
    describe('Authentication', () => {
        test('should register a new user', async () => {
            const userData = {
                name: 'John Doe',
                email: 'john@example.com',
                password: 'password123'
            };

            const response = await request(app)
                .post('/api/register')
                .send(userData)
                .expect(201);

            expect(response.body).toHaveProperty('token');
            expect(response.body.user.email).toBe(userData.email);
        });

        test('should login with valid credentials', async () => {
            // Create user first
            const user = await User.create({
                name: 'John Doe',
                email: 'john@example.com',
                password: 'password123'
            });

            const response = await request(app)
                .post('/api/login')
                .send({
                    email: 'john@example.com',
                    password: 'password123'
                })
                .expect(200);

            expect(response.body).toHaveProperty('token');
            expect(response.body.user.email).toBe('john@example.com');
        });

        test('should reject invalid credentials', async () => {
            const response = await request(app)
                .post('/api/login')
                .send({
                    email: 'john@example.com',
                    password: 'wrongpassword'
                })
                .expect(401);

            expect(response.body).toHaveProperty('error');
        });
    });

    // 3. Protected Route Tests
    describe('Protected Routes', () => {
        let authToken;

        beforeEach(async () => {
            // Create user and get token
            const user = await User.create({
                name: 'John Doe',
                email: 'john@example.com',
                password: 'password123'
            });

            const response = await request(app)
                .post('/api/login')
                .send({
                    email: 'john@example.com',
                    password: 'password123'
                });

            authToken = response.body.token;
        });

        test('should access protected route with valid token', async () => {
            const response = await request(app)
                .get('/api/profile')
                .set('Authorization', `Bearer ${authToken}`)
                .expect(200);

            expect(response.body.email).toBe('john@example.com');
        });

        test('should reject access without token', async () => {
            await request(app)
                .get('/api/profile')
                .expect(401);
        });
    });

    // 4. File Upload Tests
    describe('File Upload', () => {
        test('should upload image file', async () => {
            const response = await request(app)
                .post('/api/upload')
                .attach('image', 'test/fixtures/test-image.jpg')
                .expect(200);

            expect(response.body).toHaveProperty('file');
            expect(response.body.file).toHaveProperty('thumbnails');
        });

        test('should reject non-image files', async () => {
            const response = await request(app)
                .post('/api/upload')
                .attach('image', 'test/fixtures/test-document.pdf')
                .expect(400);

            expect(response.body).toHaveProperty('error');
        });
    });

    // 5. Performance Tests
    describe('Performance', () => {
        test('should handle concurrent requests', async () => {
            const requests = Array(10).fill().map(() =>
                request(app).get('/api/users')
            );

            const responses = await Promise.all(requests);
            
            responses.forEach(response => {
                expect(response.status).toBe(200);
            });
        });
    });
});

// 6. Integration Tests
describe('Integration Tests', () => {
    test('should complete user registration flow', async () => {
        // Register user
        const registerResponse = await request(app)
            .post('/api/register')
            .send({
                name: 'John Doe',
                email: 'john@example.com',
                password: 'password123'
            })
            .expect(201);

        const token = registerResponse.body.token;

        // Access protected route
        const profileResponse = await request(app)
            .get('/api/profile')
            .set('Authorization', `Bearer ${token}`)
            .expect(200);

        expect(profileResponse.body.email).toBe('john@example.com');
    });
});
```

#### **Q8. How would you implement a microservices architecture?**

**Answer:** Complete microservices implementation:

```javascript
// 1. API Gateway
const express = require('express');
const httpProxy = require('http-proxy-middleware');
const jwt = require('jsonwebtoken');

const app = express();

// Service registry
const services = {
    'user-service': 'http://localhost:3001',
    'product-service': 'http://localhost:3002',
    'order-service': 'http://localhost:3003'
};

// Authentication middleware
const authenticateToken = (req, res, next) => {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1];

    if (!token) {
        return res.status(401).json({ error: 'Access token required' });
    }

    jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
        if (err) {
            return res.status(403).json({ error: 'Invalid token' });
        }
        req.user = user;
        next();
    });
};

// Proxy middleware
const createProxy = (target) => {
    return httpProxy({
        target,
        changeOrigin: true,
        pathRewrite: {
            '^/api': ''
        }
    });
};

// Routes
app.use('/api/users', authenticateToken, createProxy(services['user-service']));
app.use('/api/products', createProxy(services['product-service']));
app.use('/api/orders', authenticateToken, createProxy(services['order-service']));

// Start gateway
app.listen(3000, () => {
    console.log('API Gateway running on port 3000');
});

// 2. User Service
const userApp = express();
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
    name: String,
    email: String,
    password: String
});

const User = mongoose.model('User', userSchema);

userApp.get('/users', async (req, res) => {
    const users = await User.find();
    res.json(users);
});

userApp.get('/users/:id', async (req, res) => {
    const user = await User.findById(req.params.id);
    res.json(user);
});

userApp.listen(3001, () => {
    console.log('User service running on port 3001');
});

// 3. Product Service
const productApp = express();

const productSchema = new mongoose.Schema({
    name: String,
    price: Number,
    description: String
});

const Product = mongoose.model('Product', productSchema);

productApp.get('/products', async (req, res) => {
    const products = await Product.find();
    res.json(products);
});

productApp.get('/products/:id', async (req, res) => {
    const product = await Product.findById(req.params.id);
    res.json(product);
});

productApp.listen(3002, () => {
    console.log('Product service running on port 3002');
});

// 4. Order Service
const orderApp = express();

const orderSchema = new mongoose.Schema({
    userId: String,
    productId: String,
    quantity: Number,
    status: String
});

const Order = mongoose.model('Order', orderSchema);

orderApp.get('/orders', async (req, res) => {
    const orders = await Order.find();
    res.json(orders);
});

orderApp.post('/orders', async (req, res) => {
    const order = new Order(req.body);
    await order.save();
    res.json(order);
});

orderApp.listen(3003, () => {
    console.log('Order service running on port 3003');
});
```

#### **Q9. How would you implement a comprehensive error handling system?**

**Answer:** Complete error handling implementation:

```javascript
// 1. Custom Error Classes
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

class AuthenticationError extends AppError {
    constructor(message = 'Authentication failed') {
        super(message, 401);
        this.name = 'AuthenticationError';
    }
}

class AuthorizationError extends AppError {
    constructor(message = 'Access denied') {
        super(message, 403);
        this.name = 'AuthorizationError';
    }
}

class NotFoundError extends AppError {
    constructor(message = 'Resource not found') {
        super(message, 404);
        this.name = 'NotFoundError';
    }
}

// 2. Error Handler Middleware
const errorHandler = (err, req, res, next) => {
    let error = { ...err };
    error.message = err.message;

    // Log error
    console.error(err);

    // Mongoose bad ObjectId
    if (err.name === 'CastError') {
        const message = 'Resource not found';
        error = new NotFoundError(message);
    }

    // Mongoose duplicate key
    if (err.code === 11000) {
        const message = 'Duplicate field value entered';
        error = new ValidationError(message);
    }

    // Mongoose validation error
    if (err.name === 'ValidationError') {
        const message = Object.values(err.errors).map(val => val.message).join(', ');
        error = new ValidationError(message);
    }

    // JWT errors
    if (err.name === 'JsonWebTokenError') {
        const message = 'Invalid token';
        error = new AuthenticationError(message);
    }

    if (err.name === 'TokenExpiredError') {
        const message = 'Token expired';
        error = new AuthenticationError(message);
    }

    res.status(error.statusCode || 500).json({
        success: false,
        error: error.message || 'Server Error',
        ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
    });
};

// 3. Async Error Handler
const asyncHandler = (fn) => (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
};

// 4. Usage in Routes
app.get('/api/users/:id', asyncHandler(async (req, res, next) => {
    const user = await User.findById(req.params.id);
    
    if (!user) {
        return next(new NotFoundError('User not found'));
    }
    
    res.json(user);
}));

app.post('/api/users', asyncHandler(async (req, res, next) => {
    const { name, email, password } = req.body;
    
    if (!name || !email || !password) {
        return next(new ValidationError('Name, email, and password are required'));
    }
    
    const existingUser = await User.findOne({ email });
    if (existingUser) {
        return next(new ValidationError('User with this email already exists'));
    }
    
    const user = await User.create({ name, email, password });
    res.status(201).json(user);
}));

// 5. Global Error Handler
process.on('uncaughtException', (err) => {
    console.error('UNCAUGHT EXCEPTION! Shutting down...');
    console.error(err.name, err.message);
    process.exit(1);
});

process.on('unhandledRejection', (err) => {
    console.error('UNHANDLED REJECTION! Shutting down...');
    console.error(err.name, err.message);
    server.close(() => {
        process.exit(1);
    });
});

// 6. Error Response Helper
const sendErrorResponse = (res, statusCode, message, error = null) => {
    res.status(statusCode).json({
        success: false,
        error: message,
        ...(process.env.NODE_ENV === 'development' && error && { details: error })
    });
};

// 7. Validation Middleware
const validateRequest = (schema) => {
    return (req, res, next) => {
        const { error } = schema.validate(req.body);
        if (error) {
            return next(new ValidationError(error.details[0].message));
        }
        next();
    };
};

// Use error handler
app.use(errorHandler);
```

#### **Q10. How would you implement a comprehensive monitoring system?**

**Answer:** Complete monitoring implementation:

```javascript
// 1. Prometheus Metrics Setup
const promClient = require('prom-client');

// Create a Registry
const register = new promClient.Registry();

// Add default metrics
promClient.collectDefaultMetrics({ register });

// Custom metrics
const httpRequestDuration = new promClient.Histogram({
    name: 'http_request_duration_seconds',
    help: 'Duration of HTTP requests in seconds',
    labelNames: ['method', 'route', 'status_code'],
    buckets: [0.1, 0.3, 0.5, 0.7, 1, 3, 5, 7, 10]
});

const httpRequestTotal = new promClient.Counter({
    name: 'http_requests_total',
    help: 'Total number of HTTP requests',
    labelNames: ['method', 'route', 'status_code']
});

const activeConnections = new promClient.Gauge({
    name: 'active_connections',
    help: 'Number of active connections'
});

// Register metrics
register.registerMetric(httpRequestDuration);
register.registerMetric(httpRequestTotal);
register.registerMetric(activeConnections);

// 2. Metrics Middleware
const metricsMiddleware = (req, res, next) => {
    const start = Date.now();
    
    res.on('finish', () => {
        const duration = (Date.now() - start) / 1000;
        const labels = {
            method: req.method,
            route: req.route?.path || req.path,
            status_code: res.statusCode
        };
        
        httpRequestDuration.observe(labels, duration);
        httpRequestTotal.inc(labels);
    });
    
    next();
};

// 3. Health Check Endpoint
app.get('/health', (req, res) => {
    const health = {
        status: 'OK',
        timestamp: new Date().toISOString(),
        uptime: process.uptime(),
        memory: process.memoryUsage(),
        version: process.env.npm_package_version
    };
    
    res.json(health);
});

// 4. Metrics Endpoint
app.get('/metrics', async (req, res) => {
    res.set('Content-Type', register.contentType);
    res.end(await register.metrics());
});

// 5. Application Monitoring
class ApplicationMonitor {
    static start() {
        // Monitor memory usage
        setInterval(() => {
            const memUsage = process.memoryUsage();
            console.log('Memory Usage:', {
                rss: `${Math.round(memUsage.rss / 1024 / 1024)} MB`,
                heapTotal: `${Math.round(memUsage.heapTotal / 1024 / 1024)} MB`,
                heapUsed: `${Math.round(memUsage.heapUsed / 1024 / 1024)} MB`,
                external: `${Math.round(memUsage.external / 1024 / 1024)} MB`
            });
        }, 30000); // Every 30 seconds

        // Monitor CPU usage
        const startUsage = process.cpuUsage();
        setInterval(() => {
            const currentUsage = process.cpuUsage(startUsage);
            const cpuPercent = (currentUsage.user + currentUsage.system) / 1000000;
            console.log('CPU Usage:', `${cpuPercent}%`);
        }, 30000);

        // Monitor event loop lag
        setInterval(() => {
            const start = process.hrtime();
            setImmediate(() => {
                const delta = process.hrtime(start);
                const nanosec = delta[0] * 1e9 + delta[1];
                const millisec = nanosec / 1e6;
                console.log('Event Loop Lag:', `${millisec}ms`);
            });
        }, 30000);
    }
}

// 6. Database Monitoring
class DatabaseMonitor {
    static monitorConnection() {
        mongoose.connection.on('connected', () => {
            console.log('Database connected');
        });

        mongoose.connection.on('error', (err) => {
            console.error('Database connection error:', err);
        });

        mongoose.connection.on('disconnected', () => {
            console.log('Database disconnected');
        });
    }
}

// 7. Error Monitoring
class ErrorMonitor {
    static logError(error, context = {}) {
        console.error('Application Error:', {
            message: error.message,
            stack: error.stack,
            context,
            timestamp: new Date().toISOString()
        });
    }

    static logPerformance(operation, duration, context = {}) {
        console.log('Performance Log:', {
            operation,
            duration: `${duration}ms`,
            context,
            timestamp: new Date().toISOString()
        });
    }
}

// Start monitoring
ApplicationMonitor.start();
DatabaseMonitor.monitorConnection();

// Use middleware
app.use(metricsMiddleware);
```

---

## 🎯 SUMMARY

**Total Node.js/Express.js Practical Questions: 10+ with complete implementations covering:**

✅ **RESTful API with Authentication**  
✅ **Real-time Chat with Socket.io**  
✅ **File Upload & Image Processing**  
✅ **Caching with Redis**  
✅ **Job Queue with Bull**  
✅ **Comprehensive Logging**  
✅ **Testing Suite**  
✅ **Microservices Architecture**  
✅ **Error Handling System**  
✅ **Monitoring & Metrics**

Each question includes:
- **Complete working code**
- **Real-world examples**
- **Best practices**
- **Error handling**
- **Performance optimization**
- **Security considerations**

**Perfect for senior Node.js developer interviews! 🚀**