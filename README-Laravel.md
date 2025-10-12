# Laravel Interview Questions & Answers
### For 7+ Years Experienced PHP Developers

---

## Table of Contents
1. [Laravel Basics](#laravel-basics)
2. [Routing](#routing)
3. [Controllers & Middleware](#controllers--middleware)
4. [Eloquent ORM](#eloquent-orm)
5. [Database & Migrations](#database--migrations)
6. [Authentication & Authorization](#authentication--authorization)
7. [Service Container & Service Providers](#service-container--service-providers)
8. [Queues & Jobs](#queues--jobs)
9. [Testing](#testing)
10. [Performance & Optimization](#performance--optimization)

---

## Laravel Basics

### Q1. What is Laravel and why is it popular?
**Answer:** Laravel is a PHP web application framework with expressive, elegant syntax. It's popular because of:
- MVC architecture
- Built-in authentication and authorization
- Eloquent ORM for database operations
- Blade templating engine
- Artisan command-line tool
- Queue and cron job management
- Large ecosystem (Laravel Mix, Horizon, Telescope, etc.)

**Real-time Example:**
```php
// routes/web.php - Simple blog application
Route::get('/', [HomeController::class, 'index']);
Route::resource('posts', PostController::class);

// app/Http/Controllers/PostController.php
class PostController extends Controller
{
    public function index()
    {
        $posts = Post::with('author')
            ->published()
            ->latest()
            ->paginate(10);
            
        return view('posts.index', compact('posts'));
    }
    
    public function store(Request $request)
    {
        $validated = $request->validate([
            'title' => 'required|max:255',
            'content' => 'required',
            'image' => 'nullable|image|max:2048'
        ]);
        
        if ($request->hasFile('image')) {
            $validated['image'] = $request->file('image')->store('posts');
        }
        
        $post = auth()->user()->posts()->create($validated);
        
        return redirect()->route('posts.show', $post)
            ->with('success', 'Post created successfully!');
    }
}
```

### Q2. Explain Laravel application lifecycle
**Answer:** The Laravel request lifecycle:
1. Entry Point (public/index.php)
2. Load Autoloader (Composer)
3. Create Application Instance
4. Kernel Handling (HTTP/Console)
5. Service Providers Registration
6. Dispatch Request to Router
7. Middleware Processing
8. Controller Action Execution
9. Response Return
10. Terminate Middleware

**Real-time Example:**
```php
// public/index.php
require __DIR__.'/../vendor/autoload.php';
$app = require_once __DIR__.'/../bootstrap/app.php';

$kernel = $app->make(Kernel::class);
$response = $kernel->handle(
    $request = Request::capture()
);
$response->send();
$kernel->terminate($request, $response);

// app/Http/Kernel.php
class Kernel extends HttpKernel
{
    protected $middleware = [
        // Global middleware
        \App\Http\Middleware\TrustProxies::class,
        \Illuminate\Foundation\Http\Middleware\ValidatePostSize::class,
    ];
    
    protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        ],
        
        'api' => [
            'throttle:api',
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],
    ];
    
    protected $routeMiddleware = [
        'auth' => \App\Http\Middleware\Authenticate::class,
        'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
    ];
}
```

### Q3. What is the directory structure of Laravel?
**Answer:**
- **app/** - Core application code (Models, Controllers, etc.)
- **bootstrap/** - Framework bootstrap files
- **config/** - Configuration files
- **database/** - Migrations, seeds, factories
- **public/** - Entry point, assets
- **resources/** - Views, raw assets, language files
- **routes/** - Route definitions
- **storage/** - Logs, cache, compiled views
- **tests/** - Automated tests
- **vendor/** - Composer dependencies

---

## Routing

### Q4. How does routing work in Laravel?
**Answer:** Laravel routing allows you to map URLs to specific controller actions or closures.

**Real-time Example:**
```php
// routes/web.php

// Basic routes
Route::get('/', function () {
    return view('welcome');
});

// Route with parameters
Route::get('/users/{id}', [UserController::class, 'show'])
    ->name('users.show');

// Route with optional parameters
Route::get('/posts/{slug?}', [PostController::class, 'show']);

// Route with constraints
Route::get('/users/{id}', function ($id) {
    return "User $id";
})->where('id', '[0-9]+');

// Named routes
Route::get('/dashboard', [DashboardController::class, 'index'])
    ->name('dashboard');

// Route groups with prefix and middleware
Route::prefix('admin')
    ->middleware(['auth', 'admin'])
    ->name('admin.')
    ->group(function () {
        Route::get('/dashboard', [AdminController::class, 'dashboard'])
            ->name('dashboard');
        Route::resource('users', AdminUserController::class);
    });

// API routes with versioning
Route::prefix('api/v1')
    ->middleware('api')
    ->group(function () {
        Route::apiResource('posts', Api\PostController::class);
    });

// Usage in controller
public function redirect()
{
    return redirect()->route('users.show', ['id' => 1]);
    // Generates: /users/1
}

// In Blade
<a href="{{ route('dashboard') }}">Dashboard</a>
<a href="{{ route('users.show', $user->id) }}">View User</a>
```

### Q5. What is route model binding?
**Answer:** Route model binding automatically injects model instances directly into routes based on the route parameter.

**Real-time Example:**
```php
// Implicit binding
Route::get('/posts/{post}', function (Post $post) {
    return view('posts.show', compact('post'));
});
// Laravel automatically finds Post by ID

// Custom key for binding
public function getRouteKeyName()
{
    return 'slug'; // Use slug instead of id
}

Route::get('/posts/{post:slug}', function (Post $post) {
    return view('posts.show', compact('post'));
});

// Explicit binding in RouteServiceProvider
public function boot()
{
    Route::model('user', User::class);
    
    Route::bind('post', function ($value) {
        return Post::where('slug', $value)
            ->published()
            ->firstOrFail();
    });
}

// Controller example with multiple bindings
Route::get('/users/{user}/posts/{post}', [PostController::class, 'show']);

class PostController extends Controller
{
    public function show(User $user, Post $post)
    {
        // Both models are automatically injected
        // Ensures post belongs to user
        if ($post->user_id !== $user->id) {
            abort(404);
        }
        
        return view('posts.show', compact('user', 'post'));
    }
}

// Scoped binding (Laravel 7+)
Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    // Automatically scopes post to user
    return view('posts.show', compact('user', 'post'));
})->scopeBindings();
```

---

## Controllers & Middleware

### Q6. What are controllers in Laravel?
**Answer:** Controllers group related request handling logic into a single class.

**Real-time Example:**
```php
// app/Http/Controllers/PostController.php
namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\Request;

class PostController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth')->except(['index', 'show']);
        $this->middleware('verified')->only(['create', 'store']);
    }
    
    public function index()
    {
        $posts = Post::with(['author', 'category'])
            ->withCount('comments')
            ->latest()
            ->paginate(15);
            
        return view('posts.index', compact('posts'));
    }
    
    public function create()
    {
        $categories = Category::pluck('name', 'id');
        return view('posts.create', compact('categories'));
    }
    
    public function store(Request $request)
    {
        $validated = $request->validate([
            'title' => 'required|string|max:255',
            'slug' => 'required|string|unique:posts,slug',
            'content' => 'required|string',
            'category_id' => 'required|exists:categories,id',
            'image' => 'nullable|image|mimes:jpg,png|max:2048',
            'tags' => 'nullable|array',
            'tags.*' => 'exists:tags,id'
        ]);
        
        if ($request->hasFile('image')) {
            $validated['image'] = $request->file('image')
                ->store('posts', 'public');
        }
        
        $post = $request->user()->posts()->create($validated);
        
        if ($request->has('tags')) {
            $post->tags()->attach($request->tags);
        }
        
        return redirect()
            ->route('posts.show', $post)
            ->with('success', 'Post created successfully!');
    }
    
    public function show(Post $post)
    {
        $post->load(['author', 'comments.user', 'tags']);
        $post->increment('views');
        
        $relatedPosts = Post::where('category_id', $post->category_id)
            ->where('id', '!=', $post->id)
            ->limit(3)
            ->get();
        
        return view('posts.show', compact('post', 'relatedPosts'));
    }
    
    public function edit(Post $post)
    {
        $this->authorize('update', $post);
        
        $categories = Category::pluck('name', 'id');
        $tags = Tag::pluck('name', 'id');
        
        return view('posts.edit', compact('post', 'categories', 'tags'));
    }
    
    public function update(Request $request, Post $post)
    {
        $this->authorize('update', $post);
        
        $validated = $request->validate([
            'title' => 'required|string|max:255',
            'content' => 'required|string',
            'category_id' => 'required|exists:categories,id',
        ]);
        
        $post->update($validated);
        
        if ($request->has('tags')) {
            $post->tags()->sync($request->tags);
        }
        
        return redirect()
            ->route('posts.show', $post)
            ->with('success', 'Post updated successfully!');
    }
    
    public function destroy(Post $post)
    {
        $this->authorize('delete', $post);
        
        $post->delete();
        
        return redirect()
            ->route('posts.index')
            ->with('success', 'Post deleted successfully!');
    }
}
```

### Q7. What is middleware in Laravel?
**Answer:** Middleware provides a mechanism for filtering HTTP requests entering your application.

**Real-time Example:**
```php
// Creating middleware
php artisan make:middleware CheckUserRole

// app/Http/Middleware/CheckUserRole.php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class CheckUserRole
{
    public function handle(Request $request, Closure $next, ...$roles)
    {
        if (!$request->user()) {
            return redirect('login');
        }
        
        if (!$request->user()->hasAnyRole($roles)) {
            abort(403, 'Unauthorized action.');
        }
        
        return $next($request);
    }
}

// Registering in app/Http/Kernel.php
protected $routeMiddleware = [
    'role' => \App\Http\Middleware\CheckUserRole::class,
];

// Using in routes
Route::get('/admin/dashboard', [AdminController::class, 'dashboard'])
    ->middleware('role:admin,super-admin');

// Custom API rate limiting middleware
class ThrottleApiRequests
{
    public function handle(Request $request, Closure $next)
    {
        $user = $request->user();
        $key = 'api_rate_limit:' . $user->id;
        
        if (Cache::has($key)) {
            $attempts = Cache::get($key);
            
            if ($attempts >= 100) {
                return response()->json([
                    'error' => 'Rate limit exceeded. Try again in 1 minute.'
                ], 429);
            }
            
            Cache::increment($key);
        } else {
            Cache::put($key, 1, 60); // 60 seconds
        }
        
        return $next($request);
    }
}

// Logging middleware
class LogActivity
{
    public function handle(Request $request, Closure $next)
    {
        $startTime = microtime(true);
        
        $response = $next($request);
        
        $duration = microtime(true) - $startTime;
        
        ActivityLog::create([
            'user_id' => auth()->id(),
            'url' => $request->fullUrl(),
            'method' => $request->method(),
            'ip' => $request->ip(),
            'user_agent' => $request->userAgent(),
            'response_code' => $response->getStatusCode(),
            'duration' => $duration
        ]);
        
        return $response;
    }
}

// Terminate middleware (executed after response sent)
class TerminatingMiddleware
{
    public function handle(Request $request, Closure $next)
    {
        return $next($request);
    }
    
    public function terminate(Request $request, Response $response)
    {
        // Perform action after response is sent to browser
        Log::info('Response sent', [
            'url' => $request->url(),
            'status' => $response->getStatusCode()
        ]);
    }
}
```

---

## Eloquent ORM

### Q8. What is Eloquent ORM?
**Answer:** Eloquent is Laravel's active record ORM implementation for database interactions.

**Real-time Example:**
```php
// app/Models/Post.php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Database\Eloquent\Factories\HasFactory;

class Post extends Model
{
    use HasFactory, SoftDeletes;
    
    protected $fillable = [
        'title', 'slug', 'content', 'excerpt',
        'image', 'user_id', 'category_id', 'published_at'
    ];
    
    protected $casts = [
        'published_at' => 'datetime',
        'views' => 'integer',
        'is_featured' => 'boolean',
        'metadata' => 'array'
    ];
    
    protected $dates = ['published_at', 'deleted_at'];
    
    protected $appends = ['is_published', 'reading_time'];
    
    // Relationships
    public function user()
    {
        return $this->belongsTo(User::class);
    }
    
    public function category()
    {
        return $this->belongsTo(Category::class);
    }
    
    public function tags()
    {
        return $this->belongsToMany(Tag::class)
            ->withTimestamps();
    }
    
    public function comments()
    {
        return $this->hasMany(Comment::class);
    }
    
    // Scopes
    public function scopePublished($query)
    {
        return $query->where('published_at', '<=', now());
    }
    
    public function scopeFeatured($query)
    {
        return $query->where('is_featured', true);
    }
    
    public function scopeByAuthor($query, User $user)
    {
        return $query->where('user_id', $user->id);
    }
    
    // Accessors
    public function getIsPublishedAttribute()
    {
        return $this->published_at && $this->published_at->isPast();
    }
    
    public function getReadingTimeAttribute()
    {
        $wordCount = str_word_count(strip_tags($this->content));
        return ceil($wordCount / 200); // 200 words per minute
    }
    
    public function getExcerptAttribute($value)
    {
        return $value ?: substr(strip_tags($this->content), 0, 150) . '...';
    }
    
    // Mutators
    public function setTitleAttribute($value)
    {
        $this->attributes['title'] = $value;
        $this->attributes['slug'] = Str::slug($value);
    }
    
    // Events
    protected static function boot()
    {
        parent::boot();
        
        static::creating(function ($post) {
            if (empty($post->user_id)) {
                $post->user_id = auth()->id();
            }
        });
        
        static::created(function ($post) {
            // Clear cache, send notifications, etc.
            Cache::tags('posts')->flush();
        });
        
        static::deleting(function ($post) {
            // Delete associated records
            $post->comments()->delete();
        });
    }
}

// Usage examples
// Create
$post = Post::create([
    'title' => 'My Post',
    'content' => 'Post content',
    'category_id' => 1
]);

// Find
$post = Post::find(1);
$post = Post::findOrFail(1);
$post = Post::where('slug', 'my-post')->first();

// Update
$post->update(['title' => 'Updated Title']);

// Delete
$post->delete(); // Soft delete
$post->forceDelete(); // Permanent delete

// Query with relationships
$posts = Post::with(['user', 'category', 'tags'])
    ->published()
    ->latest()
    ->paginate(10);

// Eager loading
$posts = Post::with('user')->get();

// Lazy eager loading
$posts = Post::all();
$posts->load('user');

// Count relationships
$users = User::withCount('posts')->get();

// Exists check
$exists = Post::where('slug', 'my-post')->exists();

// Advanced queries
$posts = Post::where('published_at', '<=', now())
    ->whereHas('tags', function ($query) {
        $query->where('name', 'laravel');
    })
    ->orWhereHas('category', function ($query) {
        $query->where('slug', 'php');
    })
    ->orderBy('views', 'desc')
    ->limit(10)
    ->get();
```

### Q9. Explain relationships in Eloquent
**Answer:** Eloquent supports various types of relationships:

**Real-time Example:**
```php
// One to One
class User extends Model
{
    public function profile()
    {
        return $this->hasOne(Profile::class);
    }
}

class Profile extends Model
{
    public function user()
    {
        return $this->belongsTo(User::class);
    }
}

// Usage
$user = User::find(1);
$profile = $user->profile;
$user = $profile->user;

// One to Many
class Category extends Model
{
    public function posts()
    {
        return $this->hasMany(Post::class);
    }
}

class Post extends Model
{
    public function category()
    {
        return $this->belongsTo(Category::class);
    }
}

// Usage
$category = Category::find(1);
$posts = $category->posts;
$category = $posts->first()->category;

// Many to Many
class Post extends Model
{
    public function tags()
    {
        return $this->belongsToMany(Tag::class)
            ->withPivot('created_by')
            ->withTimestamps();
    }
}

class Tag extends Model
{
    public function posts()
    {
        return $this->belongsToMany(Post::class);
    }
}

// Usage
$post = Post::find(1);
$tags = $post->tags;

// Attach/Detach
$post->tags()->attach([1, 2, 3]);
$post->tags()->detach(1);
$post->tags()->sync([1, 2, 3]); // Remove all except these
$post->tags()->syncWithoutDetaching([4, 5]); // Add without removing

// Has Many Through
class Country extends Model
{
    public function posts()
    {
        return $this->hasManyThrough(
            Post::class,
            User::class,
            'country_id', // Foreign key on users table
            'user_id',    // Foreign key on posts table
            'id',         // Local key on countries table
            'id'          // Local key on users table
        );
    }
}

// Polymorphic Relations
class Image extends Model
{
    public function imageable()
    {
        return $this->morphTo();
    }
}

class Post extends Model
{
    public function images()
    {
        return $this->morphMany(Image::class, 'imageable');
    }
}

class User extends Model
{
    public function images()
    {
        return $this->morphMany(Image::class, 'imageable');
    }
}

// Usage
$post = Post::find(1);
$post->images()->create(['url' => 'image.jpg']);

$image = Image::find(1);
$owner = $image->imageable; // Can be Post or User

// Many to Many Polymorphic
class Tag extends Model
{
    public function posts()
    {
        return $this->morphedByMany(Post::class, 'taggable');
    }
    
    public function videos()
    {
        return $this->morphedByMany(Video::class, 'taggable');
    }
}

class Post extends Model
{
    public function tags()
    {
        return $this->morphToMany(Tag::class, 'taggable');
    }
}
```

---

## Database & Migrations

### Q10. What are migrations in Laravel?
**Answer:** Migrations are version control for your database, allowing you to define and share database schema.

**Real-time Example:**
```php
// Creating migration
php artisan make:migration create_posts_table

// database/migrations/2025_10_12_000000_create_posts_table.php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up()
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->onDelete('cascade');
            $table->foreignId('category_id')->constrained()->onDelete('restrict');
            $table->string('title');
            $table->string('slug')->unique();
            $table->text('excerpt')->nullable();
            $table->longText('content');
            $table->string('image')->nullable();
            $table->integer('views')->default(0);
            $table->boolean('is_featured')->default(false);
            $table->json('metadata')->nullable();
            $table->timestamp('published_at')->nullable();
            $table->timestamps();
            $table->softDeletes();
            
            // Indexes
            $table->index('published_at');
            $table->index(['user_id', 'published_at']);
            $table->fullText('title');
            $table->fullText(['title', 'content']);
        });
        
        // Pivot table
        Schema::create('post_tag', function (Blueprint $table) {
            $table->foreignId('post_id')->constrained()->onDelete('cascade');
            $table->foreignId('tag_id')->constrained()->onDelete('cascade');
            $table->timestamps();
            
            $table->primary(['post_id', 'tag_id']);
        });
    }
    
    public function down()
    {
        Schema::dropIfExists('post_tag');
        Schema::dropIfExists('posts');
    }
};

// Modifying table
php artisan make:migration add_views_to_posts_table

public function up()
{
    Schema::table('posts', function (Blueprint $table) {
        $table->integer('views')->default(0)->after('content');
        $table->boolean('is_featured')->default(false)->after('views');
    });
}

public function down()
{
    Schema::table('posts', function (Blueprint $table) {
        $table->dropColumn(['views', 'is_featured']);
    });
}

// Running migrations
php artisan migrate
php artisan migrate:rollback
php artisan migrate:rollback --step=2
php artisan migrate:reset
php artisan migrate:refresh
php artisan migrate:fresh --seed
```

This covers key Laravel concepts. Would you like me to continue with more sections?

