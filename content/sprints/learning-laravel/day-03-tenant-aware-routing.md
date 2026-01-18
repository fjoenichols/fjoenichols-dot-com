+++
title = "Day 3: Tenant Aware Routing"
date = 2026-01-13T22:45:00-07:00
categories = ["21-Day Sprints"]
sprints = ["Learning Laravel"]
tags = ["laravel", "php", "tech"]
+++

### Summary
Going into today's session I had the following goals:
- Middleware correctly pulls sudomain from host string
- Middleware throws 404 if no slug in database
- Navigating to mountainlotus.rhizomecms.test displays site name

As part of the setup I added some test domains to /etc/hosts so I could work with them and not have to mess with a dns server. 

```
127.0.0.1   rhizomecms.test
127.0.0.1   mountainlotus.rhizomecms.test
127.0.0.1   another-site.rhizomecms.test
```

### The Work Session
Using `php artisan` again I created the `IdentifyTenant` middleware.

```
$ php artisan make:middleware IdentifyTenant

   INFO  Middleware [app/Http/Middleware/IdentifyTenant.php] created successfully.
```

Again, I love that the console output gives the path to file file that was created because I needed to edit it. 

In the handle function I added variables for `host`, `subdomain`, and `site`. This is also when I learned that throwing a 404 for slugs that didn't exist in the database was going to be really simple because `firstOrFail()` does exactly that!

``` php
class IdentifyTenant
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        $host = $request->getHost();
        $subdomain = explode('.', $host)[0];

        $site = \App\Models\Site::where('slug', $subdomain)->firstOrFail();

        app()->instance('current_site', $site);
        
        return $next($request);
    }
}

```

After modifying the `IdentifyTenant` middleware I headed to `bootstrap/app.php` to register it. 

``` php
    ->withMiddleware(function (Middleware $middleware): void {
        $middleware->alias([
            'identify.tenant' => \App\Http\Middleware\IdentifyTenant::class,
        ]);
    })
```

Then the last step was to set up two route groups. The first route group was just to wrap the existing routes that were there. The second route group was to handle wildcard routing based on tenant. 

``` php
Route::domain('{tenant}.rhizomecms.test')
    ->middleware(['identify.tenant'])
    ->group(function () {
        Route::get('/', function () {
            $site = app('current_site');
            return "Welcome to " . $site->name;
        });
    });

Route::domain('rhizomecms.test')->group(function () {
    Route::view('/', 'welcome');

    Route::view('dashboard', 'dashboard')
        ->middleware(['auth', 'verified'])
        ->name('dashboard');

    Route::view('profile', 'profile')
        ->middleware(['auth'])
        ->name('profile');

    require __DIR__.'/auth.php';
});
```

After all this I was finally ready to test that subdomain routing worked, but first I had to start the dev server using `php artisan`.

``` bash
$ php artisan serve --host=0.0.0.0

   INFO  Server running on [http://0.0.0.0:8000].
```

And then I was finally ready to test...

``` bash
$ curl mountainlotus.rhizomecms.test:8000
Welcome to Mountain Lotus Digital
```

Success!

Then I tested that subdomains that didn't actually exist would throw a 404. 
``` bash
$ curl another-site.rhizomecms.test:8000 -I
HTTP/1.1 404 Not Found
```

Success again!

All of this only took me about 30 minutes to walk through so I decided to see if I could implement a tenant specific home page using a controller in the remaining time. 

So back to `php artisan` to generate the `SiteController`
``` bash
$ php artisan make:controller Tenant/SiteController

   INFO  Controller [app/Http/Controllers/Tenant/SiteController.php] created successfully.
```

Then into the site controller to create the index logic.

``` php
class SiteController extends Controller
{
    public function index()
    {
        $site = app('currentl_site');
        return response()->json)[
            'site_name' => $site->name,
            'owner' => $site->user->name,
            'message' => "Welcome to the Rhizome."
        ]);
    }
}
```

And finally back to `routes/web.php` to tell the `/` route to use the new `SiteController` controller.

Trying to load the page `404` was displayed for slugs that didn't have tenants. The breeze login page worked for the standard URL, but I had a blank page when trying to access `mountainlotus.rhizomecms.test:8000`.  It took a bit of troubleshooting but finally I found that I had mistakenly nested the new route into the old route . 

``` php
Route::domain('{tenant}.rhizomecms.test')
    ->middleware(['identify.tenant'])
    ->group(function () {
        Route::get('/', function () {
            Route::get('/', [App\Http\Controllers\Tenant\SiteController::class, 'index']);
        });
    });
```
I fixed this issue and hit refresh on the page and... ANOTHER ERROR!

```
ErrorException
app/Http/Controllers/Tenant/SiteController.php:15
Attempt to read property "name" on null
```

Turns out I never updated my Site model with a function to find the site owner. 

``` php
class Site extends Model
{
    protected $fillable = [
        'name',
        'slug',
        'custom_domain',
    ];

    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

And I refresh the page to yet another error. This time a namespace collision.

``` 
TypeError
app/Models/Site.php:17
App\Models\Site::user(): Return value must be of type App\Models\BelongsTo, Illuminate\Database\Eloquent\Relations\BelongsTo returned
```

I can fix this by just using the correct `BelongsTo` at the top of the Site model though...

```
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Site extends Model
```

Refresh the page one more time and...

```
{"site_name":"Mountain Lotus Digital","owner":"Joe Nichols","message":"Welcome to the Rhizome."}
```

Success!

### Recap
To recap I learned to create the IdentifyTenant middleware, register middleware, and create new routes and route groups which is what I set out to do.  Then I decided I had some time to implement the controller also and ran in to a chain of errors, but I was able to work through them eventually implementing a `user(): BelongsTo` function in the Site model and creating a controller to return unique results per tenant. 

Overall I went a little bit over an hour at 1 hour 15 minutes, but I'd rather be a little over than a little under and get behind. 

Todays code commit is [commit 6288eb4](https://github.com/fjoenichols/RhizomeCMS/commit/6288eb41e8ce8534ce5784413b00ce1d62702072).