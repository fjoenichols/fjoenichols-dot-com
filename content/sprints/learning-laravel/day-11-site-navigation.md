+++
title = "Day 11: Site Navigation"
date = 2026-01-22T19:40:00-07:00
categories = ["21-Day Sprints"]
sprints = ["Learning Laravel"]
tags = ["laravel", "php", "tech"]
+++

### Summary
For Day 11 my goal is to get dynamic navigation working for the pages on a site. 

Today's Definition of Done:
- Links dynmically created for page
- Routing between pages works
- Active link is a different color

### Work Session
I reinstalled my PC yesterday so I spent a lot of time today getting my PC set back up with the dev environments for both RhizomeCMS and my Hugo Site. 

After I got everything set back up I was able to get to work on my day 11 objectives starting with updating the Tenant Controller to get a list of all sibling pages.
``` php
class SiteController extends Controller
{
    public function index()
    {
        $site = app('current_site');
        $pages = $site->pages()->get();

        return view('tenant.home', compact('site'));
    }

    public function showPage($tenant, $page_slug)
    {
        $site = app('current_site');
        $pages = $site->pages()->get();
        $page = $site->pages()->where('slug', $page_slug)->firstOrFail();

        return view('tenant.page', compact('site', 'page', 'pages'));
    }
}
```

Then I needed to update `resources/views/tenant/home.blade.php` and `resources/views/tenant/page.blade.php` to include the nav links at the top.
``` html
        <nav class="flex items-center space-x-6 mb-8 pb-4 border-b border-gray-100">
            <a href="/" 
            class="text-sm font-bold {{ request()->is('/') ? 'text-green-600' : 'text-gray-400 hover:text-gray-600' }}">
                Home
            </a>

            @foreach($pages as $p)
                <a href="/{{ $p->slug }}" 
                class="text-sm font-bold {{ request()->is($p->slug) ? 'text-green-600' : 'text-gray-900' }}">
                    {{ $p->title }}
                </a>
            @endforeach
        </nav>
```
And this is the result...

![Home Page w/ Menu](/images/learning-laravel/day-11-site-navigate-home.png)
![About Page w/ Menu](/images/learning-laravel/day-11-site-navigate-about.png)

###  Recap 
I think the most frustrating part about today was getting everything set back up again. Laravel is becoming pretty easy to use now so I think I can scope more work for future days and get a marketable mvp ready in the next 10 days. 

The commit for todays session is [commit a02479a](https://github.com/fjoenichols/RhizomeCMS/commit/a02479af6002444e8a2a01bdcadc987b53470261).
