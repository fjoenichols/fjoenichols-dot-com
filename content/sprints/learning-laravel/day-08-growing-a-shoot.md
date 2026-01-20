+++
title = "Day 8: Growing a Shoot"
date = 2026-01-19T19:00:00-07:00
categories = ["21-Day Sprints"]
sprints = ["Learning Laravel"]
tags = ["laravel", "php", "tech"]
+++

### Summary
For Day 8 the goal is to turn a Shoot into a functional multi-page site. 

Today's Definition of Done:
- `pages` table linked to `sites` table via `site_id`
- Site model can have many Page models
- Routing for pages works
- Sub-pages can be viewed

### Work Session
The first thing I did today is create a model using artisan for the Page model.
``` bash
$ php artisan make:model Page -m

   INFO  Model [app/Models/Page.php] created successfully.

   INFO  Migration [database/migrations/2026_01_20_033243_create_pages_table.php] created successfully.
```

Then I edited the migration to give the `pages` table the right schema.
``` php
public function up(): void
{
    Schema::create('pages', function (Blueprint $table) {
        $table->id();
        $table->foreignId('site_id')->constrained()->onDelete('cascade');
        $table->string('title');
        $table->string('slug');
        $table->text('content');
        $table->timestamps();

        $table->unique(['site_id', 'slug']); 
    });
}
```

And of course use artisan to run the migration.
``` bash
$ php artisan migrate

   INFO  Running migrations.

  2026_01_20_033243_create_pages_table ................................................................... 8.15ms DONE
```

With that the model is done, and then I could move on to establishing the relationship between sites and pages. 

In the Site model I told the site it can have multiple pages.
``` php
public function pages()
{
    return $this->hasMany(Page::class);
}
```

And then in the Page model I told it that sites belong to pages. 
``` php
protected $fillable = ['site_id', 'title', 'slug', 'content'];

public function site()
{
    return $this->belongsTo(Site::class);
}
```

The next thing I did was make the pages routable. 
``` php 
Route::get('/{page_slug}', [App\Http\Controllers\Tenant\SiteController::class, 'showPage']);
```

Then I edited the `Tenant\SiteController` logic so it could find these pages. 
``` php
public function showPage($tenant, $page_slug)
{
    $site = app('current_site');
    $page = $site->pages()->where('slug', $page_slug)->firstOrFail();

    return view('tenant.page', compact('site', 'page'));
}
```

And finally I created a simple view for the page. I first just copied the home.blade.php view and renamed it page.blade.php swapped out the `name` and `description` variables for `title` and `content`.

``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ $page->title }} | {{ $site->name }}</title>
    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
<body class="bg-gray-100 font-sans antialiased">

    @php
        $colors = [
            'blue' => 'bg-blue-600 border-blue-800',
            'green' => 'bg-green-600 border-green-800',
            'purple' => 'bg-purple-600 border-purple-800',
        ];
        $themeClasses = $colors[$site->theme_color] ?? $colors['blue'];
    @endphp

    <div class="min-h-screen flex flex-col items-center py-12">
        <div class="max-w-3xl w-full bg-white shadow-xl rounded-lg overflow-hidden border-t-8 {{ $themeClasses }}">
            <div class="p-8">
                <nav class="mb-8">
                    <a href="/" class="text-sm font-semibold text-gray-500 hover:text-gray-800">
                        ← {{ $site->name }} Home
                    </a>
                </nav>

                <h1 class="text-4xl font-extrabold text-gray-900 mb-6">
                    {{ $page->title }}
                </h1>
                
                <div class="prose max-w-none text-lg text-gray-600 leading-relaxed">
                    {!! nl2br(e($page->content)) !!}
                </div>

                <footer class="mt-12 pt-6 border-t border-gray-100 text-sm text-gray-500 italic">
                    Powered by 
                    <a href="{{ 'http://' . config('app.url') . '/register' }}" class="font-semibold hover:text-gray-600 transition">
                        RhizomeCMS
                    </a> | Sprouted by {{ $site->user->name }}
                </footer>
            </div>
        </div>
    </div>
</body>
</html>
```

Since I haven't implemented full CRUD functionality in the UI yet I'll use tinker to insert a page record in the database.

``` bash
s$ php artisan tinker
Psy Shell v0.12.18 (PHP 8.4.1 — cli) by Justin Hileman
New PHP manual is available (latest: 3.0.1). Update with `doc --update-manual`
> $site = App\Models\Site::where('slug', 'joes-site')->first();
= App\Models\Site {#6369
    id: 9,
    created_at: "2026-01-20 03:48:16",
    updated_at: "2026-01-20 03:48:16",
    user_id: 1,
    name: "Joes Site",
    slug: "joes-site",
    custom_domain: null,
    description: "This is Joes Site",
    theme_color: "green",
  }

> $site->pages()->create(['title' => 'About Us', 'slug' => 'about', 'content' => 'This is the story of how Joe started his Rhizome.']);
= App\Models\Page {#6863
    title: "About Us",
    slug: "about",
    content: "This is the story of how Joe started his Rhizome.",
    site_id: 9,
    updated_at: "2026-01-20 03:56:32",
    created_at: "2026-01-20 03:56:32",
    id: 1,
  }
```

After this I tried to go to `joes-site.rhizomecms.test:8000/about` expecting an about page, but I got the regular Joes Site page.

To try to debug this I added a die and dump above the `showPage` method in `Tenant\SiteController`. This should let me see if the method is even getting called.
``` php
dd("You are trying to view the page: " . $page_slug);
```

After refreshing the about page I don't see the `dd` text so it seems like the route is never getting matched. 

Then I checked my routes using `route:list`

``` bash
$ php artisan route:list

  GET|HEAD  {tenant}.rhizomecms.test/ .................................................... Tenant\SiteController@index
  GET|HEAD  rhizomecms.test/ .........................................................................................
  GET|HEAD  rhizomecms.test/confirm-password ........................................................ password.confirm
  GET|HEAD  rhizomecms.test/dashboard ...................................................................... dashboard
  GET|HEAD  rhizomecms.test/forgot-password ......................................................... password.request
  GET|HEAD  livewire/livewire.js ......................... Livewire\Mechanisms › FrontendAssets@returnJavaScriptAsFile
  GET|HEAD  livewire/livewire.min.js.map ................................... Livewire\Mechanisms › FrontendAssets@maps
  GET|HEAD  livewire/preview-file/{filename} livewire.preview-file › Livewire\Features › FilePreviewController@handle
  POST      livewire/update ...................... livewire.update › Livewire\Mechanisms › HandleRequests@handleUpdate
  POST      livewire/upload-file .............. livewire.upload-file › Livewire\Features › FileUploadController@handle
  GET|HEAD  rhizomecms.test/login .............................................................................. login
  GET|HEAD  rhizomecms.test/profile .......................................................................... profile
  GET|HEAD  rhizomecms.test/register ........................................................................ register
  GET|HEAD  rhizomecms.test/reset-password/{token} .................................................... password.reset
  POST      rhizomecms.test/sites ................................................. sites.store › SiteController@store
  PUT       rhizomecms.test/sites/{site} ........................................ sites.update › SiteController@update
  DELETE    rhizomecms.test/sites/{site} ...................................... sites.destroy › SiteController@destroy
  GET|HEAD  rhizomecms.test/sites/{site}/edit ....................................... sites.edit › SiteController@edit
  GET|HEAD  storage/{path} ............................................................................. storage.local
  GET|HEAD  up .......................................................................................................
  GET|HEAD  rhizomecms.test/verify-email ......................................................... verification.notice
  GET|HEAD  rhizomecms.test/verify-email/{id}/{hash} ................ verification.verify › Auth\VerifyEmailController
  GET|HEAD  {tenant}.rhizomecms.test/{page_slug} ...................................... Tenant\SiteController@showPage

                                                                                                   Showing [23] routes
```

Everything there looks correct.  The last thing I did is check the page view and the home view and somehow I ended up with them being identical. When I was pasting the code block for this post I think I swapped them and ended up with 2 of the home view.

After fixing my page view and refreshing it finally worked! 

![Joes Site about page](/images/learning-laravel/day-08-about-page.png)

###  Recap 
Today was going really smooth until I decided to take the long way around and cause an error for myself to troubleshoot.  The Model, Controller, Route, View workflow is a pretty easy pattern to follow so continuing to work this way should be good.  

The commit for todays session is [commit fe53035](https://github.com/fjoenichols/RhizomeCMS/commit/fe53035b8dd70b148fa7e536dfeeb50e5de72716).
