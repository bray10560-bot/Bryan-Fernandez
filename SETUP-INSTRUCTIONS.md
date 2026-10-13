# Setup Instructions — Role-Based User Management & Dashboards

Roles used: **admin, staff, customer** (matches your Project Proposal: Admin, Staff, Customer).

## Step 1 — Copy files into your project

| File in this package | Goes to (in your project) |
|---|---|
| `database/migrations/2026_09_15_000000_add_role_to_users_table.php` | `database/migrations/` |
| `app/Http/Middleware/CheckRole.php` | `app/Http/Middleware/` |
| `app/Http/Controllers/UserManagementController.php` | `app/Http/Controllers/` |
| `app/Http/Controllers/Auth/AuthenticatedSessionController.php` | **replaces** your existing Breeze file at the same path |
| `resources/views/users/*.blade.php` | `resources/views/users/` |
| `resources/views/dashboard/*.blade.php` | `resources/views/dashboard/` |
| `routes/web-routes-to-add.php` | **copy its contents** into your `routes/web.php` (don't drop the file in as-is) |

## Step 2 — Run the migration

```bash
php artisan migrate
```

This adds a `role` column (default `customer`) to your `users` table.

## Step 3 — Update `app/Models/User.php`

Add `'role'` to the `$fillable` array so it can be mass-assigned:

```php
protected $fillable = [
    'name',
    'email',
    'password',
    'role',
];
```

## Step 4 — Register the `role` middleware

**If your project is Laravel 11+** (no `app/Http/Kernel.php` file, you have `bootstrap/app.php` instead):

```php
// bootstrap/app.php
->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'role' => \App\Http\Middleware\CheckRole::class,
    ]);
})
```

**If your project is Laravel 10 or earlier** (you have `app/Http/Kernel.php`):

```php
// app/Http/Kernel.php
protected $middlewareAliases = [
    // ...existing entries
    'role' => \App\Http\Middleware\CheckRole::class,
];
```
(Older Laravel versions may call this array `$routeMiddleware` instead — add it there.)

Not sure which you have? Run `php artisan --version`.

## Step 5 — Update `routes/web.php`

1. Delete (or comment out) Breeze's default dashboard route:
   ```php
   Route::get('/dashboard', function () {
       return view('dashboard');
   })->middleware(['auth', 'verified'])->name('dashboard');
   ```
2. Paste in the contents of `routes/web-routes-to-add.php`.

## Step 6 — Update any "Dashboard" links in your navigation

Breeze's default nav (`resources/views/layouts/navigation.blade.php`) links to `route('dashboard')`, which no longer exists. Point it to the user's role instead, e.g.:

```blade
<x-nav-link :href="match(auth()->user()->role) {
        'admin' => route('dashboard.admin'),
        'staff' => route('dashboard.staff'),
        default => route('dashboard.customer'),
    }" :active="request()->routeIs('dashboard.*')">
    {{ __('Dashboard') }}
</x-nav-link>
```

## Step 7 — Manually promote your first admin

Since the User Management page is admin-only, you need at least one admin to get in. Register a normal account, then run:

```bash
php artisan tinker
```
```php
$u = App\Models\User::first();
$u->role = 'admin';
$u->save();
```

## Step 8 — Test it (this is what you'll demo)

1. Log in as the **admin** → should land on `/dashboard/admin`.
2. Go to **Manage Users**, create a user with role `staff` and one with role `customer`.
3. Log out, log in as the **staff** user → should land on `/dashboard/staff`.
4. While logged in as staff, manually type `/dashboard/admin` or `/users` in the URL bar → should get a **403 Access Denied** (this is your middleware working).
5. Repeat for the **customer** account.

That covers every rubric line in both labs: role field ✅, CRUD ✅, dashboards ✅, login redirect ✅, authorization ✅, blocked direct-URL access ✅.
