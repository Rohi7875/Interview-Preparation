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
11. [Advanced Laravel Concepts](#advanced-laravel-concepts)
12. [Laravel Security](#laravel-security)
13. [Laravel API Development](#laravel-api-development)
14. [Laravel Caching & Sessions](#laravel-caching--sessions)
15. [Laravel File Storage](#laravel-file-storage)
16. [Laravel Broadcasting & Events](#laravel-broadcasting--events)
17. [Laravel Artisan Commands](#laravel-artisan-commands)
18. [Laravel Localization](#laravel-localization)
19. [Laravel Validation](#laravel-validation)
20. [Laravel Blade Templates](#laravel-blade-templates)

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

### Q11. What are Laravel Events and Listeners?
**Answer:** Events provide a simple observer pattern implementation.

**Example:**
```php
// Event
class OrderPlaced
{
    public $order;
    public function __construct(Order $order) {
        $this->order = $order;
    }
}

// Listener
class SendOrderConfirmation
{
    public function handle(OrderPlaced $event) {
        Mail::to($event->order->customer->email)
            ->send(new OrderConfirmationMail($event->order));
    }
}

// Usage
event(new OrderPlaced($order));
```

### Q12. Explain Laravel Collections
**Answer:** Collections wrap arrays with helpful methods.

```php
$users = User::all();
$activeUsers = $users->where('active', true);
$names = $users->pluck('name');
$grouped = $users->groupBy('role');
$sorted = $users->sortBy('name');
```

### Q13. What are Policies and Gates?
**Answer:** Authorization logic organization.

```php
// Policy
class PostPolicy {
    public function update(User $user, Post $post) {
        return $user->id === $post->user_id;
    }
}

// Usage
$this->authorize('update', $post);
```

### Q14. Laravel Notifications
**Answer:** Multi-channel notification system.

```php
class InvoicePaid extends Notification {
    public function via($notifiable) {
        return ['mail', 'database', 'slack'];
    }
    public function toMail($notifiable) {
        return (new MailMessage)->line('Invoice paid!');
    }
}

$user->notify(new InvoicePaid($invoice));
```

### Q15. Laravel Sanctum
**Answer:** API authentication for SPAs and mobile apps.

```php
// Create token
$token = $user->createToken('api-token')->plainTextToken;

// Protected routes
Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});
```

### Q16. Laravel Queues & Jobs - WHY and HOW?
**Answer:** Queues allow you to defer time-consuming tasks (like sending emails, processing images, or calling external APIs) to be processed later, making your application feel faster to users.

**Why Use Queues?**
- Don't make users wait for slow operations
- Handle tasks in background
- Retry failed operations automatically
- Process tasks during off-peak hours

**Real-time Example:**
```php
// PROBLEM: Without Queue - User waits 5+ seconds
public function register(Request $request)
{
    $user = User::create($request->all());
    
    // These slow operations block the response
    Mail::to($user)->send(new WelcomeEmail($user));        // 2 seconds
    $this->resizeProfileImage($user->image);               // 2 seconds
    $this->notifyAdmins($user);                            // 1 second
    $this->updateAnalytics($user);                         // 1 second
    
    return redirect('/dashboard'); // User waits 6 seconds!
}

// SOLUTION: With Queue - User waits <1 second
public function register(Request $request)
{
    $user = User::create($request->all());
    
    // Queue these operations - happens in background
    SendWelcomeEmail::dispatch($user);
    ResizeImage::dispatch($user->image);
    NotifyAdmins::dispatch($user);
    UpdateAnalytics::dispatch($user);
    
    return redirect('/dashboard'); // User waits <1 second!
}

// Creating the Job
php artisan make:job SendWelcomeEmail

// app/Jobs/SendWelcomeEmail.php
namespace App\Jobs;

use App\Models\User;
use App\Mail\WelcomeEmail;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Illuminate\Support\Facades\Mail;

class SendWelcomeEmail implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
    
    public $user;
    public $tries = 3; // Retry 3 times if failed
    public $timeout = 120; // Timeout after 2 minutes
    
    public function __construct(User $user)
    {
        $this->user = $user;
    }
    
    public function handle()
    {
        // This runs in background worker
        Mail::to($this->user->email)
            ->send(new WelcomeEmail($this->user));
    }
    
    // If job fails after all retries
    public function failed(\Throwable $exception)
    {
        // Send notification to admin
        // Log the error
        \Log::error('Failed to send welcome email', [
            'user_id' => $this->user->id,
            'error' => $exception->getMessage()
        ]);
    }
}

// Dispatching with delay
SendWelcomeEmail::dispatch($user)->delay(now()->addMinutes(10));

// Dispatching to specific queue
SendWelcomeEmail::dispatch($user)->onQueue('emails');

// Chain jobs - execute in sequence
Bus::chain([
    new ProcessOrder($order),
    new ChargeCustomer($order),
    new SendReceipt($order),
])->dispatch();

// Run queue worker
php artisan queue:work
php artisan queue:work --queue=emails,default --tries=3
```

### Q17. Task Scheduling
```php
// app/Console/Kernel.php
protected function schedule(Schedule $schedule) {
    $schedule->command('emails:send')->daily();
    $schedule->call(function () {
        DB::table('recent')->delete();
    })->hourly();
}
```

### Q18. Form Requests
```php
class StorePostRequest extends FormRequest {
    public function rules() {
        return ['title' => 'required|max:255'];
    }
}
```

### Q19. Resource Controllers
```php
Route::resource('posts', PostController::class);
// Generates: index, create, store, show, edit, update, destroy
```

### Q20. API Resources
```php
class UserResource extends JsonResource {
    public function toArray($request) {
        return ['id' => $this->id, 'name' => $this->name];
    }
}
```

### Q21. Blade Components
```php
// Creating: php artisan make:component Alert
<x-alert type="error" :message="$message" />
```

### Q22. View Composers
```php
View::composer('*', function ($view) {
    $view->with('user', auth()->user());
});
```

### Q23. Middleware Groups
```php
protected $middlewareGroups = [
    'web' => [StartSession::class, VerifyCsrfToken::class],
    'api' => ['throttle:60,1', 'auth:sanctum']
];
```

### Q24. Rate Limiting
```php
RateLimiter::for('api', function (Request $request) {
    return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
});
```

### Q25. Database Transactions
```php
DB::transaction(function () {
    DB::update('update users set votes = 1');
    DB::delete('delete from posts');
});
```

### Q26. Eager Loading - WHY is it critical?
**Answer:** Eager loading solves the N+1 query problem, which can kill your application's performance. Without it, displaying 100 posts could execute 101+ database queries!

**The Problem (N+1 Queries):**
```php
// BAD: This executes 101 queries for 100 posts!
$posts = Post::all(); // 1 query

foreach ($posts as $post) {
    echo $post->author->name; // 1 query PER post = 100 queries!
}
// Total: 1 + 100 = 101 queries! 😱

// WHY this is bad:
// - Slow page load (multiple database round trips)
// - High database load
// - Poor user experience
// - Scalability issues
```

**The Solution (Eager Loading):**
```php
// GOOD: This executes only 2 queries for 100 posts!
$posts = Post::with('author')->get(); // 2 queries total
// Query 1: SELECT * FROM posts
// Query 2: SELECT * FROM users WHERE id IN (1,2,3,...)

foreach ($posts as $post) {
    echo $post->author->name; // No query! Data already loaded
}
// Total: 2 queries only! ⚡

// Real-world example: Blog post listing
public function index()
{
    // Load posts with author, category, and comment count
    $posts = Post::with(['author', 'category'])
        ->withCount('comments')
        ->latest()
        ->paginate(20);
    
    // In view:
    // @foreach($posts as $post)
    //     {{ $post->author->name }}  // No extra query!
    //     {{ $post->category->name }} // No extra query!
    //     {{ $post->comments_count }} comments
    // @endforeach
    
    return view('posts.index', compact('posts'));
}

// Nested eager loading
$posts = Post::with(['author.profile', 'comments.user'])->get();
// Loads posts, their authors, author profiles, comments, and comment users

// Conditional eager loading
$posts = Post::with(['author', 'comments' => function ($query) {
    $query->where('approved', true)
          ->orderBy('created_at', 'desc')
          ->limit(5);
}])->get();

// Lazy eager loading (when you forgot to eager load)
$posts = Post::all();
if ($someCondition) {
    $posts->load('author'); // Load afterwards
}

// Performance comparison:
// Without eager loading: 0.5s per post × 100 = 50 seconds!
// With eager loading: 0.1s total for all 100 posts!
```

### Q27. Query Scopes - WHY use them instead of repeating WHERE clauses?
**Answer:** Query scopes help you reuse common query logic, making your code DRY (Don't Repeat Yourself) and easier to maintain.

**The Problem (Code Duplication):**
```php
// BAD: Repeating the same logic everywhere
// In PostController
$posts = Post::where('status', 'published')
    ->where('published_at', '<=', now())
    ->get();

// In HomeController
$posts = Post::where('status', 'published')
    ->where('published_at', '<=', now())
    ->latest()
    ->take(5)
    ->get();

// In API Controller
$posts = Post::where('status', 'published')
    ->where('published_at', '<=', now())
    ->where('featured', true)
    ->get();

// PROBLEM:
// - Repeated code (violates DRY principle)
// - Hard to maintain (change logic in multiple places)
// - Error-prone (easy to forget conditions)
// - Less readable
```

**The Solution (Query Scopes):**
```php
// GOOD: Define once, use everywhere
// app/Models/Post.php
class Post extends Model
{
    // Local scope - reusable query logic
    public function scopePublished($query)
    {
        return $query->where('status', 'published')
                    ->where('published_at', '<=', now());
    }
    
    public function scopeFeatured($query)
    {
        return $query->where('featured', true);
    }
    
    public function scopeByAuthor($query, $authorId)
    {
        return $query->where('author_id', $authorId);
    }
    
    public function scopePopular($query, $threshold = 1000)
    {
        return $query->where('views', '>=', $threshold);
    }
    
    public function scopeRecent($query, $days = 7)
    {
        return $query->where('created_at', '>=', now()->subDays($days));
    }
}

// Now use everywhere - clean and simple!
// In PostController
$posts = Post::published()->get();

// In HomeController
$posts = Post::published()->latest()->take(5)->get();

// In API Controller
$posts = Post::published()->featured()->get();

// Chain multiple scopes
$posts = Post::published()
    ->featured()
    ->popular()
    ->recent()
    ->latest()
    ->paginate(15);

// With parameters
$posts = Post::published()
    ->byAuthor($authorId)
    ->popular(5000)
    ->get();

// Real-world example: E-commerce product filtering
class Product extends Model
{
    public function scopeActive($query)
    {
        return $query->where('status', 'active')
                    ->where('stock', '>', 0);
    }
    
    public function scopeInCategory($query, $categoryId)
    {
        return $query->where('category_id', $categoryId);
    }
    
    public function scopePriceRange($query, $min, $max)
    {
        return $query->whereBetween('price', [$min, $max]);
    }
    
    public function scopeOnSale($query)
    {
        return $query->where('sale_price', '<', DB::raw('regular_price'));
    }
    
    public function scopeHighRated($query, $minRating = 4.0)
    {
        return $query->where('average_rating', '>=', $minRating);
    }
}

// Usage: Build complex queries easily
$products = Product::active()
    ->inCategory($categoryId)
    ->priceRange(50, 500)
    ->highRated(4.5)
    ->orderBy('average_rating', 'desc')
    ->paginate(20);

// WHY this is better:
// ✅ Write once, use everywhere
// ✅ Easy to maintain - change in one place
// ✅ More readable code
// ✅ Chainable and flexible
// ✅ Testable
```

### Q28. Accessors & Mutators - WHY transform data automatically?
**Answer:** Accessors and Mutators allow you to format Eloquent attributes automatically when retrieving or setting them, without manually transforming data everywhere in your code.

**Why Use Them?**
- Automatic data formatting
- Consistent transformations across the app
- Cleaner controller code
- Encapsulation of business logic

**Real-time Example:**
```php
// PROBLEM: Without Accessors/Mutators - Manual transformations everywhere
class User extends Model {}

// In Controller 1
$user = User::find(1);
$fullName = $user->first_name . ' ' . $user->last_name; // Manual concat

// In Controller 2
$user = User::find(2);
$fullName = $user->first_name . ' ' . $user->last_name; // Repeated!

// In API
$users = User::all()->map(function($user) {
    return [
        'name' => $user->first_name . ' ' . $user->last_name, // Again!
        'email' => strtolower($user->email) // Manual transformation
    ];
});

// When saving password
$user->password = bcrypt($request->password); // Manual everywhere!

// SOLUTION: With Accessors & Mutators - Automatic transformations
class User extends Model
{
    // ACCESSOR: Get full name automatically
    public function getFullNameAttribute()
    {
        return "{$this->first_name} {$this->last_name}";
    }
    
    // ACCESSOR: Format email consistently
    public function getEmailAttribute($value)
    {
        return strtolower($value);
    }
    
    // ACCESSOR: Format phone number
    public function getPhoneAttribute($value)
    {
        if (!$value) return null;
        
        // Format: (123) 456-7890
        return sprintf('(%s) %s-%s',
            substr($value, 0, 3),
            substr($value, 3, 3),
            substr($value, 6)
        );
    }
    
    // ACCESSOR: Calculate age from birthday
    public function getAgeAttribute()
    {
        return $this->date_of_birth ? 
            $this->date_of_birth->age : null;
    }
    
    // ACCESSOR: Get profile image URL
    public function getProfileImageUrlAttribute()
    {
        if ($this->profile_image) {
            return Storage::url($this->profile_image);
        }
        return asset('images/default-avatar.png');
    }
    
    // MUTATOR: Auto-hash password
    public function setPasswordAttribute($value)
    {
        $this->attributes['password'] = bcrypt($value);
    }
    
    // MUTATOR: Clean and format phone
    public function setPhoneAttribute($value)
    {
        // Remove all non-numeric characters
        $cleaned = preg_replace('/[^0-9]/', '', $value);
        $this->attributes['phone'] = $cleaned;
    }
    
    // MUTATOR: Capitalize name
    public function setFirstNameAttribute($value)
    {
        $this->attributes['first_name'] = ucfirst(strtolower($value));
    }
    
    // MUTATOR: Auto-lowercase email
    public function setEmailAttribute($value)
    {
        $this->attributes['email'] = strtolower($value);
    }
}

// Now use everywhere - clean code!
// In Controller
$user = User::find(1);
echo $user->full_name;        // "John Doe" - automatic!
echo $user->email;             // "john@example.com" - lowercase!
echo $user->phone;             // "(123) 456-7890" - formatted!
echo $user->age;               // 30 - calculated!
echo $user->profile_image_url; // Full URL - automatic!

// When saving
$user = new User();
$user->first_name = 'JOHN';           // Stored as "John"
$user->email = 'John@Example.COM';    // Stored as "john@example.com"
$user->password = 'secret123';        // Stored as bcrypt hash
$user->phone = '(123) 456-7890';      // Stored as "1234567890"
$user->save();

// In API Response
return UserResource::collection(User::all());
// Every user automatically has:
// - full_name (concatenated)
// - formatted email (lowercase)
// - formatted phone
// - age (calculated)
// - profile_image_url (with full path)

// WHY this is better:
// ✅ No manual transformations needed
// ✅ Consistent data format everywhere
// ✅ Single source of truth
// ✅ Cleaner controller code
// ✅ Easier to maintain
// ✅ Automatic validation/formatting

// Advanced example: Money formatting
public function getPriceFormattedAttribute()
{
    return '$' . number_format($this->price, 2);
}

// Usage
$product->price;           // 99.99
$product->price_formatted; // "$99.99"
```

### Q29. Model Events
```php
class Post extends Model {
    protected static function booted() {
        static::creating(function ($post) {
            $post->slug = Str::slug($post->title);
        });
    }
}
```

### Q30. File Storage
```php
Storage::disk('s3')->put('file.jpg', $contents);
$url = Storage::url('file.jpg');
Storage::delete('file.jpg');
```

### Q31. Cache
```php
Cache::remember('users', 3600, function () {
    return DB::table('users')->get();
});
Cache::forget('users');
```

### Q32. Session
```php
session(['key' => 'value']);
$value = session('key');
session()->flash('message', 'Success!');
```

### Q33. Validation Rules
```php
$request->validate([
    'email' => 'required|email|unique:users',
    'age' => 'required|integer|min:18|max:65',
    'website' => 'nullable|url'
]);
```

### Q34. Custom Validation
```php
Validator::extend('uppercase', function ($attribute, $value) {
    return strtoupper($value) === $value;
});
```

### Q35. Pagination
```php
$users = DB::table('users')->paginate(15);
// In view: {{ $users->links() }}
```

### Q36. Soft Deletes
```php
class Post extends Model {
    use SoftDeletes;
}
Post::withTrashed()->get();
$post->restore();
```

### Q37. Database Seeding
```php
class DatabaseSeeder extends Seeder {
    public function run() {
        User::factory(50)->create();
    }
}
```

### Q38. Model Factories
```php
User::factory()->count(10)->create();
User::factory()->unverified()->create();
```

### Q39. Broadcasting
```php
class OrderShipped implements ShouldBroadcast {
    public function broadcastOn() {
        return new PrivateChannel('orders.'.$this->order->id);
    }
}
```

### Q40. Laravel Mix
```php
mix.js('resources/js/app.js', 'public/js')
   .sass('resources/sass/app.scss', 'public/css');
```

### Q41. Environment Configuration
```php
$debug = config('app.debug');
$url = env('APP_URL', 'http://localhost');
```

### Q42. Helpers
```php
$value = array_get($array, 'key', 'default');
$slug = str_slug('Laravel Framework');
$url = route('profile', ['id' => 1]);
```

### Q43. HTTP Client
```php
$response = Http::post('http://example.com/users', [
    'name' => 'Steve', 'role' => 'Network Administrator',
]);
```

### Q44. Mail
```php
Mail::to($user)->send(new OrderShipped($order));
```

### Q45. Artisan Commands
```php
// Make: php artisan make:command SendEmails
class SendEmails extends Command {
    protected $signature = 'email:send {user}';
    public function handle() {
        $this->info('Sending emails!');
    }
}
```

### Q46. Localization
```php
// lang/en/messages.php: ['welcome' => 'Welcome']
__('messages.welcome');
@lang('messages.welcome')
```

### Q47. Encryption
```php
$encrypted = Crypt::encryptString('Hello');
$decrypted = Crypt::decryptString($encrypted);
```

### Q48. Hashing
```php
$hashed = Hash::make('password');
Hash::check('password', $hashed);
```

### Q49. URL Generation
```php
url('/posts/' . $post->id);
route('posts.show', ['post' => $post]);
action([UserController::class, 'index']);
```

### Q50. Redirect
```php
return redirect('/home');
return redirect()->route('profile');
return redirect()->back()->withInput();
```

### Q51. Response Types
```php
return response()->json(['name' => 'John']);
return response()->download($pathToFile);
return response()->view('view', $data);
```

### Q52. Cookie
```php
Cookie::queue('name', 'value', $minutes);
$value = Cookie::get('name');
```

### Q53. Error Handling
```php
try {
    // Code
} catch (Exception $e) {
    report($e);
    return false;
}
```

### Q54. Logging
```php
Log::emergency($message);
Log::alert($message);
Log::critical($message);
Log::error($message);
```

### Q55. Testing
```php
public function test_example() {
    $response = $this->get('/');
    $response->assertStatus(200);
}
```

### Q56. Feature Tests
```php
$this->post('/posts', ['title' => 'Test']);
$this->assertDatabaseHas('posts', ['title' => 'Test']);
```

### Q57. Mocking
```php
Storage::fake('photos');
Mail::fake();
Event::fake();
```

### Q58. Database Testing
```php
use RefreshDatabase;
$this->seed();
```

### Q59. Middleware
```php
php artisan make:middleware CheckAge
public function handle($request, Closure $next) {
    if ($request->age <= 200) {
        return redirect('home');
    }
    return $next($request);
}
```

### Q60. CSRF Protection
```php
<form method="POST">
    @csrf
    <!-- Form fields -->
</form>
```

### Q61. Mass Assignment
```php
protected $fillable = ['name', 'email'];
protected $guarded = ['admin'];
```

### Q62. Relationships
```php
// One to One
public function phone() { return $this->hasOne(Phone::class); }
// One to Many
public function posts() { return $this->hasMany(Post::class); }
// Many to Many
public function roles() { return $this->belongsToMany(Role::class); }
```

### Q63. Polymorphic Relations
```php
public function commentable() {
    return $this->morphTo();
}
public function comments() {
    return $this->morphMany(Comment::class, 'commentable');
}
```

### Q64. Query Builder
```php
DB::table('users')->where('votes', '>', 100)->get();
DB::table('users')->orderBy('name')->get();
DB::table('users')->join('contacts', 'users.id', '=', 'contacts.user_id')->get();
```

### Q65. Raw Expressions
```php
$users = DB::table('users')
    ->select(DB::raw('count(*) as user_count, status'))
    ->groupBy('status')
    ->get();
```

### Q66. Chunk Results
```php
DB::table('users')->orderBy('id')->chunk(100, function ($users) {
    foreach ($users as $user) { //
    }
});
```

### Q67. Exists/Doesn't Exist
```php
if (User::where('email', $email)->exists()) { }
if (User::where('email', $email)->doesntExist()) { }
```

### Q68. First or Create
```php
$user = User::firstOrCreate(['email' => $email]);
$user = User::firstOrNew(['email' => $email]);
```

### Q69. Update or Create
```php
$flight = Flight::updateOrCreate(
    ['departure' => 'Oakland', 'destination' => 'San Diego'],
    ['price' => 99, 'discounted' => 1]
);
```

### Q70. Increment/Decrement
```php
Post::where('id', 1)->increment('views');
Post::where('id', 1)->decrement('likes', 5);
```

### Q71. Ordering
```php
$users = User::orderBy('name', 'desc')->get();
$users = User::latest()->get();
$users = User::oldest()->get();
```

### Q72. Grouping
```php
$users = User::groupBy('account_id')->having('account_id', '>', 100)->get();
```

### Q73. Conditional Clauses
```php
$users = User::when($role, function ($query, $role) {
    return $query->where('role', $role);
})->get();
```

### Q74. Subquery Selects
```php
$users = User::addSelect(['last_post' => Post::select('created_at')
    ->whereColumn('user_id', 'users.id')
    ->latest()
    ->limit(1)
])->get();
```

### Q75. Advanced Where
```php
$users = User::where([
    ['status', '=', '1'],
    ['subscribed', '<>', '1'],
])->get();
```

### Q76. Where Between
```php
$users = User::whereBetween('votes', [1, 100])->get();
$users = User::whereNotBetween('votes', [1, 100])->get();
```

### Q77. Where In
```php
$users = User::whereIn('id', [1, 2, 3])->get();
$users = User::whereNotIn('id', [1, 2, 3])->get();
```

### Q78. Where Null
```php
$users = User::whereNull('updated_at')->get();
$users = User::whereNotNull('updated_at')->get();
```

### Q79. Where Date
```php
$users = User::whereDate('created_at', '2025-01-01')->get();
$users = User::whereMonth('created_at', '12')->get();
$users = User::whereYear('created_at', '2025')->get();
```

### Q80. Where Column
```php
$users = User::whereColumn('first_name', 'last_name')->get();
$users = User::whereColumn('updated_at', '>', 'created_at')->get();
```

---

## Service Container & Service Providers

### Q81. What is Laravel's Service Container and how does it work?
**Answer:** The Service Container is Laravel's dependency injection container that manages class dependencies and performs dependency injection.

**Real-time Example:**
```php
// app/Services/PaymentService.php
namespace App\Services;

interface PaymentServiceInterface
{
    public function processPayment($amount, $cardToken);
}

class StripePaymentService implements PaymentServiceInterface
{
    public function processPayment($amount, $cardToken)
    {
        // Stripe payment logic
        return ['status' => 'success', 'transaction_id' => 'stripe_123'];
    }
}

class PayPalPaymentService implements PaymentServiceInterface
{
    public function processPayment($amount, $cardToken)
    {
        // PayPal payment logic
        return ['status' => 'success', 'transaction_id' => 'paypal_456'];
    }
}

// app/Providers/AppServiceProvider.php
namespace App\Providers;

use App\Services\PaymentServiceInterface;
use App\Services\StripePaymentService;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register()
    {
        // Bind interface to implementation
        $this->app->bind(PaymentServiceInterface::class, function ($app) {
            $paymentMethod = config('payment.default_method');
            
            switch ($paymentMethod) {
                case 'stripe':
                    return new StripePaymentService();
                case 'paypal':
                    return new PayPalPaymentService();
                default:
                    return new StripePaymentService();
            }
        });
        
        // Singleton binding
        $this->app->singleton('payment.manager', function ($app) {
            return new PaymentManager($app->make(PaymentServiceInterface::class));
        });
    }
}

// Usage in Controller
class OrderController extends Controller
{
    protected $paymentService;
    
    public function __construct(PaymentServiceInterface $paymentService)
    {
        $this->paymentService = $paymentService;
    }
    
    public function processOrder(Request $request)
    {
        $result = $this->paymentService->processPayment(
            $request->amount,
            $request->card_token
        );
        
        return response()->json($result);
    }
}

// Manual resolution
$paymentService = app(PaymentServiceInterface::class);
$paymentManager = app('payment.manager');

// Contextual binding
$this->app->when(OrderController::class)
    ->needs(PaymentServiceInterface::class)
    ->give(StripePaymentService::class);
```

### Q82. What are Service Providers and how to create custom ones?
**Answer:** Service Providers are the central place to configure your application. They register services with the container and configure application components.

**Real-time Example:**
```php
// Creating Service Provider
php artisan make:provider CustomServiceProvider

// app/Providers/CustomServiceProvider.php
namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use App\Services\EmailService;
use App\Services\SmsService;
use App\Services\NotificationService;

class CustomServiceProvider extends ServiceProvider
{
    public function register()
    {
        // Register services
        $this->app->singleton(EmailService::class, function ($app) {
            return new EmailService(
                $app->make('mailer'),
                config('mail.from.address')
            );
        });
        
        $this->app->singleton(SmsService::class, function ($app) {
            return new SmsService(
                config('sms.api_key'),
                config('sms.sender_id')
            );
        });
        
        // Register notification service
        $this->app->singleton(NotificationService::class, function ($app) {
            return new NotificationService(
                $app->make(EmailService::class),
                $app->make(SmsService::class)
            );
        });
        
        // Register custom configuration
        $this->app->configure('custom');
    }
    
    public function boot()
    {
        // Boot services
        $this->loadViewsFrom(__DIR__.'/../resources/views', 'custom');
        $this->loadMigrationsFrom(__DIR__.'/../database/migrations');
        $this->loadTranslationsFrom(__DIR__.'/../resources/lang', 'custom');
        
        // Publish assets
        $this->publishes([
            __DIR__.'/../config/custom.php' => config_path('custom.php'),
        ], 'config');
        
        $this->publishes([
            __DIR__.'/../resources/views' => resource_path('views/vendor/custom'),
        ], 'views');
        
        // Register routes
        $this->loadRoutesFrom(__DIR__.'/../routes/custom.php');
        
        // Register middleware
        $this->app['router']->aliasMiddleware('custom', \App\Http\Middleware\CustomMiddleware::class);
    }
}

// Custom Service Classes
class EmailService
{
    protected $mailer;
    protected $fromEmail;
    
    public function __construct($mailer, $fromEmail)
    {
        $this->mailer = $mailer;
        $this->fromEmail = $fromEmail;
    }
    
    public function send($to, $subject, $body)
    {
        return $this->mailer->to($to)->send(new CustomMail($subject, $body));
    }
}

class NotificationService
{
    protected $emailService;
    protected $smsService;
    
    public function __construct(EmailService $emailService, SmsService $smsService)
    {
        $this->emailService = $emailService;
        $this->smsService = $smsService;
    }
    
    public function sendNotification($user, $message, $channels = ['email'])
    {
        $notifications = [];
        
        if (in_array('email', $channels)) {
            $notifications[] = $this->emailService->send($user->email, 'Notification', $message);
        }
        
        if (in_array('sms', $channels)) {
            $notifications[] = $this->smsService->send($user->phone, $message);
        }
        
        return $notifications;
    }
}
```

---

## Queues & Jobs

### Q83. Laravel Queues & Jobs - WHY and HOW?
**Answer:** Queues allow you to defer time-consuming tasks (like sending emails, processing images, or calling external APIs) to be processed later, making your application feel faster to users.

**Why Use Queues?**
- Don't make users wait for slow operations
- Handle tasks in background
- Retry failed operations automatically
- Process tasks during off-peak hours

**Real-time Example:**
```php
// PROBLEM: Without Queue - User waits 5+ seconds
public function register(Request $request)
{
    $user = User::create($request->all());
    
    // These slow operations block the response
    Mail::to($user)->send(new WelcomeEmail($user));        // 2 seconds
    $this->resizeProfileImage($user->image);               // 2 seconds
    $this->notifyAdmins($user);                            // 1 second
    $this->updateAnalytics($user);                         // 1 second
    
    return redirect('/dashboard'); // User waits 6 seconds!
}

// SOLUTION: With Queue - User waits <1 second
public function register(Request $request)
{
    $user = User::create($request->all());
    
    // Queue these operations - happens in background
    SendWelcomeEmail::dispatch($user);
    ResizeImage::dispatch($user->image);
    NotifyAdmins::dispatch($user);
    UpdateAnalytics::dispatch($user);
    
    return redirect('/dashboard'); // User waits <1 second!
}

// Creating the Job
php artisan make:job SendWelcomeEmail

// app/Jobs/SendWelcomeEmail.php
namespace App\Jobs;

use App\Models\User;
use App\Mail\WelcomeEmail;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Illuminate\Support\Facades\Mail;

class SendWelcomeEmail implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
    
    public $user;
    public $tries = 3; // Retry 3 times if failed
    public $timeout = 120; // Timeout after 2 minutes
    
    public function __construct(User $user)
    {
        $this->user = $user;
    }
    
    public function handle()
    {
        // This runs in background worker
        Mail::to($this->user->email)
            ->send(new WelcomeEmail($this->user));
    }
    
    // If job fails after all retries
    public function failed(\Throwable $exception)
    {
        // Send notification to admin
        // Log the error
        \Log::error('Failed to send welcome email', [
            'user_id' => $this->user->id,
            'error' => $exception->getMessage()
        ]);
    }
}

// Dispatching with delay
SendWelcomeEmail::dispatch($user)->delay(now()->addMinutes(10));

// Dispatching to specific queue
SendWelcomeEmail::dispatch($user)->onQueue('emails');

// Chain jobs - execute in sequence
Bus::chain([
    new ProcessOrder($order),
    new ChargeCustomer($order),
    new SendReceipt($order),
])->dispatch();

// Run queue worker
php artisan queue:work
php artisan queue:work --queue=emails,default --tries=3
```

### Q84. Task Scheduling
```php
// app/Console/Kernel.php
protected function schedule(Schedule $schedule) {
    $schedule->command('emails:send')->daily();
    $schedule->call(function () {
        DB::table('recent')->delete();
    })->hourly();
}
```

---

## Testing

### Q85. Laravel Testing - Unit, Feature, and Browser Tests
**Answer:** Laravel provides comprehensive testing tools for unit, feature, and browser testing.

**Real-time Example:**
```php
// Unit Test Example
// tests/Unit/UserTest.php
namespace Tests\Unit;

use Tests\TestCase;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;

class UserTest extends TestCase
{
    use RefreshDatabase;
    
    public function test_user_can_be_created()
    {
        $user = User::factory()->create([
            'name' => 'John Doe',
            'email' => 'john@example.com'
        ]);
        
        $this->assertInstanceOf(User::class, $user);
        $this->assertEquals('John Doe', $user->name);
        $this->assertEquals('john@example.com', $user->email);
    }
    
    public function test_user_full_name_accessor()
    {
        $user = User::factory()->create([
            'first_name' => 'John',
            'last_name' => 'Doe'
        ]);
        
        $this->assertEquals('John Doe', $user->full_name);
    }
    
    public function test_user_can_have_posts()
    {
        $user = User::factory()->create();
        $post = $user->posts()->create([
            'title' => 'Test Post',
            'content' => 'Test Content'
        ]);
        
        $this->assertTrue($user->posts->contains($post));
        $this->assertEquals(1, $user->posts->count());
    }
}

// Feature Test Example
// tests/Feature/UserRegistrationTest.php
namespace Tests\Feature;

use Tests\TestCase;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Support\Facades\Hash;

class UserRegistrationTest extends TestCase
{
    use RefreshDatabase;
    
    public function test_user_can_register()
    {
        $userData = [
            'name' => 'John Doe',
            'email' => 'john@example.com',
            'password' => 'password123',
            'password_confirmation' => 'password123'
        ];
        
        $response = $this->post('/register', $userData);
        
        $response->assertRedirect('/dashboard');
        $this->assertDatabaseHas('users', [
            'email' => 'john@example.com'
        ]);
    }
    
    public function test_user_registration_requires_valid_email()
    {
        $userData = [
            'name' => 'John Doe',
            'email' => 'invalid-email',
            'password' => 'password123',
            'password_confirmation' => 'password123'
        ];
        
        $response = $this->post('/register', $userData);
        
        $response->assertSessionHasErrors('email');
        $this->assertDatabaseMissing('users', [
            'email' => 'invalid-email'
        ]);
    }
    
    public function test_authenticated_user_can_access_dashboard()
    {
        $user = User::factory()->create();
        
        $response = $this->actingAs($user)->get('/dashboard');
        
        $response->assertStatus(200);
        $response->assertSee('Dashboard');
    }
}

// Browser Test Example (Laravel Dusk)
// tests/Browser/UserRegistrationTest.php
namespace Tests\Browser;

use Tests\DuskTestCase;
use Laravel\Dusk\Browser;
use App\Models\User;

class UserRegistrationTest extends DuskTestCase
{
    public function test_user_can_register_via_browser()
    {
        $this->browse(function (Browser $browser) {
            $browser->visit('/register')
                    ->type('name', 'John Doe')
                    ->type('email', 'john@example.com')
                    ->type('password', 'password123')
                    ->type('password_confirmation', 'password123')
                    ->press('Register')
                    ->assertPathIs('/dashboard')
                    ->assertSee('Welcome, John Doe!');
        });
    }
    
    public function test_user_can_login_via_browser()
    {
        $user = User::factory()->create([
            'email' => 'john@example.com',
            'password' => bcrypt('password123')
        ]);
        
        $this->browse(function (Browser $browser) use ($user) {
            $browser->visit('/login')
                    ->type('email', 'john@example.com')
                    ->type('password', 'password123')
                    ->press('Login')
                    ->assertPathIs('/dashboard')
                    ->assertSee('Dashboard');
        });
    }
}

// API Test Example
// tests/Feature/ApiUserTest.php
namespace Tests\Feature;

use Tests\TestCase;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;

class ApiUserTest extends TestCase
{
    use RefreshDatabase;
    
    public function test_can_get_users_via_api()
    {
        $users = User::factory()->count(3)->create();
        
        $response = $this->getJson('/api/users');
        
        $response->assertStatus(200)
                ->assertJsonCount(3, 'data')
                ->assertJsonStructure([
                    'data' => [
                        '*' => ['id', 'name', 'email', 'created_at']
                    ]
                ]);
    }
    
    public function test_can_create_user_via_api()
    {
        $userData = [
            'name' => 'John Doe',
            'email' => 'john@example.com',
            'password' => 'password123'
        ];
        
        $response = $this->postJson('/api/users', $userData);
        
        $response->assertStatus(201)
                ->assertJsonFragment(['name' => 'John Doe'])
                ->assertJsonStructure(['data' => ['id', 'name', 'email']]);
    }
    
    public function test_api_requires_authentication()
    {
        $response = $this->getJson('/api/protected-route');
        
        $response->assertStatus(401);
    }
}

// Test Database Setup
// tests/TestCase.php
namespace Tests;

use Illuminate\Foundation\Testing\TestCase as BaseTestCase;
use Illuminate\Foundation\Testing\RefreshDatabase;

abstract class TestCase extends BaseTestCase
{
    use CreatesApplication, RefreshDatabase;
    
    protected function setUp(): void
    {
        parent::setUp();
        
        // Run migrations
        $this->artisan('migrate');
        
        // Seed test data
        $this->artisan('db:seed');
    }
}

// Running Tests
// php artisan test
// php artisan test --filter=UserTest
// php artisan test tests/Feature/UserRegistrationTest.php
// php artisan dusk
```

---

## Performance & Optimization

### Q86. Laravel Performance Optimization Techniques
**Answer:** Laravel provides various techniques to optimize application performance including caching, database optimization, and code improvements.

**Real-time Example:**
```php
// Database Optimization
class OptimizedPostController extends Controller
{
    public function index()
    {
        // BAD: N+1 Query Problem
        // $posts = Post::all(); // 1 query
        // foreach ($posts as $post) {
        //     echo $post->author->name; // 1 query per post!
        // }
        
        // GOOD: Eager Loading
        $posts = Post::with(['author', 'category', 'tags'])
            ->withCount('comments')
            ->published()
            ->latest()
            ->paginate(15);
        
        return view('posts.index', compact('posts'));
    }
    
    public function show(Post $post)
    {
        // Load relationships efficiently
        $post->load(['author.profile', 'comments.user', 'tags']);
        
        return view('posts.show', compact('post'));
    }
}

// Caching Strategies
class CacheService
{
    public function getPopularPosts()
    {
        return Cache::remember('popular_posts', 3600, function () {
            return Post::with('author')
                ->published()
                ->orderBy('views', 'desc')
                ->limit(10)
                ->get();
        });
    }
    
    public function getUserPosts($userId)
    {
        $cacheKey = "user_posts_{$userId}";
        
        return Cache::tags(['posts', "user_{$userId}"])
            ->remember($cacheKey, 1800, function () use ($userId) {
                return Post::where('user_id', $userId)
                    ->with('category')
                    ->latest()
                    ->get();
            });
    }
    
    public function clearUserCache($userId)
    {
        Cache::tags(["user_{$userId}"])->flush();
    }
}

// Query Optimization
class OptimizedQueries
{
    public function getUsersWithPosts()
    {
        // Use select to limit columns
        return User::select(['id', 'name', 'email'])
            ->with(['posts' => function ($query) {
                $query->select(['id', 'title', 'user_id', 'created_at'])
                      ->published()
                      ->latest();
            }])
            ->whereHas('posts')
            ->get();
    }
    
    public function getPostsWithStats()
    {
        // Use subqueries for counts
        return Post::addSelect([
            'comments_count' => Comment::selectRaw('count(*)')
                ->whereColumn('post_id', 'posts.id'),
            'likes_count' => Like::selectRaw('count(*)')
                ->whereColumn('post_id', 'posts.id')
        ])->get();
    }
    
    public function getPostsByCategory($categoryId)
    {
        // Use database indexes
        return Post::where('category_id', $categoryId)
            ->where('published_at', '<=', now())
            ->orderBy('published_at', 'desc')
            ->limit(20)
            ->get();
    }
}

// Memory Optimization
class MemoryOptimizedController extends Controller
{
    public function exportUsers()
    {
        // Process in chunks to avoid memory issues
        $users = User::chunk(100, function ($users) {
            foreach ($users as $user) {
                // Process each user
                $this->processUser($user);
            }
        });
    }
    
    public function processLargeDataset()
    {
        // Use cursor for large datasets
        User::where('created_at', '>', now()->subDays(30))
            ->cursor()
            ->each(function ($user) {
                $this->processUser($user);
            });
    }
}

// Application Performance Monitoring
class PerformanceMiddleware
{
    public function handle($request, Closure $next)
    {
        $startTime = microtime(true);
        $startMemory = memory_get_usage();
        
        $response = $next($request);
        
        $endTime = microtime(true);
        $endMemory = memory_get_usage();
        
        $duration = $endTime - $startTime;
        $memoryUsed = $endMemory - $startMemory;
        
        // Log performance metrics
        if ($duration > 1.0) { // Log slow requests
            \Log::warning('Slow request detected', [
                'url' => $request->fullUrl(),
                'duration' => $duration,
                'memory' => $memoryUsed,
                'user_id' => auth()->id()
            ]);
        }
        
        return $response;
    }
}

// Database Indexing
// Migration for indexes
public function up()
{
    Schema::table('posts', function (Blueprint $table) {
        $table->index(['published_at', 'category_id']);
        $table->index(['user_id', 'created_at']);
        $table->fullText(['title', 'content']);
    });
}

// Redis Caching
class RedisCacheService
{
    public function cacheUserData($userId)
    {
        $user = User::with(['profile', 'roles'])->find($userId);
        
        Redis::setex("user:{$userId}", 3600, json_encode($user));
        
        return $user;
    }
    
    public function getCachedUser($userId)
    {
        $cached = Redis::get("user:{$userId}");
        
        if ($cached) {
            return json_decode($cached, true);
        }
        
        return $this->cacheUserData($userId);
    }
}

// Queue Optimization
class OptimizedJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
    
    public $timeout = 300;
    public $tries = 3;
    public $maxExceptions = 3;
    
    public function handle()
    {
        // Process in chunks
        $this->processInChunks();
    }
    
    private function processInChunks()
    {
        $chunkSize = 100;
        
        User::chunk($chunkSize, function ($users) {
            foreach ($users as $user) {
                $this->processUser($user);
            }
        });
    }
}
```

---

## Advanced Laravel Concepts

### Q81. What is Laravel's Service Container and how does it work?
**Answer:** The Service Container is Laravel's dependency injection container that manages class dependencies and performs dependency injection.

**Real-time Example:**
```php
// app/Services/PaymentService.php
namespace App\Services;

interface PaymentServiceInterface
{
    public function processPayment($amount, $cardToken);
}

class StripePaymentService implements PaymentServiceInterface
{
    public function processPayment($amount, $cardToken)
    {
        // Stripe payment logic
        return ['status' => 'success', 'transaction_id' => 'stripe_123'];
    }
}

class PayPalPaymentService implements PaymentServiceInterface
{
    public function processPayment($amount, $cardToken)
    {
        // PayPal payment logic
        return ['status' => 'success', 'transaction_id' => 'paypal_456'];
    }
}

// app/Providers/AppServiceProvider.php
namespace App\Providers;

use App\Services\PaymentServiceInterface;
use App\Services\StripePaymentService;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register()
    {
        // Bind interface to implementation
        $this->app->bind(PaymentServiceInterface::class, function ($app) {
            $paymentMethod = config('payment.default_method');
            
            switch ($paymentMethod) {
                case 'stripe':
                    return new StripePaymentService();
                case 'paypal':
                    return new PayPalPaymentService();
                default:
                    return new StripePaymentService();
            }
        });
        
        // Singleton binding
        $this->app->singleton('payment.manager', function ($app) {
            return new PaymentManager($app->make(PaymentServiceInterface::class));
        });
    }
}

// Usage in Controller
class OrderController extends Controller
{
    protected $paymentService;
    
    public function __construct(PaymentServiceInterface $paymentService)
    {
        $this->paymentService = $paymentService;
    }
    
    public function processOrder(Request $request)
    {
        $result = $this->paymentService->processPayment(
            $request->amount,
            $request->card_token
        );
        
        return response()->json($result);
    }
}

// Manual resolution
$paymentService = app(PaymentServiceInterface::class);
$paymentManager = app('payment.manager');

// Contextual binding
$this->app->when(OrderController::class)
    ->needs(PaymentServiceInterface::class)
    ->give(StripePaymentService::class);
```

### Q82. Explain Laravel's Event System with real-world examples
**Answer:** Laravel's event system provides a simple observer pattern implementation for decoupling application components.

**Real-time Example:**
```php
// app/Events/OrderPlaced.php
namespace App\Events;

use App\Models\Order;
use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderPlaced implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public $order;
    public $user;

    public function __construct(Order $order)
    {
        $this->order = $order;
        $this->user = $order->user;
    }

    public function broadcastOn()
    {
        return [
            new PrivateChannel('orders.' . $this->order->id),
            new Channel('orders')
        ];
    }

    public function broadcastWith()
    {
        return [
            'order_id' => $this->order->id,
            'user_name' => $this->user->name,
            'total' => $this->order->total,
            'status' => $this->order->status
        ];
    }
}

// app/Listeners/SendOrderConfirmation.php
namespace App\Listeners;

use App\Events\OrderPlaced;
use App\Mail\OrderConfirmationMail;
use App\Notifications\OrderPlacedNotification;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Support\Facades\Mail;
use Illuminate\Support\Facades\Notification;

class SendOrderConfirmation implements ShouldQueue
{
    use InteractsWithQueue;

    public $tries = 3;
    public $timeout = 120;

    public function handle(OrderPlaced $event)
    {
        $order = $event->order;
        $user = $event->user;

        // Send email
        Mail::to($user->email)->send(new OrderConfirmationMail($order));

        // Send notification
        $user->notify(new OrderPlacedNotification($order));

        // Send SMS (if configured)
        if (config('sms.enabled')) {
            $this->sendSMS($user->phone, "Order #{$order->id} placed successfully!");
        }
    }

    public function failed(OrderPlaced $event, $exception)
    {
        // Log failure
        \Log::error('Failed to send order confirmation', [
            'order_id' => $event->order->id,
            'error' => $exception->getMessage()
        ]);

        // Send to admin
        Mail::to(config('mail.admin'))->send(new OrderConfirmationFailedMail($event->order));
    }

    private function sendSMS($phone, $message)
    {
        // SMS sending logic
    }
}

// app/Listeners/UpdateInventory.php
namespace App\Listeners;

use App\Events\OrderPlaced;
use App\Models\Product;
use Illuminate\Contracts\Queue\ShouldQueue;

class UpdateInventory implements ShouldQueue
{
    public function handle(OrderPlaced $event)
    {
        $order = $event->order;

        foreach ($order->items as $item) {
            Product::where('id', $item->product_id)
                ->decrement('stock', $item->quantity);
        }
    }
}

// app/Listeners/GenerateInvoice.php
namespace App\Listeners;

use App\Events\OrderPlaced;
use App\Services\InvoiceService;
use Illuminate\Contracts\Queue\ShouldQueue;

class GenerateInvoice implements ShouldQueue
{
    protected $invoiceService;

    public function __construct(InvoiceService $invoiceService)
    {
        $this->invoiceService = $invoiceService;
    }

    public function handle(OrderPlaced $event)
    {
        $this->invoiceService->generate($event->order);
    }
}

// EventServiceProvider registration
namespace App\Providers;

use App\Events\OrderPlaced;
use App\Listeners\SendOrderConfirmation;
use App\Listeners\UpdateInventory;
use App\Listeners\GenerateInvoice;
use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

class EventServiceProvider extends ServiceProvider
{
    protected $listen = [
        OrderPlaced::class => [
            SendOrderConfirmation::class,
            UpdateInventory::class,
            GenerateInvoice::class,
        ],
    ];

    protected $subscribe = [
        OrderEventSubscriber::class,
    ];
}

// Usage in Controller
class OrderController extends Controller
{
    public function store(Request $request)
    {
        $order = Order::create($request->validated());

        // Dispatch event
        event(new OrderPlaced($order));

        return response()->json(['order' => $order]);
    }
}

// Event Subscriber
namespace App\Listeners;

use App\Events\OrderPlaced;
use App\Events\OrderCancelled;
use Illuminate\Events\Dispatcher;

class OrderEventSubscriber
{
    public function handleOrderPlaced($event)
    {
        // Handle order placed
    }

    public function handleOrderCancelled($event)
    {
        // Handle order cancelled
    }

    public function subscribe(Dispatcher $events)
    {
        $events->listen(OrderPlaced::class, [OrderEventSubscriber::class, 'handleOrderPlaced']);
        $events->listen(OrderCancelled::class, [OrderEventSubscriber::class, 'handleOrderCancelled']);
    }
}
```

### Q83. What are Laravel Observers and when to use them?
**Answer:** Observers are classes that listen to various Eloquent model events and allow you to organize event handling logic.

**Real-time Example:**
```php
// app/Observers/UserObserver.php
namespace App\Observers;

use App\Models\User;
use App\Services\EmailService;
use App\Services\AnalyticsService;
use Illuminate\Support\Facades\Cache;

class UserObserver
{
    protected $emailService;
    protected $analyticsService;

    public function __construct(EmailService $emailService, AnalyticsService $analyticsService)
    {
        $this->emailService = $emailService;
        $this->analyticsService = $analyticsService;
    }

    public function creating(User $user)
    {
        // Generate username if not provided
        if (empty($user->username)) {
            $user->username = $this->generateUsername($user->name);
        }

        // Hash password
        if (!empty($user->password)) {
            $user->password = bcrypt($user->password);
        }

        // Set default role
        if (empty($user->role)) {
            $user->role = 'user';
        }
    }

    public function created(User $user)
    {
        // Send welcome email
        $this->emailService->sendWelcomeEmail($user);

        // Track user registration
        $this->analyticsService->track('user_registered', [
            'user_id' => $user->id,
            'registration_source' => request()->header('Referer')
        ]);

        // Create user profile
        $user->profile()->create([
            'bio' => '',
            'avatar' => null,
            'settings' => json_encode(['theme' => 'light'])
        ]);

        // Clear cache
        Cache::forget('users_count');
    }

    public function updating(User $user)
    {
        // Log email changes
        if ($user->isDirty('email')) {
            \Log::info('User email changed', [
                'user_id' => $user->id,
                'old_email' => $user->getOriginal('email'),
                'new_email' => $user->email
            ]);
        }

        // Update username if name changed
        if ($user->isDirty('name')) {
            $user->username = $this->generateUsername($user->name);
        }
    }

    public function updated(User $user)
    {
        // Send email change notification
        if ($user->wasChanged('email')) {
            $this->emailService->sendEmailChangeNotification($user);
        }

        // Update analytics
        $this->analyticsService->track('user_updated', [
            'user_id' => $user->id,
            'changed_fields' => $user->getDirty()
        ]);
    }

    public function deleting(User $user)
    {
        // Check if user has orders
        if ($user->orders()->count() > 0) {
            throw new \Exception('Cannot delete user with existing orders');
        }

        // Archive user data
        $this->archiveUserData($user);
    }

    public function deleted(User $user)
    {
        // Send deletion confirmation
        $this->emailService->sendAccountDeletionConfirmation($user);

        // Track deletion
        $this->analyticsService->track('user_deleted', [
            'user_id' => $user->id,
            'deletion_reason' => request()->input('deletion_reason')
        ]);

        // Clear all user-related cache
        Cache::tags(['user_' . $user->id])->flush();
    }

    private function generateUsername($name)
    {
        $base = strtolower(str_replace(' ', '', $name));
        $username = $base;
        $counter = 1;

        while (User::where('username', $username)->exists()) {
            $username = $base . $counter;
            $counter++;
        }

        return $username;
    }

    private function archiveUserData($user)
    {
        // Archive user data to separate table
        \DB::table('archived_users')->insert([
            'original_id' => $user->id,
            'name' => $user->name,
            'email' => $user->email,
            'archived_at' => now(),
            'archived_by' => auth()->id()
        ]);
    }
}

// Register observer in AppServiceProvider
namespace App\Providers;

use App\Models\User;
use App\Observers\UserObserver;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot()
    {
        User::observe(UserObserver::class);
    }
}

// Usage
$user = User::create([
    'name' => 'John Doe',
    'email' => 'john@example.com',
    'password' => 'secret123'
]);
// Automatically triggers: creating, created events

$user->update(['email' => 'newemail@example.com']);
// Automatically triggers: updating, updated events

$user->delete();
// Automatically triggers: deleting, deleted events
```

### Q84. Explain Laravel's Broadcasting system
**Answer:** Broadcasting allows you to broadcast events to your frontend in real-time using WebSockets.

**Real-time Example:**
```php
// app/Events/MessageSent.php
namespace App\Events;

use App\Models\Message;
use App\Models\User;
use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class MessageSent implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public $message;
    public $user;

    public function __construct(Message $message, User $user)
    {
        $this->message = $message;
        $this->user = $user;
    }

    public function broadcastOn()
    {
        return [
            new PrivateChannel('chat.' . $this->message->chat_id),
            new Channel('messages')
        ];
    }

    public function broadcastWith()
    {
        return [
            'id' => $this->message->id,
            'content' => $this->message->content,
            'user' => [
                'id' => $this->user->id,
                'name' => $this->user->name,
                'avatar' => $this->user->avatar
            ],
            'timestamp' => $this->message->created_at->toISOString()
        ];
    }

    public function broadcastAs()
    {
        return 'message.sent';
    }
}

// app/Events/UserTyping.php
namespace App\Events;

use App\Models\User;
use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class UserTyping implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public $user;
    public $chatId;

    public function __construct(User $user, $chatId)
    {
        $this->user = $user;
        $this->chatId = $chatId;
    }

    public function broadcastOn()
    {
        return new PrivateChannel('chat.' . $this->chatId);
    }

    public function broadcastWith()
    {
        return [
            'user' => [
                'id' => $this->user->id,
                'name' => $this->user->name
            ],
            'typing' => true
        ];
    }

    public function broadcastAs()
    {
        return 'user.typing';
    }
}

// routes/channels.php
use Illuminate\Support\Facades\Broadcast;

Broadcast::channel('chat.{chatId}', function ($user, $chatId) {
    return $user->chats()->where('chat_id', $chatId)->exists();
});

Broadcast::channel('App.User.{userId}', function ($user, $userId) {
    return (int) $user->id === (int) $userId;
});

// Controller usage
class ChatController extends Controller
{
    public function sendMessage(Request $request)
    {
        $message = Message::create([
            'chat_id' => $request->chat_id,
            'user_id' => auth()->id(),
            'content' => $request->content
        ]);

        // Broadcast message
        broadcast(new MessageSent($message, auth()->user()));

        return response()->json(['message' => $message]);
    }

    public function startTyping(Request $request)
    {
        broadcast(new UserTyping(auth()->user(), $request->chat_id));
        
        return response()->json(['status' => 'typing']);
    }
}

// Frontend JavaScript (using Laravel Echo)
import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: process.env.MIX_PUSHER_APP_KEY,
    cluster: process.env.MIX_PUSHER_APP_CLUSTER,
    forceTLS: true,
    authEndpoint: '/broadcasting/auth'
});

// Listen to private channel
Echo.private('chat.1')
    .listen('message.sent', (e) => {
        console.log('New message:', e);
        // Add message to chat UI
    })
    .listen('user.typing', (e) => {
        console.log('User typing:', e);
        // Show typing indicator
    });

// Listen to presence channel
Echo.join('chat.1')
    .here((users) => {
        console.log('Users currently in chat:', users);
    })
    .joining((user) => {
        console.log('User joined:', user);
    })
    .leaving((user) => {
        console.log('User left:', user);
    });
```

### Q85. What is Laravel's Task Scheduling and how to use it?
**Answer:** Laravel's task scheduler allows you to define scheduled tasks within your application.

**Real-time Example:**
```php
// app/Console/Kernel.php
namespace App\Console;

use Illuminate\Console\Scheduling\Schedule;
use Illuminate\Foundation\Console\Kernel as ConsoleKernel;

class Kernel extends ConsoleKernel
{
    protected $commands = [
        Commands\SendDailyReports::class,
        Commands\CleanupExpiredTokens::class,
        Commands\GenerateSitemap::class,
    ];

    protected function schedule(Schedule $schedule)
    {
        // Daily reports at 9 AM
        $schedule->command('reports:daily')
            ->dailyAt('09:00')
            ->timezone('America/New_York')
            ->emailOutputOnFailure('admin@example.com');

        // Cleanup expired tokens every hour
        $schedule->command('tokens:cleanup')
            ->hourly()
            ->withoutOverlapping()
            ->runInBackground();

        // Generate sitemap daily
        $schedule->command('sitemap:generate')
            ->daily()
            ->at('02:00');

        // Send email reminders
        $schedule->call(function () {
            $this->sendEmailReminders();
        })->dailyAt('10:00');

        // Database backup
        $schedule->command('backup:run')
            ->daily()
            ->at('03:00')
            ->onSuccess(function () {
                \Log::info('Database backup completed successfully');
            })
            ->onFailure(function () {
                \Log::error('Database backup failed');
            });

        // Weekly maintenance
        $schedule->call(function () {
            $this->performMaintenance();
        })->weeklyOn(1, '02:00'); // Monday at 2 AM

        // Monthly reports
        $schedule->command('reports:monthly')
            ->monthlyOn(1, '09:00')
            ->emailOutputTo('reports@example.com');

        // Custom task with conditions
        $schedule->command('cache:clear')
            ->daily()
            ->when(function () {
                return config('app.debug') === false;
            });

        // Task with retry logic
        $schedule->command('sync:external-api')
            ->everyFiveMinutes()
            ->retry(3)
            ->delay(30);
    }

    private function sendEmailReminders()
    {
        // Get users with pending tasks
        $users = User::whereHas('tasks', function ($query) {
            $query->where('due_date', '<=', now()->addDay())
                  ->where('completed', false);
        })->get();

        foreach ($users as $user) {
            Mail::to($user)->send(new TaskReminderMail($user));
        }
    }

    private function performMaintenance()
    {
        // Clear old logs
        \Log::info('Starting weekly maintenance');
        
        // Clean up temporary files
        \Storage::deleteDirectory('temp');
        
        // Update statistics
        $this->updateStatistics();
        
        \Log::info('Weekly maintenance completed');
    }

    private function updateStatistics()
    {
        // Update user statistics
        $userCount = User::count();
        Cache::put('user_count', $userCount, 3600);
        
        // Update order statistics
        $orderCount = Order::whereMonth('created_at', now()->month)->count();
        Cache::put('monthly_orders', $orderCount, 3600);
    }
}

// Custom Artisan Commands
// app/Console/Commands/SendDailyReports.php
namespace App\Console\Commands;

use Illuminate\Console\Command;
use App\Services\ReportService;
use App\Mail\DailyReportMail;

class SendDailyReports extends Command
{
    protected $signature = 'reports:daily {--email=admin@example.com}';
    protected $description = 'Send daily reports to administrators';

    protected $reportService;

    public function __construct(ReportService $reportService)
    {
        parent::__construct();
        $this->reportService = $reportService;
    }

    public function handle()
    {
        $this->info('Generating daily reports...');

        $report = $this->reportService->generateDailyReport();
        
        $email = $this->option('email');
        
        Mail::to($email)->send(new DailyReportMail($report));
        
        $this->info("Daily report sent to {$email}");
        
        return 0;
    }
}

// app/Console/Commands/CleanupExpiredTokens.php
namespace App\Console\Commands;

use Illuminate\Console\Command;
use App\Models\PasswordReset;
use App\Models\EmailVerification;

class CleanupExpiredTokens extends Command
{
    protected $signature = 'tokens:cleanup {--days=7}';
    protected $description = 'Clean up expired tokens';

    public function handle()
    {
        $days = $this->option('days');
        $cutoff = now()->subDays($days);

        $passwordResets = PasswordReset::where('created_at', '<', $cutoff)->count();
        $emailVerifications = EmailVerification::where('created_at', '<', $cutoff)->count();

        PasswordReset::where('created_at', '<', $cutoff)->delete();
        EmailVerification::where('created_at', '<', $cutoff)->delete();

        $this->info("Cleaned up {$passwordResets} password reset tokens");
        $this->info("Cleaned up {$emailVerifications} email verification tokens");
    }
}

// Running the scheduler
// Add to crontab: * * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1

// Manual execution
php artisan schedule:run
php artisan schedule:list
php artisan schedule:work
```

### Q86. Explain Laravel's File Storage system
**Answer:** Laravel provides a powerful filesystem abstraction layer for working with local and cloud storage.

**Real-time Example:**
```php
// config/filesystems.php
return [
    'default' => env('FILESYSTEM_DRIVER', 'local'),
    
    'disks' => [
        'local' => [
            'driver' => 'local',
            'root' => storage_path('app'),
        ],
        
        'public' => [
            'driver' => 'local',
            'root' => storage_path('app/public'),
            'url' => env('APP_URL').'/storage',
            'visibility' => 'public',
        ],
        
        's3' => [
            'driver' => 's3',
            'key' => env('AWS_ACCESS_KEY_ID'),
            'secret' => env('AWS_SECRET_ACCESS_KEY'),
            'region' => env('AWS_DEFAULT_REGION'),
            'bucket' => env('AWS_BUCKET'),
            'url' => env('AWS_URL'),
        ],
        
        'gcs' => [
            'driver' => 'gcs',
            'project_id' => env('GOOGLE_CLOUD_PROJECT_ID'),
            'key_file' => env('GOOGLE_CLOUD_KEY_FILE'),
            'bucket' => env('GOOGLE_CLOUD_STORAGE_BUCKET'),
        ],
    ],
];

// app/Services/FileService.php
namespace App\Services;

use Illuminate\Support\Facades\Storage;
use Illuminate\Http\UploadedFile;
use Intervention\Image\Facades\Image;

class FileService
{
    public function uploadFile(UploadedFile $file, $directory = 'uploads', $disk = 'public')
    {
        // Generate unique filename
        $filename = time() . '_' . $file->getClientOriginalName();
        
        // Store file
        $path = $file->storeAs($directory, $filename, $disk);
        
        return [
            'path' => $path,
            'url' => Storage::disk($disk)->url($path),
            'size' => $file->getSize(),
            'mime_type' => $file->getMimeType()
        ];
    }

    public function uploadImage(UploadedFile $file, $directory = 'images', $disk = 'public')
    {
        // Validate image
        if (!$this->isValidImage($file)) {
            throw new \Exception('Invalid image file');
        }

        // Generate filename
        $filename = time() . '_' . $file->getClientOriginalName();
        $path = $directory . '/' . $filename;

        // Store original
        Storage::disk($disk)->put($path, file_get_contents($file));

        // Create thumbnails
        $this->createThumbnails($file, $directory, $filename, $disk);

        return [
            'original' => [
                'path' => $path,
                'url' => Storage::disk($disk)->url($path)
            ],
            'thumbnails' => $this->getThumbnailUrls($directory, $filename, $disk)
        ];
    }

    private function createThumbnails(UploadedFile $file, $directory, $filename, $disk)
    {
        $sizes = [
            'small' => [150, 150],
            'medium' => [300, 300],
            'large' => [600, 600]
        ];

        foreach ($sizes as $size => $dimensions) {
            $thumbnailPath = $directory . '/thumbnails/' . $size . '_' . $filename;
            
            $image = Image::make($file)
                ->resize($dimensions[0], $dimensions[1], function ($constraint) {
                    $constraint->aspectRatio();
                    $constraint->upsize();
                });

            Storage::disk($disk)->put($thumbnailPath, $image->encode());
        }
    }

    private function getThumbnailUrls($directory, $filename, $disk)
    {
        $sizes = ['small', 'medium', 'large'];
        $urls = [];

        foreach ($sizes as $size) {
            $path = $directory . '/thumbnails/' . $size . '_' . $filename;
            $urls[$size] = Storage::disk($disk)->url($path);
        }

        return $urls;
    }

    public function deleteFile($path, $disk = 'public')
    {
        if (Storage::disk($disk)->exists($path)) {
            Storage::disk($disk)->delete($path);
            return true;
        }
        return false;
    }

    public function moveFile($from, $to, $disk = 'public')
    {
        if (Storage::disk($disk)->exists($from)) {
            Storage::disk($disk)->move($from, $to);
            return true;
        }
        return false;
    }

    public function copyFile($from, $to, $disk = 'public')
    {
        if (Storage::disk($disk)->exists($from)) {
            Storage::disk($disk)->copy($from, $to);
            return true;
        }
        return false;
    }

    public function getFileInfo($path, $disk = 'public')
    {
        if (!Storage::disk($disk)->exists($path)) {
            return null;
        }

        return [
            'size' => Storage::disk($disk)->size($path),
            'last_modified' => Storage::disk($disk)->lastModified($path),
            'mime_type' => Storage::disk($disk)->mimeType($path),
            'url' => Storage::disk($disk)->url($path)
        ];
    }

    private function isValidImage(UploadedFile $file)
    {
        $allowedMimes = ['image/jpeg', 'image/png', 'image/gif', 'image/webp'];
        return in_array($file->getMimeType(), $allowedMimes);
    }
}

// Controller usage
class FileController extends Controller
{
    protected $fileService;

    public function __construct(FileService $fileService)
    {
        $this->fileService = $fileService;
    }

    public function upload(Request $request)
    {
        $request->validate([
            'file' => 'required|file|max:10240', // 10MB max
            'type' => 'required|in:document,image'
        ]);

        $file = $request->file('file');
        $type = $request->input('type');

        try {
            if ($type === 'image') {
                $result = $this->fileService->uploadImage($file);
            } else {
                $result = $this->fileService->uploadFile($file);
            }

            return response()->json([
                'success' => true,
                'file' => $result
            ]);
        } catch (\Exception $e) {
            return response()->json([
                'success' => false,
                'message' => $e->getMessage()
            ], 400);
        }
    }

    public function delete(Request $request)
    {
        $request->validate([
            'path' => 'required|string'
        ]);

        $deleted = $this->fileService->deleteFile($request->path);

        return response()->json([
            'success' => $deleted,
            'message' => $deleted ? 'File deleted successfully' : 'File not found'
        ]);
    }
}

// Model with file handling
class Post extends Model
{
    protected $fillable = ['title', 'content', 'image'];

    public function setImageAttribute($value)
    {
        if (is_string($value)) {
            $this->attributes['image'] = $value;
        } elseif ($value instanceof UploadedFile) {
            $fileService = app(FileService::class);
            $result = $fileService->uploadImage($value, 'posts');
            $this->attributes['image'] = $result['original']['path'];
        }
    }

    public function getImageUrlAttribute()
    {
        if ($this->image) {
            return Storage::disk('public')->url($this->image);
        }
        return null;
    }

    public function getThumbnailUrlAttribute()
    {
        if ($this->image) {
            $path = str_replace('posts/', 'posts/thumbnails/medium_', $this->image);
            return Storage::disk('public')->url($path);
        }
        return null;
    }
}
```

---

## Laravel Security

### Q87. Laravel's Security Features
**Answer:** Laravel provides comprehensive security features including CSRF protection, XSS prevention, SQL injection protection, and more.

**Real-time Example:**
```php
// Security Middleware and Protection
class SecurityController extends Controller
{
    // CSRF Protection
    public function store(Request $request)
    {
        // Laravel automatically validates CSRF token
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users',
            'password' => 'required|min:8|confirmed'
        ]);
        
        // Create user with hashed password
        $user = User::create([
            'name' => $validated['name'],
            'email' => $validated['email'],
            'password' => Hash::make($validated['password'])
        ]);
        
        return response()->json(['user' => $user], 201);
    }
    
    // XSS Protection
    public function displayContent(Request $request)
    {
        $content = $request->input('content');
        
        // Laravel automatically escapes output in Blade templates
        // But for API responses, manually escape
        $safeContent = e($content); // Equivalent to htmlspecialchars()
        
        return response()->json(['content' => $safeContent]);
    }
    
    // SQL Injection Protection (Eloquent ORM handles this)
    public function searchUsers(Request $request)
    {
        $query = $request->input('search');
        
        // Safe - Eloquent automatically escapes
        $users = User::where('name', 'LIKE', "%{$query}%")
                    ->where('active', true)
                    ->get();
        
        // Never do this (vulnerable to SQL injection):
        // DB::select("SELECT * FROM users WHERE name = '{$query}'");
        
        return response()->json(['users' => $users]);
    }
    
    // Rate Limiting
    public function login(Request $request)
    {
        // Apply rate limiting in routes/api.php
        // Route::post('/login')->middleware('throttle:5,1');
        
        $credentials = $request->validate([
            'email' => 'required|email',
            'password' => 'required'
        ]);
        
        if (Auth::attempt($credentials)) {
            $user = Auth::user();
            $token = $user->createToken('auth-token')->plainTextToken;
            
            return response()->json([
                'user' => $user,
                'token' => $token
            ]);
        }
        
        return response()->json(['error' => 'Invalid credentials'], 401);
    }
}

// Security Middleware
class SecurityMiddleware
{
    public function handle($request, Closure $next)
    {
        // Add security headers
        $response = $next($request);
        
        $response->headers->set('X-Content-Type-Options', 'nosniff');
        $response->headers->set('X-Frame-Options', 'DENY');
        $response->headers->set('X-XSS-Protection', '1; mode=block');
        $response->headers->set('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
        
        return $response;
    }
}

// Input Sanitization
class InputSanitizer
{
    public static function sanitize($input)
    {
        // Remove HTML tags
        $input = strip_tags($input);
        
        // Escape special characters
        $input = htmlspecialchars($input, ENT_QUOTES, 'UTF-8');
        
        // Remove null bytes
        $input = str_replace(chr(0), '', $input);
        
        return trim($input);
    }
    
    public static function validateFile($file)
    {
        $allowedTypes = ['image/jpeg', 'image/png', 'image/gif'];
        $maxSize = 5 * 1024 * 1024; // 5MB
        
        if (!in_array($file->getMimeType(), $allowedTypes)) {
            throw new \Exception('Invalid file type');
        }
        
        if ($file->getSize() > $maxSize) {
            throw new \Exception('File too large');
        }
        
        return true;
    }
}
```

### Q88. Laravel Authentication & Authorization
**Answer:** Laravel provides built-in authentication and authorization systems with policies, gates, and middleware.

**Real-time Example:**
```php
// Authentication System
class AuthController extends Controller
{
    public function register(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|string|email|max:255|unique:users',
            'password' => 'required|string|min:8|confirmed'
        ]);
        
        $user = User::create([
            'name' => $validated['name'],
            'email' => $validated['email'],
            'password' => Hash::make($validated['password'])
        ]);
        
        // Send verification email
        $user->sendEmailVerificationNotification();
        
        return response()->json(['message' => 'User registered successfully']);
    }
    
    public function login(Request $request)
    {
        $credentials = $request->validate([
            'email' => 'required|email',
            'password' => 'required'
        ]);
        
        if (Auth::attempt($credentials)) {
            $user = Auth::user();
            $token = $user->createToken('auth-token')->plainTextToken;
            
            return response()->json([
                'user' => $user,
                'token' => $token
            ]);
        }
        
        return response()->json(['error' => 'Invalid credentials'], 401);
    }
    
    public function logout(Request $request)
    {
        $request->user()->currentAccessToken()->delete();
        
        return response()->json(['message' => 'Logged out successfully']);
    }
}

// Authorization with Gates
class PostController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth');
    }
    
    public function store(Request $request)
    {
        // Check authorization
        if (!Gate::allows('create-post')) {
            return response()->json(['error' => 'Unauthorized'], 403);
        }
        
        $validated = $request->validate([
            'title' => 'required|string|max:255',
            'content' => 'required|string'
        ]);
        
        $post = Post::create([
            'title' => $validated['title'],
            'content' => $validated['content'],
            'user_id' => Auth::id()
        ]);
        
        return response()->json(['post' => $post], 201);
    }
    
    public function update(Request $request, Post $post)
    {
        // Check if user can update this post
        if (!Gate::allows('update-post', $post)) {
            return response()->json(['error' => 'Unauthorized'], 403);
        }
        
        $validated = $request->validate([
            'title' => 'sometimes|string|max:255',
            'content' => 'sometimes|string'
        ]);
        
        $post->update($validated);
        
        return response()->json(['post' => $post]);
    }
}

// Policy-based Authorization
class PostPolicy
{
    public function view(User $user, Post $post)
    {
        return $post->is_published || $user->id === $post->user_id;
    }
    
    public function create(User $user)
    {
        return $user->hasRole('author') || $user->hasRole('admin');
    }
    
    public function update(User $user, Post $post)
    {
        return $user->id === $post->user_id || $user->hasRole('admin');
    }
    
    public function delete(User $user, Post $post)
    {
        return $user->id === $post->user_id || $user->hasRole('admin');
    }
}

// Role-based Access Control
class RoleMiddleware
{
    public function handle($request, Closure $next, $role)
    {
        if (!Auth::check() || !Auth::user()->hasRole($role)) {
            return response()->json(['error' => 'Unauthorized'], 403);
        }
        
        return $next($request);
    }
}
```

---

## Laravel API Development

### Q89. Laravel API Development Best Practices
**Answer:** Laravel provides excellent tools for building RESTful APIs with proper structure, authentication, and documentation.

**Real-time Example:**
```php
// API Resource Controllers
class UserController extends Controller
{
    public function index(Request $request)
    {
        $query = User::query();
        
        // Search functionality
        if ($request->has('search')) {
            $query->where('name', 'like', '%' . $request->search . '%')
                  ->orWhere('email', 'like', '%' . $request->search . '%');
        }
        
        // Filtering
        if ($request->has('role')) {
            $query->whereHas('roles', function($q) use ($request) {
                $q->where('name', $request->role);
            });
        }
        
        // Sorting
        $sortBy = $request->get('sort_by', 'created_at');
        $sortOrder = $request->get('sort_order', 'desc');
        $query->orderBy($sortBy, $sortOrder);
        
        // Pagination
        $perPage = $request->get('per_page', 15);
        $users = $query->paginate($perPage);
        
        return UserResource::collection($users);
    }
    
    public function store(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users',
            'password' => 'required|min:8|confirmed',
            'role' => 'sometimes|string|in:user,admin,moderator'
        ]);
        
        $user = User::create([
            'name' => $validated['name'],
            'email' => $validated['email'],
            'password' => Hash::make($validated['password'])
        ]);
        
        if (isset($validated['role'])) {
            $user->assignRole($validated['role']);
        }
        
        return new UserResource($user);
    }
    
    public function show(User $user)
    {
        return new UserResource($user);
    }
    
    public function update(Request $request, User $user)
    {
        $validated = $request->validate([
            'name' => 'sometimes|string|max:255',
            'email' => 'sometimes|email|unique:users,email,' . $user->id,
            'role' => 'sometimes|string|in:user,admin,moderator'
        ]);
        
        $user->update($validated);
        
        if (isset($validated['role'])) {
            $user->syncRoles([$validated['role']]);
        }
        
        return new UserResource($user);
    }
    
    public function destroy(User $user)
    {
        $user->delete();
        
        return response()->json(['message' => 'User deleted successfully']);
    }
}

// API Resources
class UserResource extends JsonResource
{
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at->toISOString(),
            'updated_at' => $this->updated_at->toISOString(),
            'roles' => $this->whenLoaded('roles', function() {
                return $this->roles->pluck('name');
            }),
            'profile' => new UserProfileResource($this->whenLoaded('profile'))
        ];
    }
}

// API Versioning
class ApiV1Controller extends Controller
{
    public function users()
    {
        return User::all();
    }
}

class ApiV2Controller extends Controller
{
    public function users()
    {
        return UserResource::collection(User::all());
    }
}

// API Rate Limiting
class ApiRateLimiter
{
    public function handle($request, Closure $next)
    {
        $key = $request->user() ? $request->user()->id : $request->ip();
        
        if (RateLimiter::tooManyAttempts($key, 100)) {
            return response()->json(['error' => 'Too many requests'], 429);
        }
        
        RateLimiter::hit($key, 60); // 1 minute
        
        return $next($request);
    }
}
```

---

## Laravel Caching & Sessions

### Q90. Laravel Caching System
**Answer:** Laravel provides multiple caching drivers and strategies for improving application performance.

**Real-time Example:**
```php
// Caching Strategies
class CacheService
{
    // Basic Caching
    public function getUserData($userId)
    {
        $cacheKey = "user_data_{$userId}";
        
        return Cache::remember($cacheKey, 3600, function() use ($userId) {
            return User::with(['profile', 'roles', 'posts'])
                      ->find($userId);
        });
    }
    
    // Cache Tags (Redis)
    public function getPostsByCategory($categoryId)
    {
        return Cache::tags(['posts', 'category_' . $categoryId])
                    ->remember("posts_category_{$categoryId}", 1800, function() use ($categoryId) {
                        return Post::where('category_id', $categoryId)
                                  ->with(['author', 'tags'])
                                  ->published()
                                  ->get();
                    });
    }
    
    // Cache Invalidation
    public function updatePost($postId, $data)
    {
        $post = Post::find($postId);
        $post->update($data);
        
        // Invalidate related caches
        Cache::tags(['posts', 'category_' . $post->category_id])->flush();
        Cache::forget("post_{$postId}");
        
        return $post;
    }
    
    // Cache Locking
    public function processExpensiveOperation($data)
    {
        $lockKey = 'expensive_operation_' . md5(serialize($data));
        
        return Cache::lock($lockKey, 60)->get(function() use ($data) {
            // Expensive operation that should only run once
            return $this->performExpensiveCalculation($data);
        });
    }
    
    // Cache Warming
    public function warmCache()
    {
        // Pre-cache popular data
        $popularPosts = Post::popular()->take(100)->get();
        Cache::put('popular_posts', $popularPosts, 3600);
        
        $categories = Category::with('posts')->get();
        Cache::put('categories_with_posts', $categories, 7200);
    }
}

// Session Management
class SessionController extends Controller
{
    public function storeInSession(Request $request)
    {
        // Store data in session
        session(['user_preferences' => $request->all()]);
        
        // Flash data (available for next request only)
        session()->flash('success', 'Preferences saved successfully');
        
        return response()->json(['message' => 'Data stored in session']);
    }
    
    public function getFromSession()
    {
        $preferences = session('user_preferences', []);
        $success = session('success');
        
        return response()->json([
            'preferences' => $preferences,
            'success' => $success
        ]);
    }
    
    public function clearSession()
    {
        session()->flush();
        
        return response()->json(['message' => 'Session cleared']);
    }
}
```

---

## Laravel Broadcasting & Events

### Q91. Laravel Broadcasting System
**Answer:** Laravel's broadcasting system allows real-time communication between server and client using WebSockets, Pusher, or Redis.

**Real-time Example:**
```php
// Broadcasting Events
class MessageSent implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public $message;
    public $user;

    public function __construct($message, $user)
    {
        $this->message = $message;
        $this->user = $user;
    }

    public function broadcastOn()
    {
        return new PrivateChannel('chat.' . $this->message->room_id);
    }

    public function broadcastWith()
    {
        return [
            'message' => $this->message,
            'user' => $this->user,
            'timestamp' => now()->toISOString()
        ];
    }
}

// Real-time Notifications
class UserNotificationSent implements ShouldBroadcast
{
    public $notification;
    public $userId;

    public function __construct($notification, $userId)
    {
        $this->notification = $notification;
        $this->userId = $userId;
    }

    public function broadcastOn()
    {
        return new PrivateChannel('user.' . $this->userId);
    }
}

// Broadcasting Controller
class ChatController extends Controller
{
    public function sendMessage(Request $request)
    {
        $validated = $request->validate([
            'message' => 'required|string|max:1000',
            'room_id' => 'required|exists:chat_rooms,id'
        ]);

        $message = Message::create([
            'content' => $validated['message'],
            'user_id' => Auth::id(),
            'room_id' => $validated['room_id']
        ]);

        // Broadcast the message
        broadcast(new MessageSent($message, Auth::user()));

        return response()->json(['message' => $message]);
    }

    public function joinRoom($roomId)
    {
        $room = ChatRoom::findOrFail($roomId);
        
        // Authorize user can join room
        $this->authorize('view', $room);
        
        return response()->json(['room' => $room]);
    }
}

// Frontend JavaScript (Laravel Echo)
/*
window.Echo.private('chat.' + roomId)
    .listen('MessageSent', (e) => {
        console.log('New message:', e.message);
        // Add message to chat interface
    });

window.Echo.private('user.' + userId)
    .listen('UserNotificationSent', (e) => {
        console.log('New notification:', e.notification);
        // Show notification to user
    });
*/
```

---

## Laravel Artisan Commands

### Q92. Custom Artisan Commands
**Answer:** Laravel's Artisan command system allows creating custom CLI commands for various tasks.

**Real-time Example:**
```php
// Custom Artisan Command
class GenerateReportCommand extends Command
{
    protected $signature = 'report:generate 
                           {type : Type of report (daily, weekly, monthly)}
                           {--format=pdf : Output format (pdf, excel, csv)}
                           {--email : Send report via email}
                           {--recipients=* : Email recipients}';

    protected $description = 'Generate various types of reports';

    public function handle()
    {
        $type = $this->argument('type');
        $format = $this->option('format');
        $email = $this->option('email');
        $recipients = $this->option('recipients');

        $this->info("Generating {$type} report in {$format} format...");

        try {
            $report = $this->generateReport($type, $format);
            
            if ($email && !empty($recipients)) {
                $this->sendReportEmail($report, $recipients);
                $this->info('Report sent to: ' . implode(', ', $recipients));
            } else {
                $this->info("Report saved to: {$report['path']}");
            }
            
        } catch (\Exception $e) {
            $this->error("Failed to generate report: {$e->getMessage()}");
            return 1;
        }

        return 0;
    }

    private function generateReport($type, $format)
    {
        $data = $this->getReportData($type);
        
        switch ($format) {
            case 'pdf':
                return $this->generatePdfReport($data, $type);
            case 'excel':
                return $this->generateExcelReport($data, $type);
            case 'csv':
                return $this->generateCsvReport($data, $type);
            default:
                throw new \Exception("Unsupported format: {$format}");
        }
    }

    private function getReportData($type)
    {
        switch ($type) {
            case 'daily':
                return [
                    'users' => User::whereDate('created_at', today())->count(),
                    'orders' => Order::whereDate('created_at', today())->count(),
                    'revenue' => Order::whereDate('created_at', today())->sum('total')
                ];
            case 'weekly':
                return [
                    'users' => User::whereBetween('created_at', [now()->subWeek(), now()])->count(),
                    'orders' => Order::whereBetween('created_at', [now()->subWeek(), now()])->count(),
                    'revenue' => Order::whereBetween('created_at', [now()->subWeek(), now()])->sum('total')
                ];
            case 'monthly':
                return [
                    'users' => User::whereMonth('created_at', now()->month)->count(),
                    'orders' => Order::whereMonth('created_at', now()->month)->count(),
                    'revenue' => Order::whereMonth('created_at', now()->month)->sum('total')
                ];
            default:
                throw new \Exception("Unsupported report type: {$type}");
        }
    }
}

// Database Maintenance Command
class DatabaseMaintenanceCommand extends Command
{
    protected $signature = 'db:maintenance 
                           {--backup : Create backup before maintenance}
                           {--optimize : Optimize tables}
                           {--cleanup : Clean up old data}';

    protected $description = 'Perform database maintenance tasks';

    public function handle()
    {
        if ($this->option('backup')) {
            $this->createBackup();
        }

        if ($this->option('optimize')) {
            $this->optimizeTables();
        }

        if ($this->option('cleanup')) {
            $this->cleanupOldData();
        }

        $this->info('Database maintenance completed successfully');
    }

    private function createBackup()
    {
        $this->info('Creating database backup...');
        
        $filename = 'backup_' . now()->format('Y_m_d_H_i_s') . '.sql';
        $path = storage_path('backups/' . $filename);
        
        Artisan::call('backup:run', [
            '--filename' => $filename
        ]);
        
        $this->info("Backup created: {$path}");
    }

    private function optimizeTables()
    {
        $this->info('Optimizing database tables...');
        
        $tables = ['users', 'orders', 'products', 'categories'];
        
        foreach ($tables as $table) {
            DB::statement("OPTIMIZE TABLE {$table}");
            $this->line("Optimized table: {$table}");
        }
    }

    private function cleanupOldData()
    {
        $this->info('Cleaning up old data...');
        
        // Clean up old sessions
        DB::table('sessions')->where('last_activity', '<', now()->subDays(30))->delete();
        
        // Clean up old logs
        DB::table('logs')->where('created_at', '<', now()->subDays(90))->delete();
        
        // Clean up old notifications
        DB::table('notifications')->where('created_at', '<', now()->subDays(30))->delete();
        
        $this->info('Old data cleanup completed');
    }
}
```

---

## Laravel Localization

### Q93. Laravel Localization System
**Answer:** Laravel provides comprehensive internationalization (i18n) support for multi-language applications.

**Real-time Example:**
```php
// Language Files
// resources/lang/en/messages.php
return [
    'welcome' => 'Welcome, :name!',
    'greeting' => 'Hello :name, you have :count unread messages.',
    'validation' => [
        'required' => 'The :attribute field is required.',
        'email' => 'The :attribute must be a valid email address.',
        'min' => 'The :attribute must be at least :min characters.',
    ],
    'user' => [
        'profile' => 'Profile',
        'settings' => 'Settings',
        'logout' => 'Logout'
    ]
];

// resources/lang/es/messages.php
return [
    'welcome' => '¡Bienvenido, :name!',
    'greeting' => 'Hola :name, tienes :count mensajes no leídos.',
    'validation' => [
        'required' => 'El campo :attribute es obligatorio.',
        'email' => 'El :attribute debe ser una dirección de correo válida.',
        'min' => 'El :attribute debe tener al menos :min caracteres.',
    ],
    'user' => [
        'profile' => 'Perfil',
        'settings' => 'Configuración',
        'logout' => 'Cerrar sesión'
    ]
];

// Localization Controller
class LocalizationController extends Controller
{
    public function setLocale($locale)
    {
        if (in_array($locale, ['en', 'es', 'fr', 'de'])) {
            session(['locale' => $locale]);
            app()->setLocale($locale);
        }
        
        return redirect()->back();
    }
    
    public function getLocalizedContent()
    {
        $content = [
            'welcome' => __('messages.welcome', ['name' => Auth::user()->name]),
            'greeting' => trans_choice('messages.greeting', 5, [
                'name' => Auth::user()->name,
                'count' => 5
            ]),
            'navigation' => [
                'profile' => __('messages.user.profile'),
                'settings' => __('messages.user.settings'),
                'logout' => __('messages.user.logout')
            ]
        ];
        
        return response()->json($content);
    }
}

// Localized Validation
class UserRequest extends FormRequest
{
    public function rules()
    {
        return [
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users',
            'password' => 'required|min:8|confirmed'
        ];
    }
    
    public function messages()
    {
        return [
            'name.required' => __('messages.validation.required', ['attribute' => 'name']),
            'email.required' => __('messages.validation.required', ['attribute' => 'email']),
            'email.email' => __('messages.validation.email', ['attribute' => 'email']),
            'password.required' => __('messages.validation.required', ['attribute' => 'password']),
            'password.min' => __('messages.validation.min', ['attribute' => 'password', 'min' => 8])
        ];
    }
}

// Multi-language Model
class Post extends Model
{
    protected $fillable = ['title', 'content', 'locale'];
    
    public function getLocalizedTitleAttribute()
    {
        return $this->getTranslation('title', app()->getLocale());
    }
    
    public function getLocalizedContentAttribute()
    {
        return $this->getTranslation('content', app()->getLocale());
    }
    
    private function getTranslation($field, $locale)
    {
        $translations = json_decode($this->attributes[$field], true) ?? [];
        return $translations[$locale] ?? $translations['en'] ?? '';
    }
    
    public function setTranslation($field, $locale, $value)
    {
        $translations = json_decode($this->attributes[$field], true) ?? [];
        $translations[$locale] = $value;
        $this->attributes[$field] = json_encode($translations);
    }
}
```

---

## Laravel Validation

### Q94. Advanced Laravel Validation
**Answer:** Laravel provides powerful validation features including custom rules, conditional validation, and complex validation scenarios.

**Real-time Example:**
```php
// Custom Validation Rules
class StrongPasswordRule implements Rule
{
    public function passes($attribute, $value)
    {
        return preg_match('/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/', $value);
    }
    
    public function message()
    {
        return 'The :attribute must contain at least one uppercase letter, one lowercase letter, one number, and one special character.';
    }
}

class UniqueEmailRule implements Rule
{
    private $ignoreId;
    
    public function __construct($ignoreId = null)
    {
        $this->ignoreId = $ignoreId;
    }
    
    public function passes($attribute, $value)
    {
        $query = User::where('email', $value);
        
        if ($this->ignoreId) {
            $query->where('id', '!=', $this->ignoreId);
        }
        
        return !$query->exists();
    }
    
    public function message()
    {
        return 'The :attribute has already been taken.';
    }
}

// Complex Validation Request
class UserRegistrationRequest extends FormRequest
{
    public function authorize()
    {
        return true;
    }
    
    public function rules()
    {
        $rules = [
            'name' => 'required|string|max:255',
            'email' => ['required', 'email', new UniqueEmailRule()],
            'password' => ['required', 'string', 'min:8', 'confirmed', new StrongPasswordRule()],
            'phone' => 'required|string|regex:/^\+?[1-9]\d{1,14}$/',
            'date_of_birth' => 'required|date|before:today',
            'terms' => 'required|accepted'
        ];
        
        // Conditional validation
        if ($this->input('user_type') === 'business') {
            $rules['company_name'] = 'required|string|max:255';
            $rules['tax_id'] = 'required|string|size:9';
            $rules['business_license'] = 'required|file|mimes:pdf,jpg,png|max:2048';
        }
        
        return $rules;
    }
    
    public function messages()
    {
        return [
            'name.required' => 'Your name is required.',
            'email.required' => 'Email address is required.',
            'email.email' => 'Please provide a valid email address.',
            'password.required' => 'Password is required.',
            'password.min' => 'Password must be at least 8 characters.',
            'password.confirmed' => 'Password confirmation does not match.',
            'phone.required' => 'Phone number is required.',
            'phone.regex' => 'Please provide a valid phone number.',
            'date_of_birth.required' => 'Date of birth is required.',
            'date_of_birth.before' => 'You must be at least 18 years old.',
            'terms.required' => 'You must accept the terms and conditions.',
            'terms.accepted' => 'You must accept the terms and conditions.'
        ];
    }
    
    public function withValidator($validator)
    {
        $validator->after(function ($validator) {
            if ($this->input('date_of_birth')) {
                $age = Carbon::parse($this->input('date_of_birth'))->age;
                if ($age < 18) {
                    $validator->errors()->add('date_of_birth', 'You must be at least 18 years old to register.');
                }
            }
        });
    }
}

// Dynamic Validation
class DynamicValidationController extends Controller
{
    public function validateUser(Request $request)
    {
        $rules = $this->buildValidationRules($request);
        
        $validator = Validator::make($request->all(), $rules);
        
        if ($validator->fails()) {
            return response()->json([
                'errors' => $validator->errors()
            ], 422);
        }
        
        return response()->json(['message' => 'Validation passed']);
    }
    
    private function buildValidationRules(Request $request)
    {
        $rules = [
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users,email'
        ];
        
        // Add rules based on user role
        if ($request->has('role')) {
            switch ($request->input('role')) {
                case 'admin':
                    $rules['admin_code'] = 'required|string|size:10';
                    break;
                case 'moderator':
                    $rules['moderator_approval'] = 'required|boolean';
                    break;
                case 'user':
                    $rules['phone'] = 'required|string|regex:/^\+?[1-9]\d{1,14}$/';
                    break;
            }
        }
        
        // Add rules based on registration source
        if ($request->has('registration_source')) {
            switch ($request->input('registration_source')) {
                case 'social':
                    $rules['social_id'] = 'required|string';
                    $rules['social_provider'] = 'required|string|in:google,facebook,twitter';
                    break;
                case 'invite':
                    $rules['invite_code'] = 'required|string|exists:invites,code';
                    break;
            }
        }
        
        return $rules;
    }
}
```

---

## Laravel Blade Templates

### Q95. Advanced Blade Templating
**Answer:** Laravel's Blade templating engine provides powerful features for creating dynamic, reusable templates.

**Real-time Example:**
```php
// Master Layout Template
// resources/views/layouts/app.blade.php
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="csrf-token" content="{{ csrf_token() }}">
    
    <title>@yield('title', config('app.name'))</title>
    
    <!-- Styles -->
    @stack('styles')
    <link href="{{ asset('css/app.css') }}" rel="stylesheet">
</head>
<body>
    <div id="app">
        @include('partials.navigation')
        
        <main class="main-content">
            @yield('content')
        </main>
        
        @include('partials.footer')
    </div>
    
    <!-- Scripts -->
    @stack('scripts')
    <script src="{{ asset('js/app.js') }}"></script>
</body>
</html>

// Component-based Template
// resources/views/components/user-card.blade.php
@props(['user', 'showActions' => true])

<div class="user-card">
    <div class="user-avatar">
        <img src="{{ $user->avatar_url ?? '/images/default-avatar.png' }}" 
             alt="{{ $user->name }}" 
             class="rounded-full w-16 h-16">
    </div>
    
    <div class="user-info">
        <h3 class="user-name">{{ $user->name }}</h3>
        <p class="user-email">{{ $user->email }}</p>
        <p class="user-role">{{ $user->role->name ?? 'User' }}</p>
    </div>
    
    @if($showActions)
        <div class="user-actions">
            <a href="{{ route('users.show', $user) }}" class="btn btn-primary">View</a>
            <a href="{{ route('users.edit', $user) }}" class="btn btn-secondary">Edit</a>
        </div>
    @endif
</div>

// Advanced Blade Features
// resources/views/users/index.blade.php
@extends('layouts.app')

@section('title', 'Users Management')

@push('styles')
    <link href="{{ asset('css/datatables.css') }}" rel="stylesheet">
@endpush

@section('content')
<div class="container">
    <div class="row">
        <div class="col-12">
            <div class="card">
                <div class="card-header">
                    <h2>Users Management</h2>
                    <a href="{{ route('users.create') }}" class="btn btn-primary">
                        <i class="fas fa-plus"></i> Add New User
                    </a>
                </div>
                
                <div class="card-body">
                    <!-- Search and Filters -->
                    <div class="row mb-3">
                        <div class="col-md-6">
                            <input type="text" 
                                   class="form-control" 
                                   placeholder="Search users..." 
                                   id="user-search">
                        </div>
                        <div class="col-md-3">
                            <select class="form-control" id="role-filter">
                                <option value="">All Roles</option>
                                @foreach($roles as $role)
                                    <option value="{{ $role->id }}">{{ $role->name }}</option>
                                @endforeach
                            </select>
                        </div>
                        <div class="col-md-3">
                            <select class="form-control" id="status-filter">
                                <option value="">All Status</option>
                                <option value="active">Active</option>
                                <option value="inactive">Inactive</option>
                            </select>
                        </div>
                    </div>
                    
                    <!-- Users Table -->
                    <div class="table-responsive">
                        <table class="table table-striped" id="users-table">
                            <thead>
                                <tr>
                                    <th>Name</th>
                                    <th>Email</th>
                                    <th>Role</th>
                                    <th>Status</th>
                                    <th>Created</th>
                                    <th>Actions</th>
                                </tr>
                            </thead>
                            <tbody>
                                @forelse($users as $user)
                                    <tr>
                                        <td>
                                            <div class="d-flex align-items-center">
                                                <img src="{{ $user->avatar_url ?? '/images/default-avatar.png' }}" 
                                                     alt="{{ $user->name }}" 
                                                     class="rounded-circle me-2" 
                                                     width="32" height="32">
                                                {{ $user->name }}
                                            </div>
                                        </td>
                                        <td>{{ $user->email }}</td>
                                        <td>
                                            <span class="badge bg-{{ $user->role->color ?? 'secondary' }}">
                                                {{ $user->role->name ?? 'User' }}
                                            </span>
                                        </td>
                                        <td>
                                            <span class="badge bg-{{ $user->is_active ? 'success' : 'danger' }}">
                                                {{ $user->is_active ? 'Active' : 'Inactive' }}
                                            </span>
                                        </td>
                                        <td>{{ $user->created_at->format('M d, Y') }}</td>
                                        <td>
                                            <div class="btn-group" role="group">
                                                <a href="{{ route('users.show', $user) }}" 
                                                   class="btn btn-sm btn-outline-primary">
                                                    <i class="fas fa-eye"></i>
                                                </a>
                                                <a href="{{ route('users.edit', $user) }}" 
                                                   class="btn btn-sm btn-outline-secondary">
                                                    <i class="fas fa-edit"></i>
                                                </a>
                                                @can('delete', $user)
                                                    <button type="button" 
                                                            class="btn btn-sm btn-outline-danger"
                                                            onclick="deleteUser({{ $user->id }})">
                                                        <i class="fas fa-trash"></i>
                                                    </button>
                                                @endcan
                                            </div>
                                        </td>
                                    </tr>
                                @empty
                                    <tr>
                                        <td colspan="6" class="text-center">
                                            <div class="py-4">
                                                <i class="fas fa-users fa-3x text-muted mb-3"></i>
                                                <p class="text-muted">No users found</p>
                                                <a href="{{ route('users.create') }}" class="btn btn-primary">
                                                    Add First User
                                                </a>
                                            </div>
                                        </td>
                                    </tr>
                                @endforelse
                            </tbody>
                        </table>
                    </div>
                    
                    <!-- Pagination -->
                    @if($users->hasPages())
                        <div class="d-flex justify-content-center">
                            {{ $users->links() }}
                        </div>
                    @endif
                </div>
            </div>
        </div>
    </div>
</div>
@endsection

@push('scripts')
<script>
    // User search functionality
    document.getElementById('user-search').addEventListener('input', function() {
        const searchTerm = this.value.toLowerCase();
        const rows = document.querySelectorAll('#users-table tbody tr');
        
        rows.forEach(row => {
            const name = row.cells[0].textContent.toLowerCase();
            const email = row.cells[1].textContent.toLowerCase();
            
            if (name.includes(searchTerm) || email.includes(searchTerm)) {
                row.style.display = '';
            } else {
                row.style.display = 'none';
            }
        });
    });
    
    // Delete user function
    function deleteUser(userId) {
        if (confirm('Are you sure you want to delete this user?')) {
            fetch(`/users/${userId}`, {
                method: 'DELETE',
                headers: {
                    'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').content,
                    'Content-Type': 'application/json'
                }
            })
            .then(response => response.json())
            .then(data => {
                if (data.success) {
                    location.reload();
                } else {
                    alert('Error deleting user: ' + data.message);
                }
            })
            .catch(error => {
                console.error('Error:', error);
                alert('An error occurred while deleting the user');
            });
        }
    }
</script>
@endpush
```

**Total Laravel Questions: 100+ covering all major topics with deep explanations and real-world examples!**

---

## 🎯 PRACTICAL INTERVIEW QUESTIONS & ANSWERS

### 🔥 Most Asked Laravel Interview Questions

#### **Q1. How would you implement a complete user authentication system in Laravel?**

**Answer:** Here's a comprehensive implementation:

```php
// 1. User Model with relationships
class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;

    protected $fillable = [
        'name', 'email', 'password', 'role', 'email_verified_at'
    ];

    protected $hidden = ['password', 'remember_token'];

    // Relationships
    public function posts()
    {
        return $this->hasMany(Post::class);
    }

    public function comments()
    {
        return $this->hasMany(Comment::class);
    }

    // Accessor
    public function getFullNameAttribute()
    {
        return $this->first_name . ' ' . $this->last_name;
    }

    // Mutator
    public function setPasswordAttribute($value)
    {
        $this->attributes['password'] = Hash::make($value);
    }
}

// 2. Authentication Controller
class AuthController extends Controller
{
    public function register(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|string|email|max:255|unique:users',
            'password' => 'required|string|min:8|confirmed',
        ]);

        $user = User::create($validated);
        $token = $user->createToken('auth_token')->plainTextToken;

        return response()->json([
            'access_token' => $token,
            'token_type' => 'Bearer',
            'user' => $user
        ]);
    }

    public function login(Request $request)
    {
        if (!Auth::attempt($request->only('email', 'password'))) {
            return response()->json(['message' => 'Invalid credentials'], 401);
        }

        $user = User::where('email', $request->email)->firstOrFail();
        $token = $user->createToken('auth_token')->plainTextToken;

        return response()->json([
            'access_token' => $token,
            'token_type' => 'Bearer',
            'user' => $user
        ]);
    }

    public function logout(Request $request)
    {
        $request->user()->currentAccessToken()->delete();
        return response()->json(['message' => 'Logged out successfully']);
    }
}

// 3. Middleware for role-based access
class RoleMiddleware
{
    public function handle($request, Closure $next, $role)
    {
        if (!Auth::check() || Auth::user()->role !== $role) {
            return response()->json(['message' => 'Unauthorized'], 403);
        }

        return $next($request);
    }
}

// 4. Routes
Route::post('/register', [AuthController::class, 'register']);
Route::post('/login', [AuthController::class, 'login']);

Route::middleware('auth:sanctum')->group(function () {
    Route::post('/logout', [AuthController::class, 'logout']);
    Route::get('/user', function (Request $request) {
        return $request->user();
    });
});
```

#### **Q2. How would you implement a multi-tenant application in Laravel?**

**Answer:** Here's a complete multi-tenant implementation:

```php
// 1. Tenant Model
class Tenant extends Model
{
    protected $fillable = ['name', 'domain', 'database', 'is_active'];

    public function users()
    {
        return $this->hasMany(User::class);
    }
}

// 2. Tenant Middleware
class TenantMiddleware
{
    public function handle($request, Closure $next)
    {
        $tenant = $this->resolveTenant($request);
        
        if (!$tenant) {
            return response()->json(['error' => 'Tenant not found'], 404);
        }

        if (!$tenant->is_active) {
            return response()->json(['error' => 'Tenant is inactive'], 403);
        }

        // Set tenant in service container
        app()->instance('tenant', $tenant);
        
        // Switch database connection
        config(['database.default' => $tenant->database]);
        
        return $next($request);
    }

    private function resolveTenant($request)
    {
        $domain = $request->getHost();
        return Tenant::where('domain', $domain)->first();
    }
}

// 3. Tenant-aware User Model
class User extends Authenticatable
{
    protected $fillable = ['name', 'email', 'password', 'tenant_id'];

    public function tenant()
    {
        return $this->belongsTo(Tenant::class);
    }

    protected static function boot()
    {
        parent::boot();
        
        static::creating(function ($user) {
            if (!$user->tenant_id) {
                $user->tenant_id = app('tenant')->id;
            }
        });
    }
}
```

#### **Q3. How would you implement a real-time chat system using Laravel?**

**Answer:** Complete real-time chat implementation:

```php
// 1. Message Model
class Message extends Model
{
    protected $fillable = ['user_id', 'room_id', 'message', 'type'];

    public function user()
    {
        return $this->belongsTo(User::class);
    }

    public function room()
    {
        return $this->belongsTo(ChatRoom::class);
    }
}

// 2. Message Controller
class MessageController extends Controller
{
    public function store(Request $request)
    {
        $validated = $request->validate([
            'room_id' => 'required|exists:chat_rooms,id',
            'message' => 'required|string|max:1000',
            'type' => 'in:text,image,file'
        ]);

        $message = Message::create([
            'user_id' => auth()->id(),
            'room_id' => $validated['room_id'],
            'message' => $validated['message'],
            'type' => $validated['type'] ?? 'text'
        ]);

        // Broadcast the message
        broadcast(new MessageSent($message))->toOthers();

        return response()->json([
            'message' => $message->load('user'),
            'status' => 'Message sent successfully'
        ]);
    }
}

// 3. Message Sent Event
class MessageSent implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public $message;

    public function __construct(Message $message)
    {
        $this->message = $message->load('user');
    }

    public function broadcastOn()
    {
        return new PresenceChannel('chat.room.' . $this->message->room_id);
    }

    public function broadcastAs()
    {
        return 'message.sent';
    }
}
```

#### **Q4. How would you implement a file upload system with image optimization?**

**Answer:** Complete file upload implementation:

```php
// 1. File Upload Controller
class FileUploadController extends Controller
{
    public function upload(Request $request)
    {
        $validated = $request->validate([
            'file' => 'required|file|mimes:jpg,jpeg,png,gif,pdf,doc,docx|max:10240',
            'type' => 'required|in:avatar,document,image'
        ]);

        $file = $validated['file'];
        $type = $validated['type'];

        // Generate unique filename
        $filename = time() . '_' . Str::random(10) . '.' . $file->getClientOriginalExtension();
        
        // Store file
        $path = $file->storeAs('uploads/' . $type, $filename, 'public');

        // If it's an image, create thumbnails
        if (in_array($file->getClientOriginalExtension(), ['jpg', 'jpeg', 'png', 'gif'])) {
            $this->createThumbnails($path, $filename, $type);
        }

        // Save file record to database
        $fileRecord = FileUpload::create([
            'user_id' => auth()->id(),
            'original_name' => $file->getClientOriginalName(),
            'filename' => $filename,
            'path' => $path,
            'type' => $type,
            'size' => $file->getSize(),
            'mime_type' => $file->getMimeType()
        ]);

        return response()->json([
            'file' => $fileRecord,
            'url' => Storage::url($path)
        ]);
    }

    private function createThumbnails($path, $filename, $type)
    {
        $fullPath = Storage::path($path);
        
        // Create different sizes
        $sizes = [
            'thumb' => [150, 150],
            'medium' => [300, 300],
            'large' => [800, 800]
        ];

        foreach ($sizes as $sizeName => $dimensions) {
            $thumbnailPath = 'uploads/' . $type . '/thumbnails/' . $sizeName . '_' . $filename;
            
            // Use Intervention Image for optimization
            $image = Image::make($fullPath)
                ->fit($dimensions[0], $dimensions[1])
                ->encode('jpg', 80); // 80% quality
            
            Storage::put($thumbnailPath, $image);
        }
    }
}
```

#### **Q5. How would you implement a caching system for an e-commerce application?**

**Answer:** Comprehensive caching implementation:

```php
// 1. Cache Service Class
class CacheService
{
    public function cacheProducts($categoryId = null)
    {
        $cacheKey = $categoryId ? "products_category_{$categoryId}" : 'products_all';
        
        return Cache::remember($cacheKey, 3600, function () use ($categoryId) {
            $query = Product::with(['category', 'images', 'reviews']);
            
            if ($categoryId) {
                $query->where('category_id', $categoryId);
            }
            
            return $query->where('is_active', true)->get();
        });
    }

    public function cacheUserCart($userId)
    {
        $cacheKey = "user_cart_{$userId}";
        
        return Cache::remember($cacheKey, 1800, function () use ($userId) {
            return Cart::where('user_id', $userId)
                ->with('product')
                ->get();
        });
    }

    public function invalidateProductCache($productId)
    {
        // Clear specific product cache
        Cache::forget("product_details_{$productId}");
        
        // Clear category cache
        $product = Product::find($productId);
        if ($product) {
            Cache::forget("products_category_{$product->category_id}");
        }
        
        // Clear all products cache
        Cache::forget('products_all');
    }
}
```

#### **Q6. How would you implement a payment gateway integration in Laravel?**

**Answer:** Complete payment gateway implementation:

```php
// 1. Payment Service Interface
interface PaymentGatewayInterface
{
    public function createPayment(array $data);
    public function processPayment(string $paymentId);
    public function refundPayment(string $paymentId, float $amount);
}

// 2. Stripe Payment Gateway
class StripePaymentGateway implements PaymentGatewayInterface
{
    protected $stripe;

    public function __construct()
    {
        \Stripe\Stripe::setApiKey(config('services.stripe.secret'));
    }

    public function createPayment(array $data)
    {
        try {
            $paymentIntent = \Stripe\PaymentIntent::create([
                'amount' => $data['amount'] * 100, // Convert to cents
                'currency' => $data['currency'] ?? 'usd',
                'customer' => $data['customer_id'],
                'metadata' => $data['metadata'] ?? []
            ]);

            return [
                'success' => true,
                'payment_id' => $paymentIntent->id,
                'client_secret' => $paymentIntent->client_secret
            ];
        } catch (\Exception $e) {
            return [
                'success' => false,
                'error' => $e->getMessage()
            ];
        }
    }
}

// 3. Payment Controller
class PaymentController extends Controller
{
    protected $paymentGateway;

    public function __construct(PaymentGatewayInterface $paymentGateway)
    {
        $this->paymentGateway = $paymentGateway;
    }

    public function createPayment(Request $request)
    {
        $validated = $request->validate([
            'amount' => 'required|numeric|min:0.01',
            'currency' => 'required|string|size:3',
            'order_id' => 'required|exists:orders,id'
        ]);

        // Get order details
        $order = Order::findOrFail($validated['order_id']);
        
        $paymentData = [
            'amount' => $validated['amount'],
            'currency' => $validated['currency'],
            'customer_id' => $order->customer_id,
            'metadata' => [
                'order_id' => $order->id
            ]
        ];

        $result = $this->paymentGateway->createPayment($paymentData);

        if ($result['success']) {
            // Save payment record
            $payment = Payment::create([
                'order_id' => $order->id,
                'payment_gateway' => 'stripe',
                'payment_id' => $result['payment_id'],
                'amount' => $validated['amount'],
                'currency' => $validated['currency'],
                'status' => 'pending'
            ]);

            return response()->json([
                'payment' => $payment,
                'client_secret' => $result['client_secret']
            ]);
        }

        return response()->json(['error' => $result['error']], 400);
    }
}
```

#### **Q7. How would you implement a comprehensive logging system in Laravel?**

**Answer:** Complete logging implementation:

```php
// 1. Custom Log Channel
// config/logging.php
'channels' => [
    'user_actions' => [
        'driver' => 'daily',
        'path' => storage_path('logs/user_actions.log'),
        'level' => 'info',
        'days' => 30,
    ],
    
    'api_requests' => [
        'driver' => 'daily',
        'path' => storage_path('logs/api_requests.log'),
        'level' => 'info',
        'days' => 7,
    ],
],

// 2. Logging Middleware
class LoggingMiddleware
{
    public function handle($request, Closure $next)
    {
        $startTime = microtime(true);
        
        $response = $next($request);
        
        $endTime = microtime(true);
        $duration = $endTime - $startTime;
        
        // Log API request
        Log::channel('api_requests')->info('API Request', [
            'method' => $request->method(),
            'url' => $request->fullUrl(),
            'ip' => $request->ip(),
            'user_agent' => $request->userAgent(),
            'user_id' => auth()->id(),
            'status_code' => $response->getStatusCode(),
            'duration' => $duration
        ]);
        
        return $response;
    }
}

// 3. User Action Logger
class UserActionLogger
{
    public static function log($action, $model = null, $data = [])
    {
        Log::channel('user_actions')->info('User Action', [
            'user_id' => auth()->id(),
            'action' => $action,
            'model_type' => $model ? get_class($model) : null,
            'model_id' => $model ? $model->id : null,
            'data' => $data,
            'ip' => request()->ip(),
            'user_agent' => request()->userAgent()
        ]);
    }
}
```

#### **Q8. How would you implement a job queue system for background processing?**

**Answer:** Complete job queue implementation:

```php
// 1. Email Job
class SendWelcomeEmail implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    protected $user;

    public function __construct(User $user)
    {
        $this->user = $user;
    }

    public function handle()
    {
        Mail::to($this->user->email)->send(new WelcomeEmail($this->user));
    }

    public function failed(Throwable $exception)
    {
        Log::error('Welcome email failed', [
            'user_id' => $this->user->id,
            'error' => $exception->getMessage()
        ]);
    }
}

// 2. Image Processing Job
class ProcessUploadedImage implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    protected $image;

    public function __construct(Image $image)
    {
        $this->image = $image;
    }

    public function handle()
    {
        // Process image in background
        $this->createThumbnails();
        $this->extractMetadata();
        $this->optimizeImage();
    }

    private function createThumbnails()
    {
        $sizes = ['thumb' => [150, 150], 'medium' => [300, 300]];
        
        foreach ($sizes as $size => $dimensions) {
            $thumbnail = Image::make(storage_path('app/' . $this->image->path))
                ->fit($dimensions[0], $dimensions[1])
                ->encode('jpg', 80);
            
            $thumbnailPath = 'thumbnails/' . $size . '_' . $this->image->filename;
            Storage::put($thumbnailPath, $thumbnail);
        }
    }
}
```

#### **Q9. How would you implement a comprehensive testing suite in Laravel?**

**Answer:** Complete testing implementation:

```php
// 1. Feature Test - User Authentication
class UserAuthenticationTest extends TestCase
{
    use RefreshDatabase;

    public function test_user_can_register()
    {
        $userData = [
            'name' => 'John Doe',
            'email' => 'john@example.com',
            'password' => 'password123',
            'password_confirmation' => 'password123'
        ];

        $response = $this->postJson('/api/register', $userData);

        $response->assertStatus(201)
            ->assertJsonStructure([
                'user' => ['id', 'name', 'email'],
                'access_token'
            ]);

        $this->assertDatabaseHas('users', [
            'email' => 'john@example.com'
        ]);
    }

    public function test_user_can_login()
    {
        $user = User::factory()->create([
            'email' => 'test@example.com',
            'password' => Hash::make('password123')
        ]);

        $response = $this->postJson('/api/login', [
            'email' => 'test@example.com',
            'password' => 'password123'
        ]);

        $response->assertStatus(200)
            ->assertJsonStructure([
                'user' => ['id', 'name', 'email'],
                'access_token'
            ]);
    }
}

// 2. Unit Test - User Model
class UserTest extends TestCase
{
    use RefreshDatabase;

    public function test_user_has_many_posts()
    {
        $user = User::factory()->create();
        $posts = Post::factory()->count(3)->create(['user_id' => $user->id]);

        $this->assertInstanceOf(Collection::class, $user->posts);
        $this->assertCount(3, $user->posts);
    }

    public function test_user_password_is_hashed()
    {
        $user = User::factory()->create(['password' => 'password123']);

        $this->assertNotEquals('password123', $user->password);
        $this->assertTrue(Hash::check('password123', $user->password));
    }
}
```

#### **Q10. How would you implement a comprehensive API documentation system?**

**Answer:** Complete API documentation implementation:

```php
// 1. API Resource Classes
class UserResource extends JsonResource
{
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at->toISOString(),
            'updated_at' => $this->updated_at->toISOString(),
            'posts' => PostResource::collection($this->whenLoaded('posts')),
        ];
    }
}

// 2. API Controller with Documentation
class UserController extends Controller
{
    /**
     * @OA\Get(
     *     path="/api/users",
     *     summary="Get all users",
     *     description="Retrieve a list of all users",
     *     operationId="getUsers",
     *     tags={"Users"},
     *     @OA\Response(
     *         response=200,
     *         description="Successful operation",
     *         @OA\JsonContent(
     *             @OA\Property(property="data", type="array", @OA\Items(ref="#/components/schemas/User"))
     *         )
     *     )
     * )
     */
    public function index()
    {
        $users = User::paginate(10);
        return UserResource::collection($users);
    }

    /**
     * @OA\Post(
     *     path="/api/users",
     *     summary="Create a new user",
     *     description="Create a new user with the provided data",
     *     operationId="createUser",
     *     tags={"Users"},
     *     @OA\RequestBody(
     *         required=true,
     *         @OA\JsonContent(ref="#/components/schemas/UserCreate")
     *     ),
     *     @OA\Response(
     *         response=201,
     *         description="User created successfully",
     *         @OA\JsonContent(ref="#/components/schemas/User")
     *     )
     * )
     */
    public function store(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|string|email|max:255|unique:users',
            'password' => 'required|string|min:8|confirmed'
        ]);

        $user = User::create($validated);
        return new UserResource($user);
    }
}
```

---

## 🎯 SUMMARY

**Total Laravel Practical Questions: 10+ with complete implementations covering:**

✅ **Authentication & Authorization**  
✅ **Multi-tenant Applications**  
✅ **Real-time Features**  
✅ **File Upload & Processing**  
✅ **Caching Strategies**  
✅ **Payment Integration**  
✅ **Logging Systems**  
✅ **Job Queues**  
✅ **Testing Suites**  
✅ **API Documentation**

Each question includes:
- **Complete working code**
- **Real-world examples**
- **Best practices**
- **Error handling**
- **Performance optimization**
- **Security considerations**

**Perfect for senior Laravel developer interviews! 🚀**

