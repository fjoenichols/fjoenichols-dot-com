+++
title = "Day 6: Templating & Dynamic Styling"
date = 2026-01-16T19:00:00-07:00
categories = ["21-Day Sprints"]
sprints = ["Learning Laravel"]
tags = ["laravel", "php", "tech"]
+++

## Day 6: Templating & Dynamic Styling
### Summary
Today's Definition of Done:
- Template Creation
- Primary colors changed based on `theme_color`.
- `name` and `description` rendered in HTML
- `Tenant/SiteController` returns html instead of json.

### Work Session
In order to handle tenant homepages I created a new view at `resources/views/tenant/home.blade.php`.

``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ $site->name }}</title>
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

    <div class="min-h-screen flex flex-col items-center justify-center">
        <div class="max-w-2xl w-full bg-white shadow-xl rounded-lg overflow-hidden border-t-8 {{ $themeClasses }}">
            <div class="p-8">
                <h1 class="text-4xl font-extrabold text-gray-900 mb-4">
                    {{ $site->name }}
                </h1>
                
                <p class="text-lg text-gray-600 leading-relaxed mb-6">
                    {{ $site->description ?? 'This shoot is just beginning to grow.' }}
                </p>

                <div class="pt-6 border-t border-gray-100">
                    <p class="text-sm text-gray-500 italic">
                        Sprouted by {{ $site->user->name }}
                    </p>
                </div>
            </div>
        </div>
    </div>
</body>
</html>
```

Then I updated `Tenant\SiteController` to use the new view I just created. 

``` php
public function index()
{
    $site = app('current_site');

    return view('tenant.home', compact('site'));
}
```

After that I made sure `npm run dev` was going in my terminal to compile what needed to be compiled at page load and then tried to load "Joes Site". 

![Joes Site w/ Green Styling](/images/learning-laravel/day-06-dynamic-style.png)

And the last thing I did for the day was add a footer at the bottom of the page saying "Powered by RhizomeCMS."

``` html
<div class="pt-6 border-t border-gray-100">
    <p class="text-sm text-gray-500 italic">
        Powered by 
        <a href="{{ 'http://' . config('app.url') . '/register' }}" class="font-semibold hover:text-gray-600 transition">
        RhizomeCMS
        </a> | Sprouted by {{ $site->user->name }}
    </p>
</div>
```

###  Recap 
Today I created a new view to act a template for Shoots and then updated the `Tenant\SiteController` to use that view instead of just returning html. 

The commit for todays session is [commit 83fc030](https://github.com/fjoenichols/RhizomeCMS/commit/83fc0309dbfd8364f34bf57ae10d3948182b9b2b).
