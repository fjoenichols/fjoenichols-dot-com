+++
title = "Day 12: Improving Site Templates"
date = 2026-01-22T19:40:00-07:00
categories = ["21-Day Sprints"]
sprints = ["Learning Laravel"]
tags = ["laravel", "php", "tech"]
+++

### Summary
For Day 12 my goal is to improve the template for pages to be more brandable. 

Today's Definition of Done:
- Add hero_title, hero_subtitle, cta_text, and business_phone to the sites table
- Add a section in the dashboard to manage the new fields
- Add a "Hero" area on the home page

### Work Session
Since I am changing the database the first thing I had to do was a migration.

``` bash
$ php artisan make:migration add_brand_fields_to_sites_table

   INFO  Migration [database/migrations/2026_01_24_021927_add_brand_fields_to_sites_table.php] created successfully.
```

Then I created all the of the fields in the migration file
``` php 
public function up(): void
{
    Schema::table('sites', function (Blueprint $table) {
        $table->string('hero_title')->nullable();
        $table->string('hero_subtitle')->nullable();
        $table->string('cta_text')->nullable();
        $table->string('business_phone')->nullable();
    });
}
```

and finally I ran the migration.
``` bash
$ php artisan migrate

   INFO  Running migrations.

  2026_01_24_021927_add_brand_fields_to_sites_table ........................................................... 3.07ms DONE
```

After running the migration I was ready to move on to updating the edit page for sites.
``` php
<x-app-layout>
    <x-slot name="header">
        <div class="flex justify-between items-center">
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                Settings for: {{ $site->name }}
            </h2>
            <a href="{{ route('dashboard') }}" class="text-sm text-gray-500 hover:text-gray-700">
                &larr; Back to Shoots
            </a>
        </div>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            <form action="{{ route('sites.update', $site) }}" method="POST" class="space-y-8">
                @csrf
                @method('PUT')

                <div class="bg-white p-6 shadow sm:rounded-lg">
                    <h3 class="text-lg font-medium text-gray-900 mb-4 border-b pb-2 italic text-green-700">1. Basic Information</h3>
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                        <div>
                            <label class="block text-sm font-medium text-gray-700">Site Name</label>
                            <input type="text" name="name" value="{{ old('name', $site->name) }}" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:ring-green-500 focus:border-green-500">
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700">Subdomain (Slug)</label>
                            <input type="text" value="{{ $site->slug }}" class="mt-1 block w-full bg-gray-100 border-gray-300 rounded-md shadow-sm cursor-not-allowed" disabled title="Subdomains cannot be changed">
                            <p class="text-xs text-gray-400 mt-1">Subdomains are permanent to protect your SEO.</p>
                        </div>
                    </div>
                </div>

                <div class="bg-white p-6 shadow sm:rounded-lg">
                    <h3 class="text-lg font-medium text-gray-900 mb-4 border-b pb-2 italic text-green-700">2. Hero & Brand Identity</h3>
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                        <div class="col-span-1">
                            <label class="block text-sm font-medium text-gray-700">Hero Title</label>
                            <input type="text" name="hero_title" value="{{ old('hero_title', $site->hero_title) }}" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:ring-green-500 focus:border-green-500" placeholder="e.g. Expert Plumbing Services">
                        </div>
                        <div class="col-span-1">
                            <label class="block text-sm font-medium text-gray-700">Business Phone</label>
                            <input type="text" name="business_phone" value="{{ old('business_phone', $site->business_phone) }}" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:ring-green-500 focus:border-green-500" placeholder="e.g. 555-0123">
                        </div>
                        <div class="col-span-2">
                            <label class="block text-sm font-medium text-gray-700">Hero Subtitle</label>
                            <input type="text" name="hero_subtitle" value="{{ old('hero_subtitle', $site->hero_subtitle) }}" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:ring-green-500 focus:border-green-500" placeholder="e.g. Fast, reliable, and affordable plumbing.">
                        </div>
                        <div class="col-span-1">
                            <label class="block text-sm font-medium text-gray-700">Call to Action (Button Text)</label>
                            <input type="text" name="cta_text" value="{{ old('cta_text', $site->cta_text) }}" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:ring-green-500 focus:border-green-500" placeholder="e.g. Get a Free Quote">
                        </div>
                    </div>
                </div>

                <div class="bg-white p-6 shadow sm:rounded-lg">
                    <h3 class="text-lg font-medium text-gray-900 mb-4 border-b pb-2 italic text-green-700">3. Visuals & Metadata</h3>
                    <div class="grid grid-cols-1 gap-6">
                        <div>
                            <label class="block text-sm font-medium text-gray-700">Theme Color</label>
                            <select name="theme_color" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:ring-green-500 focus:border-green-500">
                                <option value="green" {{ $site->theme_color == 'green' ? 'selected' : '' }}>Green</option>
                                <option value="blue" {{ $site->theme_color == 'blue' ? 'selected' : '' }}>Blue</option>
                                <option value="purple" {{ $site->theme_color == 'purple' ? 'selected' : '' }}>Purple</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700">Site Description (SEO Meta)</label>
                            <textarea name="description" rows="3" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:ring-green-500 focus:border-green-500">{{ old('description', $site->description) }}</textarea>
                            <p class="text-xs text-gray-400 mt-1">This appears in search results and when sharing your link.</p>
                        </div>
                    </div>
                </div>

                <div class="flex items-center justify-end space-x-4">
                    @if(session('success'))
                        <span class="text-green-600 font-medium animate-pulse">Saved!</span>
                    @endif
                    <button type="submit" class="bg-green-600 text-white px-8 py-3 rounded-md font-bold text-lg hover:bg-green-700 shadow-lg transition transform hover:-translate-y-0.5">
                        Save All Settings
                    </button>
                </div>
            </form>
        </div>
    </div>
</x-app-layout>
```

Then I needed to update the SiteController to be able to capture the new fields.
``` php
public function update(Request $request, Site $site)
{
    if ($site->user_id !== auth()->id()) {
        abort(403);
    }

    $validated = $request->validate([
        'name' => 'required|string|max:255',
        'description' => 'nullable|string|max:1000',
        'theme_color' => 'required|string|in:blue,green,purple',
        'hero_title' => 'nullable|string|max:255',
        'hero_subtitle' => 'nullable|string|max:255',
        'cta_text' => 'nullable|string|max:255',
        'business_phone' => 'nullable|string|max:255',
    ]);

    $site->update($validated);

    return redirect()->route('dashboard')->with('success', 'Shoot updated successfully!');
}
```

Then the last thing I needed to do was update the home.blade.php file in tenant.
``` php
<div class="py-16 px-8 bg-gradient-to-br from-green-50 to-white rounded-2xl border border-green-100 mb-12">
    <div class="max-w-2xl">
        <h1 class="text-5xl font-extrabold text-gray-900 leading-tight mb-4">
            {{ $site->hero_title ?? $site->name }}
        </h1>
        <p class="text-xl text-gray-600 mb-8">
            {{ $site->hero_subtitle ?? $site->description }}
        </p>
        
        @if($site->cta_text)
            <a href="/contact" class="inline-block bg-green-600 text-white px-8 py-4 rounded-full font-bold text-lg hover:bg-green-700 transition">
                {{ $site->cta_text }}
            </a>
        @endif
    </div>
</div>

@if($site->business_phone)
    <div class="mt-12 py-6 border-t border-gray-200 text-center">
        <p class="text-gray-500 font-medium">Contact us today: <span class="text-green-600">{{ $site->business_phone }}</span></p>
    </div>
@endif
```

I though I was done after this but the updated fields weren't being saved to the database... Turns out..  I forgot to update the Site model.

After updating the Site model things worked as expected!

###  Recap 
The commit for todays session is [commit 5a64a8a](https://github.com/fjoenichols/RhizomeCMS/commit/5a64a8abb8a5e8b9b5f9ab060055f2bc64b586da).
