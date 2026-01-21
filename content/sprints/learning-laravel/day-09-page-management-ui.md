+++
title = "Day 9: Page Management UI"
date = 2026-01-20T19:00:00-07:00
categories = ["21-Day Sprints"]
sprints = ["Learning Laravel"]
tags = ["laravel", "php", "tech"]
+++

### Summary
For Day 9 the goal was to turn allow users to edit their pages through the dashboard.. 

Today's Definition of Done:
- Can access manage pages dashboard from shoot list
- All pages for a site listed in index view
- Create page form in the dashboard
- Can create a page and see it live 

### Work Session
The first thing I did was add a button to my main list to `Manage Pages`.

``` html
<div class="flex items-center space-x-4">
    <a href="{{ route('sites.pages.index', $site) }}" class="text-sm font-medium text-green-600 hover:text-green-900">
        Manage Pages
    </a>
    </div>
```

Then I updated the route for nested resources to handle the pages belonging to a site.

``` php
Route::resource('sites.pages', App\Http\Controllers\PageController::class);
```

Next I created a PageController 
``` bash
$ php artisan make:controller PageController --resource

   INFO  Controller [app/Http/Controllers/PageController.php] created successfully.
```

With the index, create and store methods
``` php
public function index(Site $site)
{
    // Ensure the user owns this site
    if ($site->user_id !== auth()->id()) abort(403);

    $pages = $site->pages;
    return view('pages.index', compact('site', 'pages'));
}

public function create(Site $site)
{
    return view('pages.create', compact('site'));
}

public function store(Request $request, Site $site)
{
    $validated = $request->validate([
        'title' => 'required|string|max:255',
        'slug' => 'required|string|max:255',
        'content' => 'required|string',
    ]);

    $site->pages()->create($validated);

    return redirect()->route('sites.pages.index', $site)->with('success', 'Page sprouted!');
}
```

And after that I created the views for both create and view pages.

`index.blade.php`
``` html
<x-app-layout>
    <x-slot name="header">
        <div class="flex justify-between items-center">
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                Pages for: {{ $site->name }}
            </h2>
            <a href="{{ route('sites.pages.create', $site) }}" class="bg-green-600 text-white px-4 py-2 rounded-md text-sm hover:bg-green-700">
                + Sprout New Page
            </a>
        </div>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg p-6">
                @if($pages->isEmpty())
                    <p class="text-gray-500 text-center py-8">No pages sprouted yet. Start by creating a home or contact page!</p>
                @else
                    <ul class="divide-y divide-gray-200">
                        @foreach($pages as $page)
                            <li class="py-4 flex justify-between items-center">
                                <div>
                                    <h3 class="text-lg font-bold text-gray-900">{{ $page->title }}</h3>
                                    <p class="text-sm text-gray-500">Slug: /{{ $page->slug }}</p>
                                </div>
                                <div class="flex space-x-4">
                                    <a href="http://{{ $site->slug }}.rhizomecms.test:8000/{{ $page->slug }}" target="_blank" class="text-blue-600 hover:underline text-sm">View Live</a>
                                    </div>
                            </li>
                        @endforeach
                    </ul>
                @endif
                <div class="mt-6 border-t pt-4">
                    <a href="{{ route('dashboard') }}" class="text-sm text-gray-600 hover:underline">← Back to Shoots</a>
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```

`create.blade.php`
``` html
<x-app-layout>
    <x-slot name="header">
        <h2 class="font-semibold text-xl text-gray-800 leading-tight">
            Sprout New Page for {{ $site->name }}
        </h2>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg p-6">
                <form action="{{ route('sites.pages.store', $site) }}" method="POST" class="space-y-6">
                    @csrf
                    
                    <div>
                        <label class="block text-sm font-medium text-gray-700">Page Title</label>
                        <input type="text" name="title" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm" placeholder="e.g. Contact Us" required>
                    </div>

                    <div>
                        <label class="block text-sm font-medium text-gray-700">Slug</label>
                        <input type="text" name="slug" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm" placeholder="e.g. contact" required>
                        <p class="mt-1 text-xs text-gray-500">This determines the URL: {{ $site->slug }}.rhizomecms.test/slug</p>
                    </div>

                    <div>
                        <label class="block text-sm font-medium text-gray-700">Page Content</label>
                        <textarea name="content" rows="10" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm" placeholder="Tell your story here..." required></textarea>
                    </div>

                    <div class="flex items-center justify-between pt-4 border-t">
                        <a href="{{ route('sites.pages.index', $site) }}" class="text-sm text-gray-600 hover:underline">Cancel</a>
                        <button type="submit" class="bg-green-600 text-white px-6 py-2 rounded-md hover:bg-green-700 font-bold">
                            Sprout Page
                        </button>
                    </div>
                </form>
            </div>
        </div>
    </div>
</x-app-layout>
```

After implementing the views I tried to load a "Manage Page" but I ended uip with an error `Class "App\Http\Controllers\Site" does not exist"`. This is because I didn't implement the Site model and Page model inside the controller with `use`. 

And then finally it worked! 

![Create new pages](/images/learning-laravel/day-09-create-new-pages.png)

###  Recap 
I felt like I was really getting the hang of working with Laravel today. I just hit one minor snag and immediately realize what I had done wrong.

The commit for todays session is [commit 6db5c4f](https://github.com/fjoenichols/RhizomeCMS/commit/6db5c4fd49996f781ac99595baa1fdd54bcb78d1).
