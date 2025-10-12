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

**Total Laravel Questions: 80+ covering all major topics!**

