Below is a **complete, beginner-to-intermediate explanation of Laravel Cache**, structured exactly as you asked.

---

# 🔹 What is Laravel Cache?

**Laravel Cache** is a mechanism to **store data temporarily** so your application doesn’t have to **recalculate or re-query the database** every time.

Instead of doing this repeatedly:

```text
Request → DB Query → Process → Response
```

Laravel cache does this:

```text
Request → Cache (fast) → Response
```

If data is not found in cache, Laravel:

1. Gets it from DB
2. Stores it in cache
3. Returns it

---

# 🔹 Why Use Cache?

### 🚀 Performance

- Cache is **much faster** than database queries
- Reduces server load

### 💰 Cost Saving

- Fewer DB reads
- Lower CPU usage

### 📈 Scalability

- Handles high traffic better

### ✅ Common Use Cases

- Dashboard statistics
- User profiles
- Settings/config data
- API responses
- Expensive ORM queries

---

# 🔹 Laravel Cache Drivers (Types)

Laravel supports multiple **cache drivers**:

| Driver      | Storage          | Use Case          |
| ----------- | ---------------- | ----------------- |
| `file`      | Storage folder   | Small apps        |
| `database`  | Database table   | Shared cache      |
| `redis`     | Memory           | High performance  |
| `memcached` | Memory           | Distributed cache |
| `array`     | Memory (runtime) | Testing only      |

---

# 🔹 Cache Configuration

### `.env`

```env
CACHE_DRIVER=file
```

### `config/cache.php`

```php
'default' => env('CACHE_DRIVER', 'file'),
```

---

# 🔹 Cache Type 1: File Cache

📂 Stored in:

```
storage/framework/cache/data
```

### Example

```php
use Illuminate\Support\Facades\Cache;

// Store
Cache::put('welcome_message', 'Hello Laravel', 600);

// Get
$message = Cache::get('welcome_message');

// Remove
Cache::forget('welcome_message');
```

---

# 🔹 Cache Type 2: Database Cache

### Step 1: Create Cache Table

```bash
php artisan cache:table
php artisan migrate
```

### Step 2: Update `.env`

```env
CACHE_DRIVER=database
```

### Example

```php
Cache::put('total_users', 1000, 300);

$totalUsers = Cache::get('total_users');
```

📌 Cache stored in `cache` table.

---

# 🔹 Cache Type 3: Redis Cache (Production-Ready)

```env
CACHE_DRIVER=redis
```

```php
Cache::remember('products', 3600, function () {
    return Product::all();
});
```

✔ Fastest
✔ Supports cache tags
✔ Recommended for APIs & large systems

---

# 🔹 How to Write Cache (Common Patterns)

## 1️⃣ `put()` – Manual Cache

```php
Cache::put('site_name', 'My App', now()->addHour());
```

---

## 2️⃣ `remember()` – Best Practice

```php
$users = Cache::remember('users.all', 600, function () {
    return User::all();
});
```

✔ Runs query only once
✔ Automatically caches result

---

## 3️⃣ `rememberForever()`

```php
Cache::rememberForever('settings', function () {
    return Setting::first();
});
```

---

# 🔹 Cache with Database Query (Query Builder)

```php
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Cache;

$orders = Cache::remember('orders.today', 300, function () {
    return DB::table('orders')
        ->whereDate('created_at', today())
        ->get();
});
```

---

# 🔹 Cache with Eloquent ORM (Recommended)

```php
use App\Models\User;
use Illuminate\Support\Facades\Cache;

$users = Cache::remember('users.active', 600, function () {
    return User::where('active', 1)->get();
});
```

---

# 🔹 Cache Single Model (ORM)

```php
$user = Cache::remember("user.{$id}", 300, function () use ($id) {
    return User::find($id);
});
```

---

# 🔹 Cache Pagination (ORM)

```php
$page = request('page', 1);

$posts = Cache::remember("posts.page.$page", 600, function () {
    return Post::paginate(10);
});
```

---

# 🔹 Cache Invalidation (Very Important!)

### Clear One Cache

```php
Cache::forget('users.active');
```

### Clear All Cache

```bash
php artisan cache:clear
```

### Auto-clear Cache on Model Update

```php
// App\Models\User.php
protected static function booted()
{
    static::saved(function () {
        Cache::forget('users.active');
    });
}
```

---

# 🔹 Cache with Tags (Redis / Memcached only)

```php
Cache::tags(['users'])->remember('active', 600, function () {
    return User::where('active', 1)->get();
});

// Clear all user cache
Cache::tags(['users'])->flush();
```

---

# 🔹 Best Practices

✅ Use `remember()`
✅ Use meaningful cache keys
✅ Set expiration time
✅ Clear cache when data changes
❌ Don’t cache highly dynamic data

---

## 1️⃣ Product Model (example)

```php
// app/Models/Product.php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
    protected $fillable = [
        'name',
        'price',
        'description',
    ];
}
```

---

## 2️⃣ ProductController with Cache CRUD

```php
// app/Http/Controllers/ProductController.php
namespace App\Http\Controllers;

use App\Models\Product;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Cache;

class ProductController extends Controller
{
    /**
     * READ (LIST) – Cached
     */
    public function index()
    {
        $products = Cache::remember(
            'products.all',
            now()->addMinutes(10),
            function () {
                return Product::all();
            }
        );

        return response()->json($products);
    }

    /**
     * CREATE – Clear cache
     */
    public function store(Request $request)
    {
        $product = Product::create($request->all());

        // Clear product list cache
        Cache::forget('products.all');

        return response()->json($product, 201);
    }

    /**
     * READ (SINGLE) – Cached
     */
    public function show($id)
    {
        $product = Cache::remember(
            "product.$id",
            now()->addMinutes(10),
            function () use ($id) {
                return Product::findOrFail($id);
            }
        );

        return response()->json($product);
    }

    /**
     * UPDATE – Clear cache
     */
    public function update(Request $request, $id)
    {
        $product = Product::findOrFail($id);
        $product->update($request->all());

        // Clear related cache
        Cache::forget("product.$id");
        Cache::forget('products.all');

        return response()->json($product);
    }

    /**
     * DELETE – Clear cache
     */
    public function destroy($id)
    {
        Product::findOrFail($id)->delete();

        // Clear related cache
        Cache::forget("product.$id");
        Cache::forget('products.all');

        return response()->json(['message' => 'Product deleted']);
    }
}
```

---

## 2️⃣ CustomerController with Cache CRUD

```php

namespace App\Http\Controllers;

use App\Http\Requests\StoreCustomerRequest;
use App\Http\Requests\UpdateCustomerRequest;
use App\Http\Resources\CustomerResource;
use App\Models\Customer;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Gate;

class CustomerController extends Controller
{
    public function __construct() {}

    /**
     * Display a listing of the resource.
     */
    public function index(Request $request)
    {
        $userId = Auth::id();

        // Build unique cache key per request
        $cacheKey = 'customers:index:' . md5(json_encode([
            'user_id' => $userId,
            'q' => $request->get('q'),
            'state' => $request->get('filter_by_state_division'),
            'township' => $request->get('filter_by_township'),
            'sort_by' => $request->get('sort_by', 'id'),
            'sort_direction' => $request->get('sort_direction', 'desc'),
            'page' => $request->get('page', 1),
        ]));

        $customers = Cache::tags(['customers', 'user:' . $userId])
            ->remember($cacheKey, now()->addMinutes(5), function () use ($request, $userId) {

                $query = Customer::query();

                // Search
                if ($keyword = $request->get('q')) {
                    $query->where(function ($q) use ($keyword) {
                        $q->where('name', 'like', "%{$keyword}%")
                            ->orWhere('phone', 'like', "%{$keyword}%")
                            ->orWhere('township', 'like', "%{$keyword}%")
                            ->orWhere('state_division', 'like', "%{$keyword}%")
                            ->orWhere('year', 'like', "%{$keyword}%");
                    });
                }

                // User restriction
                if ($userId != 1) {
                    $query->where('user_id', $userId);
                }

                // Filters
                if ($state = $request->get('filter_by_state_division')) {
                    $query->where('state_division', $state);
                }

                if ($township = $request->get('filter_by_township')) {
                    $query->where('township', $township);
                }

                // Sorting
                $query->orderBy(
                    $request->get('sort_by', 'id'),
                    $request->get('sort_direction', 'desc')
                );

                return $query->paginate(7);
            });

        return CustomerResource::collection($customers);
    }

    /**
     * Store a newly created resource in storage.
     */
    public function store(StoreCustomerRequest $request)
    {
        $customer = Customer::create([
            ...$request->validated(),
            'user_id' => Auth::id(),
        ]);

        $this->clearCustomerCache();

        return new CustomerResource($customer);
    }

    /**
     * Display the specified resource.
     */
    public function show(Customer $customer)
    {
        Gate::authorize('view', $customer);

        return new CustomerResource($customer);
    }

    /**
     * Update the specified resource in storage.
     */
    public function update(UpdateCustomerRequest $request, Customer $customer)
    {
        Gate::authorize('update', $customer);

        $customer->update($request->validated());

        $this->clearCustomerCache();

        return new CustomerResource($customer->fresh());
    }

    /**
     * Remove the specified resource from storage.
     */
    public function destroy(Customer $customer)
    {
        Gate::authorize('delete', $customer);

        $customer->delete();

        $this->clearCustomerCache();

        return response()->json([
            'data' => ['message' => 'Customer deleted successfully'],
        ], 200);
    }

    /**
     * Clear customer-related cache
     */
    protected function clearCustomerCache(): void
    {
        Cache::tags(['customers'])->flush();
    }
}


```

---

## 3️⃣ Routes

```php
use App\Http\Controllers\ProductController;

Route::apiResource('products', ProductController::class);
```

---

## 4️⃣ How Cache Works Here (Flow)

### 🔹 GET /products

```
Request
→ Cache (products.all)
→ If exists → return cache
→ If not → DB query → save cache → return
```

---

### 🔹 POST /products

```
Create product
→ Clear products.all cache
→ Next request will re-cache fresh data
```

---

### 🔹 GET /products/{id}

```
Cache key: product.1
Cache single product
```

---

### 🔹 PUT /products/{id}

```
Update product
→ Clear product.{id}
→ Clear products.all
```

---

### 🔹 DELETE /products/{id}

```
Delete product
→ Clear related caches
```

---

## 5️⃣ Cache Keys Used (Important)

| Key            | Purpose        |
| -------------- | -------------- |
| `products.all` | Product list   |
| `product.{id}` | Single product |

✔ Always use **consistent, predictable keys**

---

## 6️⃣ Better: Auto Clear Cache in Model (Optional)

Instead of clearing cache in controller:

```php
// app/Models/Product.php
use Illuminate\Support\Facades\Cache;

protected static function booted()
{
    static::saved(function ($product) {
        Cache::forget("product.$product->id");
        Cache::forget('products.all');
    });

    static::deleted(function ($product) {
        Cache::forget("product.$product->id");
        Cache::forget('products.all');
    });
}
```

✔ Cleaner controller
✔ Centralized cache logic

---

## 7️⃣ Summary

✔ `remember()` → read cache
✔ `forget()` → clear cache on write
✔ Cache list & single record separately
✔ Always clear cache on create/update/delete
