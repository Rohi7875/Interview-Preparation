# Node.js Array Methods - Complete Guide
### Real-World Examples for Interview Preparation

---

## Table of Contents
1. [Array Creation & Manipulation](#array-creation--manipulation)
2. [Array Iteration Methods](#array-iteration-methods)
3. [Array Search & Filter](#array-search--filter)
4. [Array Transformation](#array-transformation)
5. [Array Reduction](#array-reduction)
6. [Array Sorting](#array-sorting)
7. [Array Testing](#array-testing)
8. [Array Modification](#array-modification)
9. [ES6+ Modern Array Methods](#es6-modern-array-methods)
10. [Real-World Use Cases](#real-world-use-cases)

---

## Array Creation & Manipulation

### Q1. Array Creation Methods
**Answer:** Multiple ways to create arrays in JavaScript/Node.js.

**Real-time Example:**
```javascript
// E-commerce Product Management System

class ProductManager {
    constructor() {
        this.products = [];
    }
    
    // Method 1: Array literal
    initializeWithLiteral() {
        this.products = [
            { id: 1, name: 'Laptop', price: 999.99 },
            { id: 2, name: 'Mouse', price: 29.99 }
        ];
    }
    
    // Method 2: Array constructor
    initializeWithConstructor() {
        this.products = new Array(
            { id: 1, name: 'Laptop', price: 999.99 },
            { id: 2, name: 'Mouse', price: 29.99 }
        );
    }
    
    // Method 3: Array.of() - Creates array with elements
    createWithOf(...items) {
        return Array.of(...items);
    }
    
    // Method 4: Array.from() - Creates array from iterable
    createFromIterable(iterable) {
        return Array.from(iterable);
    }
    
    // Method 5: Array.from() with mapping function
    createProductRange(count) {
        return Array.from({ length: count }, (_, index) => ({
            id: index + 1,
            name: `Product ${index + 1}`,
            price: (index + 1) * 10
        }));
    }
    
    // Method 6: Spread operator
    cloneArray() {
        return [...this.products];
    }
    
    // Real-world example: Initialize product catalog
    initializeCatalog() {
        // Create categories array
        const categories = Array.from(
            new Set(['Electronics', 'Clothing', 'Books', 'Food'])
        );
        
        // Create products with Array.from and mapping
        const products = Array.from({ length: 10 }, (_, i) => ({
            id: i + 1,
            name: `Product ${i + 1}`,
            price: Math.random() * 1000,
            category: categories[Math.floor(Math.random() * categories.length)],
            stock: Math.floor(Math.random() * 100),
            rating: (Math.random() * 5).toFixed(1)
        }));
        
        this.products = products;
        return products;
    }
}

// Usage
const manager = new ProductManager();
const catalog = manager.initializeCatalog();
console.log('Product Catalog:', catalog);

// Create array from string
const letters = Array.from('HELLO');
console.log('Letters:', letters); // ['H', 'E', 'L', 'L', 'O']

// Create array with specific length and default values
const emptySlots = Array.from({ length: 5 }, () => null);
console.log('Empty slots:', emptySlots); // [null, null, null, null, null]
```

---

## Array Iteration Methods

### Q2. forEach vs map vs for...of
**Answer:** Different iteration methods with different purposes and return values.

**Real-time Example:**
```javascript
// User Processing System

class UserProcessor {
    constructor(users) {
        this.users = users;
    }
    
    // Method 1: forEach - No return value, side effects
    processWithForEach() {
        const results = [];
        
        this.users.forEach((user, index) => {
            console.log(`Processing user ${index + 1}: ${user.name}`);
            
            results.push({
                id: user.id,
                name: user.name.toUpperCase(),
                email: user.email.toLowerCase(),
                status: user.active ? 'Active' : 'Inactive'
            });
        });
        
        return results;
    }
    
    // Method 2: map - Returns new array
    processWithMap() {
        return this.users.map((user, index) => ({
            id: user.id,
            name: user.name.toUpperCase(),
            email: user.email.toLowerCase(),
            status: user.active ? 'Active' : 'Inactive',
            processedAt: new Date().toISOString()
        }));
    }
    
    // Method 3: for...of - More flexible, can use break/continue
    processWithForOf() {
        const results = [];
        
        for (const user of this.users) {
            // Can use break/continue
            if (!user.active) continue;
            
            results.push({
                id: user.id,
                name: user.name,
                email: user.email
            });
        }
        
        return results;
    }
    
    // Real-world example: Process orders with different methods
    async processOrders(orders) {
        const processedOrders = [];
        let totalRevenue = 0;
        
        // Use forEach for side effects
        orders.forEach(order => {
            totalRevenue += order.total;
            
            // Send email notification (async, fire and forget)
            this.sendOrderConfirmation(order.email).catch(console.error);
        });
        
        // Use map to transform data
        const orderSummaries = orders.map(order => ({
            orderId: order.id,
            customerName: order.customerName,
            items: order.items.length,
            total: order.total,
            date: new Date(order.date).toLocaleDateString()
        }));
        
        // Use for...of for async operations
        for (const order of orders) {
            try {
                await this.updateInventory(order.items);
                processedOrders.push(order.id);
            } catch (error) {
                console.error(`Failed to process order ${order.id}:`, error);
                // Can break out of loop if needed
                if (error.critical) break;
            }
        }
        
        return {
            totalRevenue,
            orderSummaries,
            processedOrders
        };
    }
    
    async sendOrderConfirmation(email) {
        // Simulate email sending
        console.log(`Sending confirmation to ${email}`);
    }
    
    async updateInventory(items) {
        // Simulate inventory update
        console.log(`Updating inventory for ${items.length} items`);
    }
}

// Sample data
const users = [
    { id: 1, name: 'John Doe', email: 'JOHN@EXAMPLE.COM', active: true },
    { id: 2, name: 'jane smith', email: 'Jane@Example.com', active: false },
    { id: 3, name: 'Bob Wilson', email: 'bob@example.com', active: true }
];

const processor = new UserProcessor(users);

console.log('=== forEach Method ===');
console.log(processor.processWithForEach());

console.log('\n=== map Method ===');
console.log(processor.processWithMap());

console.log('\n=== for...of Method ===');
console.log(processor.processWithForOf());
```

---

## Array Search & Filter

### Q3. find vs filter vs includes vs indexOf
**Answer:** Different search methods with different return values and use cases.

**Real-time Example:**
```javascript
// E-commerce Search System

class ProductSearch {
    constructor(products) {
        this.products = products;
    }
    
    // find() - Returns first matching element or undefined
    findProduct(productId) {
        return this.products.find(product => product.id === productId);
    }
    
    // findIndex() - Returns index of first matching element or -1
    findProductIndex(productId) {
        return this.products.findIndex(product => product.id === productId);
    }
    
    // filter() - Returns array of all matching elements
    filterByCategory(category) {
        return this.products.filter(product => product.category === category);
    }
    
    // includes() - Returns boolean if value exists
    hasCategory(category) {
        const categories = this.products.map(p => p.category);
        return categories.includes(category);
    }
    
    // indexOf() - Returns index of value or -1
    getCategoryIndex(category) {
        const categories = this.products.map(p => p.category);
        return categories.indexOf(category);
    }
    
    // some() - Returns true if at least one element matches
    hasInStockProducts() {
        return this.products.some(product => product.stock > 0);
    }
    
    // every() - Returns true if all elements match
    areAllProductsInStock() {
        return this.products.every(product => product.stock > 0);
    }
    
    // Real-world example: Complex product search
    searchProducts(criteria) {
        let results = [...this.products];
        
        // Filter by price range
        if (criteria.minPrice || criteria.maxPrice) {
            results = results.filter(product => {
                const price = product.price;
                const minMatch = !criteria.minPrice || price >= criteria.minPrice;
                const maxMatch = !criteria.maxPrice || price <= criteria.maxPrice;
                return minMatch && maxMatch;
            });
        }
        
        // Filter by category
        if (criteria.category) {
            results = results.filter(product => 
                product.category === criteria.category
            );
        }
        
        // Filter by search term
        if (criteria.searchTerm) {
            const term = criteria.searchTerm.toLowerCase();
            results = results.filter(product => 
                product.name.toLowerCase().includes(term) ||
                product.description?.toLowerCase().includes(term)
            );
        }
        
        // Filter by in stock
        if (criteria.inStock) {
            results = results.filter(product => product.stock > 0);
        }
        
        // Filter by rating
        if (criteria.minRating) {
            results = results.filter(product => 
                product.rating >= criteria.minRating
            );
        }
        
        // Filter by tags
        if (criteria.tags && criteria.tags.length > 0) {
            results = results.filter(product =>
                criteria.tags.some(tag => product.tags.includes(tag))
            );
        }
        
        return results;
    }
    
    // Find related products
    findRelatedProducts(productId, limit = 5) {
        const product = this.findProduct(productId);
        
        if (!product) return [];
        
        return this.products
            .filter(p => 
                p.id !== productId && 
                p.category === product.category
            )
            .sort((a, b) => b.rating - a.rating)
            .slice(0, limit);
    }
}

// Sample products
const products = [
    { 
        id: 1, 
        name: 'Laptop Pro', 
        price: 999.99, 
        category: 'Electronics', 
        stock: 10, 
        rating: 4.5,
        tags: ['laptop', 'computer', 'portable'],
        description: 'High-performance laptop for professionals'
    },
    { 
        id: 2, 
        name: 'Wireless Mouse', 
        price: 29.99, 
        category: 'Electronics', 
        stock: 50, 
        rating: 4.0,
        tags: ['mouse', 'wireless', 'accessory']
    },
    { 
        id: 3, 
        name: 'Programming Book', 
        price: 49.99, 
        category: 'Books', 
        stock: 0, 
        rating: 4.8,
        tags: ['book', 'programming', 'learning']
    },
    { 
        id: 4, 
        name: 'Mechanical Keyboard', 
        price: 129.99, 
        category: 'Electronics', 
        stock: 25, 
        rating: 4.6,
        tags: ['keyboard', 'mechanical', 'accessory']
    }
];

const search = new ProductSearch(products);

console.log('=== Find Product by ID ===');
console.log(search.findProduct(1));

console.log('\n=== Filter by Category ===');
console.log(search.filterByCategory('Electronics'));

console.log('\n=== Search with Criteria ===');
console.log(search.searchProducts({
    category: 'Electronics',
    minPrice: 50,
    maxPrice: 1000,
    inStock: true,
    minRating: 4.0
}));

console.log('\n=== Related Products ===');
console.log(search.findRelatedProducts(1, 3));

console.log('\n=== Check Stock Status ===');
console.log('Has in-stock products:', search.hasInStockProducts());
console.log('All products in stock:', search.areAllProductsInStock());
```

---

## Array Transformation

### Q4. map vs flatMap vs flat
**Answer:** Transform arrays in different ways - mapping, flattening, or both.

**Real-time Example:**
```javascript
// E-commerce Order Processing

class OrderProcessor {
    constructor(orders) {
        this.orders = orders;
    }
    
    // map() - Transform each element
    extractOrderIds() {
        return this.orders.map(order => order.id);
    }
    
    // map() with transformation
    calculateOrderTotals() {
        return this.orders.map(order => ({
            orderId: order.id,
            customerName: order.customer.name,
            itemCount: order.items.length,
            subtotal: order.items.reduce((sum, item) => 
                sum + (item.price * item.quantity), 0
            ),
            tax: order.items.reduce((sum, item) => 
                sum + (item.price * item.quantity * 0.1), 0
            ),
            total: order.items.reduce((sum, item) => 
                sum + (item.price * item.quantity * 1.1), 0
            )
        }));
    }
    
    // flatMap() - Map and flatten in one operation
    getAllOrderItems() {
        return this.orders.flatMap(order => 
            order.items.map(item => ({
                orderId: order.id,
                productId: item.productId,
                productName: item.name,
                quantity: item.quantity,
                price: item.price
            }))
        );
    }
    
    // flat() - Flatten nested arrays
    flattenCategories() {
        const categories = this.orders.map(order => 
            order.items.map(item => item.category)
        );
        
        return categories.flat();
    }
    
    // flat(depth) - Flatten with specific depth
    flattenDeep(array, depth = Infinity) {
        return array.flat(depth);
    }
    
    // Real-world example: Generate product recommendations
    generateRecommendations() {
        // Get all purchased products
        const purchasedProducts = this.orders.flatMap(order => 
            order.items.map(item => item.productId)
        );
        
        // Remove duplicates
        const uniqueProducts = [...new Set(purchasedProducts)];
        
        // Get categories of purchased products
        const categories = this.orders
            .flatMap(order => order.items)
            .map(item => item.category)
            .filter((category, index, self) => 
                self.indexOf(category) === index
            );
        
        return {
            purchasedProducts: uniqueProducts,
            categories,
            recommendationScore: uniqueProducts.length * categories.length
        };
    }
    
    // Complex transformation: Sales report
    generateSalesReport() {
        // Get all items with order details
        const allItems = this.orders.flatMap(order => 
            order.items.map(item => ({
                ...item,
                orderId: order.id,
                orderDate: order.date,
                customer: order.customer.name
            }))
        );
        
        // Group by category
        const byCategory = allItems.reduce((acc, item) => {
            const category = item.category;
            
            if (!acc[category]) {
                acc[category] = {
                    category,
                    totalSales: 0,
                    itemsSold: 0,
                    orders: []
                };
            }
            
            acc[category].totalSales += item.price * item.quantity;
            acc[category].itemsSold += item.quantity;
            acc[category].orders.push(item.orderId);
            
            return acc;
        }, {});
        
        return Object.values(byCategory).map(cat => ({
            ...cat,
            averageOrderValue: cat.totalSales / new Set(cat.orders).size
        }));
    }
}

// Sample data
const orders = [
    {
        id: 'ORD001',
        date: '2025-01-15',
        customer: { id: 1, name: 'John Doe', email: 'john@example.com' },
        items: [
            { productId: 'P001', name: 'Laptop', price: 999.99, quantity: 1, category: 'Electronics' },
            { productId: 'P002', name: 'Mouse', price: 29.99, quantity: 2, category: 'Electronics' }
        ]
    },
    {
        id: 'ORD002',
        date: '2025-01-16',
        customer: { id: 2, name: 'Jane Smith', email: 'jane@example.com' },
        items: [
            { productId: 'P003', name: 'Book', price: 19.99, quantity: 3, category: 'Books' },
            { productId: 'P001', name: 'Laptop', price: 999.99, quantity: 1, category: 'Electronics' }
        ]
    },
    {
        id: 'ORD003',
        date: '2025-01-17',
        customer: { id: 1, name: 'John Doe', email: 'john@example.com' },
        items: [
            { productId: 'P004', name: 'Shirt', price: 39.99, quantity: 2, category: 'Clothing' }
        ]
    }
];

const processor = new OrderProcessor(orders);

console.log('=== Order IDs ===');
console.log(processor.extractOrderIds());

console.log('\n=== Order Totals ===');
console.log(processor.calculateOrderTotals());

console.log('\n=== All Order Items (flatMap) ===');
console.log(processor.getAllOrderItems());

console.log('\n=== Product Recommendations ===');
console.log(processor.generateRecommendations());

console.log('\n=== Sales Report by Category ===');
console.log(processor.generateSalesReport());
```

This is a comprehensive start to the Node.js array methods guide. Would you like me to continue with more sections including reduce, advanced methods, and more real-world examples?
