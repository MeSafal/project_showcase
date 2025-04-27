# AdminRouteServiceProvider Documentation

Thsi file showcase the custom routes defination available in CMS
## Overview
The `AdminRouteServiceProvider` is a custom Laravel service provider that implements a macro-based routing system for administrative routes. It provides a standardized way to create CRUD and administrative routes with consistent URL patterns and naming conventions.

## Technical Implementation

### 1. Service Provider Registration

```php
// In config/app.php or AppServiceProvider.php
$this->app->register(AdminRouteServiceProvider::class);
```

The registration process:
1. Laravel's service container instantiates the AdminRouteServiceProvider
2. Calls the register() method (currently empty as we don't need to bind any services)
3. Calls the boot() method where our route macro is defined

### 2. Route Macro Definition

The provider defines a macro named 'adminResource' on Laravel's Route facade:

```php
Route::macro('adminResource', function (string $name, string $controller, array $options = []) {
    // Implementation
});
```

#### How Macros Work in Laravel:
- Macros extend PHP classes at runtime
- They're stored in the Route facade's macro collection
- When called, they're executed in the context of the Route facade

### 3. Route Configuration Structure

Default route configuration:
```php
$defaultRoutes = [
    'index' => ['get', 'index'],          // GET admin/resource/index
    'create' => ['get', 'form'],          // GET admin/resource/create
    'store' => ['post', 'save'],          // POST resource/store
    'edit' => ['get', 'form', '{id}'],    // GET admin/resource/edit/{id}
    'update' => ['post', 'save', '{id}'], // POST resource/update/{id}
    'view' => ['get', 'view', '{id}'],    // GET admin/resource/view/{id}
    'delete' => ['post', 'delete', '{id}'], // POST resource/delete/{id}
    'alias' => ['post', 'alias', '{id}'], // POST resource/alias/{id}
    'publish' => ['post', 'publish', '{id}/{publish}'], // POST resource/publish/{id}/{publish}
    'updateOrder' => ['post', 'updateOrder'] // POST resource/updateOrder
];
```

Each route configuration is an array with:
- Index 0: HTTP method (get/post)
- Index 1: Controller method name
- Index 2: (Optional) URL parameters

### 4. URL and Route Name Generation

The provider uses two key variables:
```php
$routeName = $options['route_name'] ?? $name;  // For route names and POST URLs
$routeUrl = 'admin/' . $name;                  // For GET URLs
```

URL Generation Rules:
- GET requests: Uses `$routeUrl` (includes 'admin' prefix)
- POST requests: Uses `$routeName` (no prefix)

Example:
```php
// For a resource named 'articles':
GET admin/articles/index  -> articles.index
POST articles/store      -> articles.store
```

### 5. Custom Route Configuration

You can customize routes in three ways:

1. **Add New Routes:**
```php
Route::adminResource('buttons', ButtonsController::class, [
    'routes' => [
        'getTemplateData' => ['get', 'getTemplateData', '{id}']
    ]
]);
```

2. **Disable Routes:**
```php
Route::adminResource('templates', TemplatesController::class, [
    'routes' => [
        'create' => false,
        'store' => false
    ]
]);
```

3. **Override Default Parameters:**
```php
Route::adminResource('pages', PagesController::class, [
    'routes' => [
        'view' => ['get', 'view', '{alias}']  // Change {id} to {alias}
    ]
]);
```

### 6. Route Registration Process

For each route:
1. Merges default and custom routes
2. Checks if route is enabled (not false)
3. Extracts method, action, and parameters
4. Determines URL based on HTTP method
5. Registers route with Laravel's router
6. Assigns route name

## Usage Examples

### Basic Usage
```php
Route::adminResource('articles', ArticlesController::class);
```
This will generate
```php
Route::prefix('admin/articles')->name('articles.')->group(function () {
    Route::get('/', [ArticlesController::class, 'index'])->name('index');
    Route::get('/create', [ArticlesController::class, 'form'])->name('create');
    Route::post('/store', [ArticlesController::class, 'save'])->name('store');
    Route::get('/edit/{id}', [ArticlesController::class, 'form'])->name('edit');
    Route::post('/update/{id}', [ArticlesController::class, 'save'])->name('update');
    Route::get('/view/{id}', [ArticlesController::class, 'view'])->name('view');
    Route::post('/delete/{id}', [ArticlesController::class, 'delete'])->name('delete');
    Route::post('/alias/{id}', [ArticlesController::class, 'alias'])->name('alias');
    Route::post('/publish/{id}/{publish}', [ArticlesController::class, 'publish'])->name('publish');
    Route::post('/updateOrder', [ArticlesController::class, 'updateOrder'])->name('updateOrder');
});
```

### With Custom Routes
```php
Route::adminResource('navigation', NavigationsController::class, [
    'routes' => [
        'fetch-content' => ['get', 'fetchContent'],
        'getData' => ['get', 'getData', '{id}']
    ]
]);
```

### With Disabled Routes
```php
Route::adminResource('templates', TemplatesController::class, [
    'routes' => [
        'create' => false,
        'store' => false,
        'getData' => ['get', 'getData', '{id}']
    ]
]);
```

### With default routes defination
You can still define the route in default laravel style with combination of above --
like disabled route, custom routes within the adminResource and laravel style route
```php
Route::adminResource('templates', TemplatesController::class, [
    'routes' => [
        'create' => false,
        'store' => false,
        'getData' => ['get', 'getData', '{id}']
    ]
]);

Route::get('/admin/templates/viewActive/{id}', [TemplateController::class, 'viewActive'])->name('templates.viewActive');
```

## Controller Method Requirements

Your controller should implement these methods to match the default routes:
- index()
- form()  // For both create and edit
- save()  // For both store and update
- view()
- delete()
- alias()
- publish()
- updateOrder()

## Benefits

1. **Standardization:**
   - Consistent URL patterns
   - Consistent route naming
   - Predictable controller method names

2. **Maintainability:**
   - Centralized route configuration
   - Easy to modify globally
   - DRY principle applied

3. **Flexibility:**
   - Custom routes support
   - Route disabling
   - Parameter customization

4. **Security:**
   - Proper HTTP method enforcement
   - Admin prefix for GET routes
   - Consistent authorization points

## Common Patterns

1. **URL Structure:**
   - GET routes: admin/{resource}/{action}/{parameters?}
   - POST routes: {resource}/{action}/{parameters?}

2. **Route Names:**
   - Always: {resource}.{action}
   - Example: articles.index, articles.store

3. **Controller Methods:**
   - CRUD operations use consistent method names
   - Custom actions can be added as needed 


### Note for adminResource

The custom routes will create the route with the same name as provided in the second argument.

- Example: **Index 1**  
  The controller method name and the route target the same method in the controller for simplicity.

> Happy Coding 🙂🙂🙂  
> Contact me if you have any queries.
