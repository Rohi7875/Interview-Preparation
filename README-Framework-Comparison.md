# Framework Comparison Guide
### Laravel vs CodeIgniter vs Express.js - Interview Questions

---

## Table of Contents
1. [Framework Overview](#framework-overview)
2. [Routing Comparison](#routing-comparison)
3. [MVC Pattern](#mvc-pattern)
4. [Database Operations](#database-operations)
5. [Authentication](#authentication)
6. [Middleware](#middleware)
7. [File Upload](#file-upload)
8. [API Development](#api-development)
9. [Testing](#testing)
10. [When to Use Which](#when-to-use-which)

---

## Framework Overview

### Comparison Table

| Feature | Laravel | CodeIgniter | Express.js |
|---------|---------|-------------|------------|
| Language | PHP | PHP | JavaScript/Node.js |
| Architecture | MVC | MVC | Minimal/Flexible |
| Learning Curve | Medium | Easy | Easy-Medium |
| ORM | Eloquent | Query Builder | None (use Mongoose/Sequelize) |
| Template Engine | Blade | PHP Views | EJS/Pug/Handlebars |
| CLI Tool | Artisan | spark | npm scripts |
| Package Manager | Composer | Composer | NPM/Yarn |
| Community | Large | Medium | Very Large |
| Real-time | Laravel Echo | None | Socket.io |
| Best For | Full-stack | Small-Medium | APIs, Real-time |

---

## Routing Comparison

### Basic Routing

**Laravel:**
```php
// routes/web.php
use App\Http\Controllers\UserController;

// Basic routes
Route::get('/', function () {
    return view('welcome');
});

Route::get('/users', [UserController::class, 'index']);
Route::get('/users/{id}', [UserController::class, 'show']);
Route::post('/users', [UserController::class, 'store']);
Route::put('/users/{id}', [UserController::class, 'update']);
Route::delete('/users/{id}', [UserController::class, 'destroy']);

// Resource routing (creates all CRUD routes)
Route::resource('posts', PostController::class);

// Route groups with middleware
Route::middleware(['auth'])->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index']);
    Route::resource('orders', OrderController::class);
});

// Route prefixes
Route::prefix('admin')->group(function () {
    Route::get('/users', [Admin\UserController::class, 'index']);
});

// Named routes
Route::get('/profile', [ProfileController::class, 'show'])->name('profile');
// Usage: route('profile')
```

**CodeIgniter:**
```php
// app/Config/Routes.php
$routes->get('/', 'Home::index');

// Basic routes
$routes->get('users', 'UserController::index');
$routes->get('users/(:num)', 'UserController::show/$1');
$routes->post('users', 'UserController::store');
$routes->put('users/(:num)', 'UserController::update/$1');
$routes->delete('users/(:num)', 'UserController::destroy/$1');

// Resource routing
$routes->resource('posts', ['controller' => 'PostController']);

// Route groups
$routes->group('admin', ['filter' => 'auth'], function($routes) {
    $routes->get('users', 'Admin\UserController::index');
    $routes->get('dashboard', 'Admin\DashboardController::index');
});

// Named routes
$routes->get('profile', 'ProfileController::show', ['as' => 'profile']);
// Usage: route_to('profile')
```

**Express.js:**
```javascript
// app.js or routes file
const express = require('express');
const app = express();
const userController = require('./controllers/UserController');

// Basic routes
app.get('/', (req, res) => {
    res.send('Welcome');
});

app.get('/users', userController.index);
app.get('/users/:id', userController.show);
app.post('/users', userController.store);
app.put('/users/:id', userController.update);
app.delete('/users/:id', userController.destroy);

// Router for modular routes
const router = express.Router();
router.get('/', userController.index);
router.get('/:id', userController.show);
router.post('/', userController.store);
router.put('/:id', userController.update);
router.delete('/:id', userController.destroy);

app.use('/users', router);

// Route with middleware
const authenticate = require('./middleware/auth');
app.get('/dashboard', authenticate, dashboardController.index);

// Route groups (using router)
const adminRouter = express.Router();
adminRouter.use(authenticate);
adminRouter.get('/users', adminUserController.index);
app.use('/admin', adminRouter);

// Named routes (using path-to-regexp or custom implementation)
const routes = {
    profile: '/profile',
    userShow: '/users/:id'
};
```

---

## MVC Pattern

### Controller Comparison

**Laravel Controller:**
```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;

class UserController extends Controller
{
    public function index(Request $request)
    {
        $users = User::query()
            ->when($request->search, function ($query, $search) {
                $query->where('name', 'like', "%{$search}%");
            })
            ->paginate(10);
        
        return view('users.index', compact('users'));
    }
    
    public function store(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users,email',
            'password' => 'required|min:8'
        ]);
        
        $validated['password'] = bcrypt($validated['password']);
        
        $user = User::create($validated);
        
        return redirect()
            ->route('users.show', $user)
            ->with('success', 'User created successfully');
    }
    
    public function show(User $user) // Route model binding
    {
        return view('users.show', compact('user'));
    }
    
    public function update(Request $request, User $user)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users,email,' . $user->id
        ]);
        
        $user->update($validated);
        
        return redirect()
            ->route('users.show', $user)
            ->with('success', 'User updated successfully');
    }
}
```

**CodeIgniter Controller:**
```php
<?php

namespace App\Controllers;

use App\Models\UserModel;

class UserController extends BaseController
{
    protected $userModel;
    
    public function __construct()
    {
        $this->userModel = new UserModel();
        helper(['form', 'url']);
    }
    
    public function index()
    {
        $search = $this->request->getGet('search');
        
        $builder = $this->userModel->builder();
        
        if ($search) {
            $builder->like('name', $search);
        }
        
        $users = $builder->paginate(10);
        $pager = $this->userModel->pager;
        
        return view('users/index', [
            'users' => $users,
            'pager' => $pager
        ]);
    }
    
    public function store()
    {
        $validation = \Config\Services::validation();
        
        $validation->setRules([
            'name' => 'required|max_length[255]',
            'email' => 'required|valid_email|is_unique[users.email]',
            'password' => 'required|min_length[8]'
        ]);
        
        if (!$validation->withRequest($this->request)->run()) {
            return redirect()->back()
                ->withInput()
                ->with('errors', $validation->getErrors());
        }
        
        $data = [
            'name' => $this->request->getPost('name'),
            'email' => $this->request->getPost('email'),
            'password' => password_hash($this->request->getPost('password'), PASSWORD_DEFAULT)
        ];
        
        $userId = $this->userModel->insert($data);
        
        return redirect()->to('/users/' . $userId)
            ->with('success', 'User created successfully');
    }
    
    public function show($id)
    {
        $user = $this->userModel->find($id);
        
        if (!$user) {
            throw new \CodeIgniter\Exceptions\PageNotFoundException('User not found');
        }
        
        return view('users/show', ['user' => $user]);
    }
    
    public function update($id)
    {
        $validation = \Config\Services::validation();
        
        $validation->setRules([
            'name' => 'required|max_length[255]',
            'email' => "required|valid_email|is_unique[users.email,id,{$id}]"
        ]);
        
        if (!$validation->withRequest($this->request)->run()) {
            return redirect()->back()
                ->withInput()
                ->with('errors', $validation->getErrors());
        }
        
        $data = [
            'name' => $this->request->getPost('name'),
            'email' => $this->request->getPost('email')
        ];
        
        $this->userModel->update($id, $data);
        
        return redirect()->to('/users/' . $id)
            ->with('success', 'User updated successfully');
    }
}
```

**Express.js Controller:**
```javascript
// controllers/UserController.js
const User = require('../models/User');
const { validationResult } = require('express-validator');

class UserController {
    static async index(req, res, next) {
        try {
            const { search, page = 1, limit = 10 } = req.query;
            
            const query = {};
            if (search) {
                query.name = new RegExp(search, 'i');
            }
            
            const users = await User.find(query)
                .limit(limit * 1)
                .skip((page - 1) * limit)
                .select('-password');
            
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
    
    static async store(req, res, next) {
        try {
            const errors = validationResult(req);
            if (!errors.isEmpty()) {
                return res.status(400).json({ errors: errors.array() });
            }
            
            const { name, email, password } = req.body;
            
            // Check if user exists
            const existingUser = await User.findOne({ email });
            if (existingUser) {
                return res.status(409).json({ error: 'Email already exists' });
            }
            
            // Hash password
            const bcrypt = require('bcrypt');
            const hashedPassword = await bcrypt.hash(password, 10);
            
            // Create user
            const user = await User.create({
                name,
                email,
                password: hashedPassword
            });
            
            res.status(201).json({
                message: 'User created successfully',
                user: {
                    id: user._id,
                    name: user.name,
                    email: user.email
                }
            });
        } catch (error) {
            next(error);
        }
    }
    
    static async show(req, res, next) {
        try {
            const user = await User.findById(req.params.id)
                .select('-password');
            
            if (!user) {
                return res.status(404).json({ error: 'User not found' });
            }
            
            res.json(user);
        } catch (error) {
            next(error);
        }
    }
    
    static async update(req, res, next) {
        try {
            const errors = validationResult(req);
            if (!errors.isEmpty()) {
                return res.status(400).json({ errors: errors.array() });
            }
            
            const { name, email } = req.body;
            
            // Check email uniqueness
            if (email) {
                const existingUser = await User.findOne({
                    email,
                    _id: { $ne: req.params.id }
                });
                
                if (existingUser) {
                    return res.status(409).json({ error: 'Email already exists' });
                }
            }
            
            const user = await User.findByIdAndUpdate(
                req.params.id,
                { name, email },
                { new: true, runValidators: true }
            ).select('-password');
            
            if (!user) {
                return res.status(404).json({ error: 'User not found' });
            }
            
            res.json({
                message: 'User updated successfully',
                user
            });
        } catch (error) {
            next(error);
        }
    }
}

module.exports = UserController;

// Validation middleware (Express)
const { body } = require('express-validator');

const userValidation = {
    store: [
        body('name').notEmpty().withMessage('Name is required')
            .isLength({ max: 255 }).withMessage('Name too long'),
        body('email').isEmail().withMessage('Invalid email'),
        body('password').isLength({ min: 8 }).withMessage('Password must be at least 8 characters')
    ],
    update: [
        body('name').notEmpty().withMessage('Name is required'),
        body('email').isEmail().withMessage('Invalid email')
    ]
};

// routes/users.js
const router = express.Router();
router.get('/', UserController.index);
router.post('/', userValidation.store, UserController.store);
router.get('/:id', UserController.show);
router.put('/:id', userValidation.update, UserController.update);
```

---

## Database Operations

### Query Comparison

**Laravel (Eloquent):**
```php
// Simple queries
$users = User::all();
$user = User::find(1);
$user = User::where('email', 'john@example.com')->first();

// Complex queries
$users = User::where('active', true)
    ->where('age', '>', 18)
    ->orderBy('created_at', 'desc')
    ->limit(10)
    ->get();

// Relationships
$user = User::with(['posts', 'comments'])->find(1);
$posts = $user->posts;

// Aggregations
$count = User::where('active', true)->count();
$average = Order::where('status', 'completed')->avg('total');

// Raw queries
$users = DB::select('SELECT * FROM users WHERE age > ?', [18]);

// Transactions
DB::transaction(function () {
    $user = User::create([...]);
    $order = Order::create([...]);
});
```

**CodeIgniter (Query Builder):**
```php
// Simple queries
$users = $this->db->get('users')->getResultArray();
$user = $this->db->get_where('users', ['id' => 1])->getRowArray();

// Complex queries
$users = $this->db->where('active', 1)
    ->where('age >', 18)
    ->order_by('created_at', 'DESC')
    ->limit(10)
    ->get('users')
    ->getResultArray();

// Joins
$this->db->select('users.*, orders.total')
    ->from('users')
    ->join('orders', 'orders.user_id = users.id')
    ->get()
    ->getResultArray();

// Aggregations
$count = $this->db->where('active', 1)
    ->count_all_results('users');

// Raw queries
$query = $this->db->query('SELECT * FROM users WHERE age > ?', [18]);
$users = $query->getResultArray();

// Transactions
$this->db->transStart();
$this->db->insert('users', $userData);
$this->db->insert('orders', $orderData);
$this->db->transComplete();
```

**Express.js (Mongoose):**
```javascript
// Simple queries
const users = await User.find();
const user = await User.findById(id);
const user = await User.findOne({ email: 'john@example.com' });

// Complex queries
const users = await User.find({
    active: true,
    age: { $gt: 18 }
})
.sort({ createdAt: -1 })
.limit(10);

// Relationships (populate)
const user = await User.findById(id)
    .populate('posts')
    .populate('comments');

// Aggregations
const count = await User.countDocuments({ active: true });
const average = await Order.aggregate([
    { $match: { status: 'completed' } },
    { $group: { _id: null, avgTotal: { $avg: '$total' } } }
]);

// Raw queries
const users = await User.find().where('age').gt(18);

// Transactions
const session = await mongoose.startSession();
session.startTransaction();
try {
    const user = await User.create([userData], { session });
    const order = await Order.create([orderData], { session });
    await session.commitTransaction();
} catch (error) {
    await session.abortTransaction();
    throw error;
} finally {
    session.endSession();
}
```

This is a great start to the framework comparison guide. Would you like me to continue with Authentication, Middleware, API Development sections, and create a final comprehensive summary?
