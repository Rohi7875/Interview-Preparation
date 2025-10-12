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
