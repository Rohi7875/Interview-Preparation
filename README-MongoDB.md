# MongoDB Interview Questions & Answers
### For 7+ Years Experienced Developers

---

## Table of Contents
1. [MongoDB Basics](#mongodb-basics)
2. [CRUD Operations](#crud-operations)
3. [Aggregation Framework](#aggregation-framework)
4. [Indexing & Performance](#indexing--performance)
5. [Schema Design](#schema-design)
6. [Replication & Sharding](#replication--sharding)
7. [Transactions](#transactions)
8. [Mongoose ODM](#mongoose-odm)
9. [Security](#security)
10. [Optimization & Best Practices](#optimization--best-practices)

---

## MongoDB Basics

### Q1. What is MongoDB and when should you use it?
**Answer:** MongoDB is a NoSQL document-oriented database that stores data in flexible, JSON-like documents (BSON format).

**Use MongoDB when:**
- Schema flexibility is needed
- Horizontal scaling is required
- Working with hierarchical data
- Rapid prototyping
- Real-time analytics
- Handling large volumes of data

**Don't use MongoDB when:**
- Complex transactions required (though supported now)
- Complex joins are frequent
- Strict data consistency is critical
- Fixed schema is better

**Real-time Example:**
```javascript
// MongoDB Document Structure (BSON)
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "username": "john_doe",
  "email": "john@example.com",
  "profile": {
    "firstName": "John",
    "lastName": "Doe",
    "age": 30,
    "address": {
      "street": "123 Main St",
      "city": "New York",
      "country": "USA",
      "zipCode": "10001"
    }
  },
  "interests": ["coding", "music", "sports"],
  "orders": [
    {
      "orderId": "ORD001",
      "date": ISODate("2025-01-15T10:00:00Z"),
      "total": 199.99,
      "items": [
        { "productId": "PROD001", "name": "Laptop", "price": 999.99, "quantity": 1 }
      ]
    }
  ],
  "lastLogin": ISODate("2025-10-12T08:30:00Z"),
  "createdAt": ISODate("2024-01-01T00:00:00Z"),
  "updatedAt": ISODate("2025-10-12T08:30:00Z")
}

// Connect to MongoDB
const { MongoClient } = require('mongodb');

const uri = 'mongodb://localhost:27017';
const client = new MongoClient(uri);

async function connectDatabase() {
    try {
        await client.connect();
        console.log('Connected to MongoDB');
        
        const db = client.db('myapp');
        return db;
    } catch (error) {
        console.error('Connection error:', error);
        throw error;
    }
}

// Database operations wrapper
class DatabaseService {
    constructor() {
        this.client = null;
        this.db = null;
    }
    
    async connect(uri, dbName) {
        this.client = new MongoClient(uri, {
            useNewUrlParser: true,
            useUnifiedTopology: true,
            maxPoolSize: 10,
            serverSelectionTimeoutMS: 5000,
            socketTimeoutMS: 45000,
        });
        
        await this.client.connect();
        this.db = this.client.db(dbName);
        console.log(`Connected to database: ${dbName}`);
    }
    
    getCollection(name) {
        return this.db.collection(name);
    }
    
    async close() {
        if (this.client) {
            await this.client.close();
            console.log('Database connection closed');
        }
    }
}

module.exports = new DatabaseService();
```

### Q2. Explain MongoDB data types and BSON format
**Answer:** BSON (Binary JSON) is a binary representation of JSON-like documents. MongoDB supports various data types:

**Data Types:**
- String
- Number (Int32, Int64, Double, Decimal128)
- Boolean
- Array
- Object/Document
- ObjectId
- Date
- Null
- Binary Data
- Regular Expression
- JavaScript Code
- Timestamp

**Real-time Example:**
```javascript
// MongoDB Data Types Example
const mongoose = require('mongoose');

const productSchema = new mongoose.Schema({
    // String
    name: { type: String, required: true, trim: true },
    sku: { type: String, unique: true, uppercase: true },
    
    // Numbers
    price: { type: Number, required: true, min: 0 },
    stock: { type: Number, default: 0 },
    rating: { type: Number, min: 0, max: 5 },
    
    // Decimal128 (for precise decimal calculations)
    priceExact: { type: mongoose.Schema.Types.Decimal128, required: true },
    
    // Boolean
    isActive: { type: Boolean, default: true },
    isFeatured: { type: Boolean, default: false },
    
    // Date
    launchDate: { type: Date },
    lastUpdated: { type: Date, default: Date.now },
    
    // ObjectId (Reference to another document)
    categoryId: { type: mongoose.Schema.Types.ObjectId, ref: 'Category' },
    supplierId: { type: mongoose.Schema.Types.ObjectId, ref: 'Supplier' },
    
    // Array of strings
    tags: [{ type: String, lowercase: true }],
    colors: [String],
    
    // Array of subdocuments
    images: [{
        url: String,
        alt: String,
        isPrimary: Boolean,
        uploadedAt: { type: Date, default: Date.now }
    }],
    
    // Nested object
    dimensions: {
        length: Number,
        width: Number,
        height: Number,
        unit: { type: String, enum: ['cm', 'in'], default: 'cm' }
    },
    
    // Mixed type (flexible schema)
    metadata: mongoose.Schema.Types.Mixed,
    
    // Array of ObjectIds
    relatedProducts: [{ type: mongoose.Schema.Types.ObjectId, ref: 'Product' }],
    
    // Buffer (for binary data)
    thumbnail: Buffer,
    
    // Map (key-value pairs)
    specifications: { type: Map, of: String },
    
    // Enum
    status: {
        type: String,
        enum: ['draft', 'published', 'archived'],
        default: 'draft'
    }
}, {
    timestamps: true, // Adds createdAt and updatedAt automatically
    collection: 'products'
});

const Product = mongoose.model('Product', productSchema);

// Usage example
async function createProduct() {
    const product = new Product({
        name: 'Premium Wireless Headphones',
        sku: 'WH-1000XM4',
        price: 349.99,
        priceExact: mongoose.Types.Decimal128.fromString('349.99'),
        stock: 50,
        rating: 4.8,
        isActive: true,
        isFeatured: true,
        launchDate: new Date('2025-01-15'),
        tags: ['electronics', 'audio', 'wireless', 'noise-canceling'],
        colors: ['Black', 'Silver', 'Blue'],
        images: [
            {
                url: '/images/headphones-1.jpg',
                alt: 'Front view',
                isPrimary: true
            },
            {
                url: '/images/headphones-2.jpg',
                alt: 'Side view',
                isPrimary: false
            }
        ],
        dimensions: {
            length: 20,
            width: 18,
            height: 8,
            unit: 'cm'
        },
        metadata: {
            manufacturer: 'Sony',
            warranty: '2 years',
            batteryLife: '30 hours'
        },
        specifications: new Map([
            ['Driver Size', '40mm'],
            ['Frequency Response', '4Hz-40kHz'],
            ['Bluetooth Version', '5.0'],
            ['Weight', '250g']
        ]),
        status: 'published'
    });
    
    await product.save();
    return product;
}
```

### Q3. What are the latest features in MongoDB 6.0, 7.0?
**Answer:**

**MongoDB 6.0 (2022):**
- Time Series collections improvements
- Encrypted collections (Queryable Encryption)
- Change streams optimization
- Improved aggregation performance

**MongoDB 7.0 (2023):**
- Queryable Encryption GA
- Compound wildcard indexes
- Improved $lookup performance
- Better time series support
- Atlas Vector Search (AI/ML)

**Real-time Example:**
```javascript
// MongoDB 7.0 Features

// 1. Compound Wildcard Indexes (MongoDB 7.0+)
db.products.createIndex({
    "specifications.$**": 1,
    "price": 1
}, {
    name: "compound_wildcard_index"
});

// Query using compound wildcard index
db.products.find({
    "specifications.color": "blue",
    "price": { $lt: 1000 }
});

// 2. Queryable Encryption (MongoDB 6.0+, GA in 7.0)
const { ClientEncryption } = require('mongodb');

const encryptionOptions = {
    keyVaultNamespace: 'encryption.__keyVault',
    kmsProviders: {
        local: {
            key: Buffer.from(process.env.MASTER_KEY, 'base64')
        }
    }
};

const clientEncryption = new ClientEncryption(client, encryptionOptions);

// Encrypt sensitive data
const encryptedSSN = await clientEncryption.encrypt('123-45-6789', {
    algorithm: 'AEAD_AES_256_CBC_HMAC_SHA_512-Deterministic',
    keyId: dataKeyId
});

// Insert encrypted data
await db.collection('users').insertOne({
    name: 'John Doe',
    ssn: encryptedSSN, // Encrypted field
    email: 'john@example.com'
});

// 3. Time Series Collections (Improved in 6.0+)
db.createCollection('weatherData', {
    timeseries: {
        timeField: 'timestamp',
        metaField: 'metadata',
        granularity: 'hours'
    },
    expireAfterSeconds: 2592000 // Expire after 30 days
});

// Insert time series data
db.weatherData.insertMany([
    {
        timestamp: new Date(),
        metadata: { sensorId: 'sensor-001', location: 'New York' },
        temperature: 25.5,
        humidity: 65,
        pressure: 1013
    },
    {
        timestamp: new Date(),
        metadata: { sensorId: 'sensor-002', location: 'Los Angeles' },
        temperature: 28.3,
        humidity: 55,
        pressure: 1015
    }
]);

// Query time series data
db.weatherData.aggregate([
    {
        $match: {
            'metadata.location': 'New York',
            timestamp: {
                $gte: new Date(Date.now() - 24 * 60 * 60 * 1000)
            }
        }
    },
    {
        $group: {
            _id: {
                $dateTrunc: {
                    date: '$timestamp',
                    unit: 'hour'
                }
            },
            avgTemp: { $avg: '$temperature' },
            avgHumidity: { $avg: '$humidity' }
        }
    },
    { $sort: { _id: 1 } }
]);

// 4. Atlas Vector Search (MongoDB 7.0+ with Atlas)
// Create vector search index
db.movies.createSearchIndex({
    name: 'vector_index',
    type: 'vectorSearch',
    definition: {
        fields: [{
            type: 'vector',
            path: 'plot_embedding',
            numDimensions: 1536,
            similarity: 'cosine'
        }]
    }
});

// Vector search query (for AI/ML applications)
db.movies.aggregate([
    {
        $search: {
            index: 'vector_index',
            vectorSearch: {
                queryVector: [0.1, 0.2, 0.3, ...], // Embedding vector
                path: 'plot_embedding',
                numCandidates: 100,
                limit: 10
            }
        }
    },
    {
        $project: {
            title: 1,
            plot: 1,
            score: { $meta: 'searchScore' }
        }
    }
]);
```

---

## CRUD Operations

### Q4. Explain CRUD operations in MongoDB with complex examples
**Answer:** CRUD operations are Create, Read, Update, and Delete operations.

**Real-time Example:**
```javascript
const mongoose = require('mongoose');

// User Model
const userSchema = new mongoose.Schema({
    username: { type: String, required: true, unique: true },
    email: { type: String, required: true, unique: true },
    password: { type: String, required: true },
    profile: {
        firstName: String,
        lastName: String,
        avatar: String,
        bio: String
    },
    role: { type: String, enum: ['user', 'admin', 'moderator'], default: 'user' },
    isActive: { type: Boolean, default: true },
    lastLogin: Date,
    loginCount: { type: Number, default: 0 },
    preferences: {
        theme: { type: String, default: 'light' },
        notifications: { type: Boolean, default: true },
        language: { type: String, default: 'en' }
    },
    tags: [String],
    metadata: mongoose.Schema.Types.Mixed
}, { timestamps: true });

const User = mongoose.model('User', userSchema);

// ==== CREATE Operations ====

class UserService {
    // 1. Create single user
    async createUser(userData) {
        try {
            const user = new User(userData);
            await user.save();
            return user;
        } catch (error) {
            if (error.code === 11000) {
                throw new Error('User already exists');
            }
            throw error;
        }
    }
    
    // 2. Create multiple users (bulk insert)
    async createManyUsers(usersData) {
        try {
            const users = await User.insertMany(usersData, {
                ordered: false // Continue on error
            });
            return users;
        } catch (error) {
            console.error('Some users failed to insert:', error);
            return error.insertedDocs || [];
        }
    }
    
    // 3. Create with validation
    async createUserWithValidation(userData) {
        // Custom validation
        if (!userData.email.includes('@')) {
            throw new Error('Invalid email format');
        }
        
        // Check if user exists
        const existingUser = await User.findOne({
            $or: [
                { email: userData.email },
                { username: userData.username }
            ]
        });
        
        if (existingUser) {
            throw new Error('User already exists');
        }
        
        const user = await User.create(userData);
        return user;
    }
    
    // ==== READ Operations ====
    
    // 1. Find by ID
    async getUserById(id) {
        return await User.findById(id).select('-password');
    }
    
    // 2. Find one with conditions
    async getUserByEmail(email) {
        return await User.findOne({ email }).select('-password');
    }
    
    // 3. Find multiple with filters
    async getUsers(filters = {}, options = {}) {
        const {
            page = 1,
            limit = 10,
            sortBy = 'createdAt',
            sortOrder = 'desc',
            search = ''
        } = options;
        
        const query = { ...filters };
        
        // Search in multiple fields
        if (search) {
            query.$or = [
                { username: new RegExp(search, 'i') },
                { email: new RegExp(search, 'i') },
                { 'profile.firstName': new RegExp(search, 'i') },
                { 'profile.lastName': new RegExp(search, 'i') }
            ];
        }
        
        const skip = (page - 1) * limit;
        const sort = { [sortBy]: sortOrder === 'desc' ? -1 : 1 };
        
        const [users, total] = await Promise.all([
            User.find(query)
                .select('-password')
                .sort(sort)
                .skip(skip)
                .limit(limit)
                .lean(), // Returns plain JavaScript objects
            User.countDocuments(query)
        ]);
        
        return {
            users,
            pagination: {
                total,
                page,
                pages: Math.ceil(total / limit),
                limit
            }
        };
    }
    
    // 4. Complex queries with aggregation
    async getUserStatistics() {
        return await User.aggregate([
            {
                $facet: {
                    // Total users by role
                    byRole: [
                        {
                            $group: {
                                _id: '$role',
                                count: { $sum: 1 }
                            }
                        }
                    ],
                    // Active vs inactive users
                    byStatus: [
                        {
                            $group: {
                                _id: '$isActive',
                                count: { $sum: 1 }
                            }
                        }
                    ],
                    // Users registered per month
                    registrationTrend: [
                        {
                            $group: {
                                _id: {
                                    year: { $year: '$createdAt' },
                                    month: { $month: '$createdAt' }
                                },
                                count: { $sum: 1 }
                            }
                        },
                        { $sort: { '_id.year': -1, '_id.month': -1 } },
                        { $limit: 12 }
                    ],
                    // Most active users (by login count)
                    topUsers: [
                        { $sort: { loginCount: -1 } },
                        { $limit: 10 },
                        {
                            $project: {
                                username: 1,
                                email: 1,
                                loginCount: 1,
                                lastLogin: 1
                            }
                        }
                    ]
                }
            }
        ]);
    }
    
    // ==== UPDATE Operations ====
    
    // 1. Update by ID
    async updateUser(id, updates) {
        const user = await User.findByIdAndUpdate(
            id,
            { $set: updates },
            { new: true, runValidators: true }
        ).select('-password');
        
        if (!user) {
            throw new Error('User not found');
        }
        
        return user;
    }
    
    // 2. Update with atomic operations
    async updateUserLogin(id) {
        return await User.findByIdAndUpdate(
            id,
            {
                $set: { lastLogin: new Date() },
                $inc: { loginCount: 1 } // Atomic increment
            },
            { new: true }
        );
    }
    
    // 3. Update multiple documents
    async deactivateInactiveUsers(days = 90) {
        const cutoffDate = new Date();
        cutoffDate.setDate(cutoffDate.getDate() - days);
        
        const result = await User.updateMany(
            {
                lastLogin: { $lt: cutoffDate },
                isActive: true
            },
            {
                $set: { isActive: false }
            }
        );
        
        return {
            matched: result.matchedCount,
            modified: result.modifiedCount
        };
    }
    
    // 4. Upsert (update or insert)
    async upsertUser(email, userData) {
        return await User.findOneAndUpdate(
            { email },
            { $set: userData },
            {
                new: true,
                upsert: true, // Create if doesn't exist
                runValidators: true
            }
        );
    }
    
    // 5. Update array fields
    async addTag(userId, tag) {
        return await User.findByIdAndUpdate(
            userId,
            {
                $addToSet: { tags: tag } // Add only if not exists
            },
            { new: true }
        );
    }
    
    async removeTags(userId, tags) {
        return await User.findByIdAndUpdate(
            userId,
            {
                $pull: { tags: { $in: tags } } // Remove multiple tags
            },
            { new: true }
        );
    }
    
    // ==== DELETE Operations ====
    
    // 1. Delete by ID
    async deleteUser(id) {
        const user = await User.findByIdAndDelete(id);
        
        if (!user) {
            throw new Error('User not found');
        }
        
        // Cleanup related data
        await this.cleanupUserData(id);
        
        return { success: true, deletedUser: user };
    }
    
    // 2. Soft delete
    async softDeleteUser(id) {
        return await User.findByIdAndUpdate(
            id,
            {
                $set: {
                    isActive: false,
                    deletedAt: new Date()
                }
            },
            { new: true }
        );
    }
    
    // 3. Delete multiple documents
    async deleteInactiveUsers(days = 180) {
        const cutoffDate = new Date();
        cutoffDate.setDate(cutoffDate.getDate() - days);
        
        const result = await User.deleteMany({
            lastLogin: { $lt: cutoffDate },
            isActive: false
        });
        
        return {
            deletedCount: result.deletedCount
        };
    }
    
    // 4. Delete with conditions
    async deleteUsersWithoutLogin() {
        return await User.deleteMany({
            lastLogin: { $exists: false },
            createdAt: {
                $lt: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000) // 30 days old
            }
        });
    }
    
    // Helper methods
    async cleanupUserData(userId) {
        // Delete user's posts, comments, etc.
        // This is where you'd clean up related collections
        console.log(`Cleaning up data for user: ${userId}`);
    }
}

module.exports = new UserService();
```

This covers the main MongoDB concepts. Would you like me to continue with more sections including Aggregation Framework, Indexing, and advanced topics?
