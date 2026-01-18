+++
title = "Day 7: Completing the CRUD Cycle"
date = 2026-01-17T16:00:00-07:00
categories = ["21-Day Sprints"]
sprints = ["Learning Laravel"]
tags = ["laravel", "php", "tech"]
+++

### Summary
Today's Definition of Done:
- Shoot owner can edit description and color
- Shoot owner can destroy a shoot after confirmation
- Edit and Delete controls visible from dashboard

### Work Session
The first thing I did for Day 7 is start working on the SiteController to add the `edit` and `update` methods. 

``` php
public function edit(Site $site)
{
    if ($site->user_id !== auth()->id()) {
        abort(403);
    }

    return view('sites.edit', compact('site'));
}

public function update(Request $request, Site $site)
{
    if ($site->user_id !== auth()->id()) {
    abort(403);
    }

    $validated = $request->validate([
        'name' => 'required|string|max:255',
        'description' => 'nullable|string|max:1000',
        'theme_color' => 'required|string|in:blue,green,purple',
    ]);

    $site->update($validated);

    return redirect()->route('dashboard')->with('success', 'Shoot updated successfully!');
}
```

Then I added the destroy method just to get everything in place in the controller. 

``` php
public function destroy(Site $site)
{
    if ($site->user_id !== auth()->id()) {
    abort(403);
    }

    $site->delete();

    return redirect()->route('dashboard')->with('success', 'Shoot pruned successfully.');
}
```

After that I added the routes to `routes/web.php`.

``` php
Route::get('/sites/{site}/edit', [SiteController::class, 'edit'])->name('sites.edit');
Route::put('/sites/{site}', [SiteController::class, 'update'])->name('sites.update');
Route::delete('/sites/{site}', [SiteController::class, 'destroy'])->name('sites.destroy');
```

Now that the controller and the routes were taken care of I needed to update the dashboard template replacing the simple "Your Shoots" section with the buttons needed.

``` html
<ul class="space-y-4">
    @foreach(auth()->user()->sites as $site)
    <li class="p-6 bg-white rounded-lg border shadow-sm flex items-center justify-between">
        <div>
            <h4 class="text-lg font-bold text-gray-900">{{ $site->name }}</h4>
            <a href="http://{{ $site->slug }}.rhizomecms.test:8000" class="text-sm text-blue-600 hover:underline" target="_blank">
                {{ $site->slug }}.rhizomecms.test
            </a>
        </div>

        <div class="flex items-center space-x-4">
            <a href="{{ route('sites.edit', $site) }}" class="text-sm font-medium text-gray-600 hover:text-gray-900">
                Edit
            </a>

            <form action="{{ route('sites.destroy', $site) }}" method="POST" onsubmit="return confirm('Are you sure you want to prune this shoot?');">
                @csrf
                @method('DELETE')
                <button type="submit" class="text-sm font-medium text-red-600 hover:text-red-900">
                    Prune
                </button>
            </form>
        </div>
    </li>
    @endforeach
</ul>
```

Then I created `views/sites/edit.blade.php` for my edit page. 
``` php
<x-app-layout>
    <x-slot name="header">
        <h2 class="font-semibold text-xl text-gray-800 leading-tight">
            Tending to: {{ $site->name }}
        </h2>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg p-6">
                <form action="{{ route('sites.update', $site) }}" method="POST" class="space-y-6">
                    @csrf
                    @method('PUT')

                    <div>
                        <label class="block text-sm font-medium text-gray-700">Shoot Name</label>
                        <input type="text" name="name" value="{{ $site->name }}" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm" required>
                    </div>

                    <div>
                        <label class="block text-sm font-medium text-gray-700">Shoot Description</label>
                        <textarea name="description" rows="4" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm">{{ $site->description }}</textarea>
                    </div>

                    <div>
                        <label class="block text-sm font-medium text-gray-700">Theme Color</label>
                        <select name="theme_color" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm">
                            <option value="blue" {{ $site->theme_color == 'blue' ? 'selected' : '' }}>Blue</option>
                            <option value="green" {{ $site->theme_color == 'green' ? 'selected' : '' }}>Green</option>
                            <option value="purple" {{ $site->theme_color == 'purple' ? 'selected' : '' }}>Purple</option>
                        </select>
                    </div>

                    <div class="flex items-center justify-between pt-4 border-t">
                        <a href="{{ route('dashboard') }}" class="text-sm text-gray-600 hover:underline">Cancel</a>
                        <button type="submit" class="bg-blue-600 text-white px-4 py-2 rounded-md hover:bg-blue-700">
                            Save Changes
                        </button>
                    </div>
                </form>
            </div>
        </div>
    </div>
</x-app-layout>
```

At this point my dashboard was looking correct, but when I tried to "Edit" or "Prune" I would get an error page with `Target class [SiteController] does not exist.` 

After a bit of digging I found the issue in my route. I didn't specify the full path to my SiteController.  Here I could have added a `use` for the Controller at the top of the route file, but since I have `SiteController` and `Tenant\SiteController` I decided to just stick with the full path. 

``` php 
Route::get('/sites/{site}/edit', [App\Http\Controllers\SiteController::class, 'edit'])->name('sites.edit');
Route::put('/sites/{site}', [App\Http\Controllers\SiteController::class, 'update'])->name('sites.update');
Route::delete('/sites/{site}', [App\Http\Controllers\SiteController::class, 'destroy'])->name('sites.destroy');
```

Now "Edit" and "Prune" work correctly.


###  Recap 
Today I was able to round out the CRUD functionality from the dashboard so now a user can create a new shoot, edit the shoot, and prune the shoot if it is no longer needed.  

The commit for todays session is [commit 6091fd3](https://github.com/fjoenichols/RhizomeCMS/commit/6091fd38fcb7cf3da2165c5f0cceeb398abb5ae7).
