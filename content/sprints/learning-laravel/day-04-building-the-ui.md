+++
title = "Day 4: Building the UI"
date = 2026-01-14T19:45:00-07:00
categories = ["21-Day Sprints"]
sprints = ["Learning Laravel"]
tags = ["laravel", "php", "tech"]
+++

## Day 4: Building the UI
### Summary
Wednesdays are usually pretty busy for me since I go to church after work and don't get home until very near the kids bedtime so I decided to break today into 2 sessions. 

Going into session 1 I had the following goals:
- Set up the "Landlord" controller
- Create the route
- Ensure the Site model works with User relationship

### Work Session 1
Using `php artisan` again I created the `SiteController` controller. Not to be confused with `Tenant\SiteController` that I created yesterday. 

``` bash
$ php artisan make:controller SiteController

   INFO  Controller [app/Http/Controllers/SiteController.php] created successfully.
```

In SiteController I included the logic to save new sites. I called the method `store` and included data validation.

``` php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\Site;

class SiteController extends Controller
{
    public function store(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'slug' => 'required|string|unique:sites,slug|max:255',
        ]);

        $request->user()->sites()->create($validated);

        return redirect()->route('dashboard')->with('success', 'Site created successfully!');
    }
}
```

Then I added the POST route for `/sites` and while I was in the file I wrapped all the routes that require authentication inside a middleware group so I didn't have to require auth per route.

``` php
    Route::middleware(['auth', 'verified'])->group(function () {
        Route::view('dashboard', 'dashboard')->name('dashboard');
        Route::view('profile', 'profile')->name('profile');
        Route::post('/sites', [App\Http\Controllers\SiteController::class, 'store'])
            ->name('sites.store');
    });
```
As I wrapped up the first session I wanted to make sure that everything was working as expected so I decided to use my old friend tinker even though I only met her once before. 

``` bash
> $user = App\Models\User::first();
= App\Models\User {#6354
    id: 1,
    name: "Joe Nichols",
    email: "[redacted]",
    email_verified_at: null,
    #password: "[redacted]",
    #remember_token: null,
    created_at: "2026-01-13 06:17:10",
    updated_at: "2026-01-13 06:17:10",
  }

> $user->sites()->create(['name' => 'Tinker Test Site', 'slug' => 'tinker-test',]);
= App\Models\Site {#6890
    name: "Tinker Test Site",
    slug: "tinker-test",
    user_id: 1,
    updated_at: "2026-01-15 00:01:40",
    created_at: "2026-01-15 00:01:40",
    id: 2,
  }

> App\Models\Site::latest()->first();
= App\Models\Site {#6408
    id: 2,
    created_at: "2026-01-15 00:01:40",
    updated_at: "2026-01-15 00:01:40",
    user_id: 1,
    name: "Tinker Test Site",
    slug: "tinker-test",
    custom_domain: null,
  }
```

### Work Session 1 Recap 
I created a `SiteController` with the logic to store a new site and created the Route to post `/sites`. Then I verified I was able to create a site that had a user relationship. 

The commit for the first session is [commit b092ebd](https://github.com/fjoenichols/RhizomeCMS/commit/b092ebd704f1f1b21954efad1ae3995e2f3ff619).

--

For session 2 I had based my goals around the UI:
- Add a site from a form
- Show success and error messages
- Update the dashboard to display sites from the database

### Work Session 2
The first thing I did after coming back was edit the dahsboard in `resources/views/dashboard.blade.php`. Laravel Breeze uses tailwind by default so that was nice to just be able to jump in and start creating the UI.  I created a form that allowed users to create a new site, and below the form I added a "My Sites" section to show sites that belong to the user.

![Image of New Site Form](/images/learning-laravel/day-04-create-new-site-form.png)

Once I verified things looked right I added success and error messages above the form.

``` php
@if(session('success'))
    <div class="mb-4 p-4 bg-green-100 text-green-700 rounded border border-green-200">
        {{ session('success') }}
    </div>
@endif

@if($errors->any())
    <div class="mb-4 p-4 bg-red-100 text-red-700 rounded border border-red-200">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif
```

Then I created a new site and something weird happened. It worked... kind of.  The success message was just black text on white without the formatting I expected.  After a bit of google I found out I needed to run `npm run build` to get the Tailwind CSS JIT compiler to build in the green classes I hadn't used previously.  Apparently during development I could have left `npm run dev` going in a terminal and it would handle this on the fly for me.

![Successful Site Creation Form](/images/learning-laravel/day-04-create-new-rhizome-success.png)

Then I clicked a link on a newly created site to make sure it was still rendering json correctly.

``` json
{
  "site_name": "Another Site",
  "owner": "Joe Nichols",
  "message": "Welcome to the Rhizome."
}
```

Woo! Still works. 

One last thing before wrapping up for day. I decided to make sure slugs were always url-friendly to ensure I didn't end up with a slug that wouldn't be able to be resolved. 

Easy enough, I just needed to force slugs to lowercase and make them dash separted busing `Str::slug` from `Illuminate\Support\Str`.

``` php
$request->merge([
    'slug' => Str::slug($request->slug),
]);
```
### Recap
The second session for tonight was focused on the UI and getting a working form on the dashboard. So far working with Laravel has been really straight forward. I think my previous experience working with similar MVC or MVT frameworks like ruby-on-rails and django helps me understand the way things are working abit better.

The commit for this session is [commit 4d2724a](https://github.com/fjoenichols/RhizomeCMS/commit/4d2724a330f678ecc8113206355b339b5e7be68a).