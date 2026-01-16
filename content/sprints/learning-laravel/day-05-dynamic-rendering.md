+++
title = "Day 5: Dynamic Rendering"
date = 2026-01-15T19:45:00-07:00
categories = ["21-Day Sprints"]
sprints = ["Learning Laravel"]
tags = ["laravel", "php", "tech"]
+++

## Day 5: Dynamic Rendering
### Summary
Before I went to bed yesterday I decided to lean into the Rhizome theme a bit harder and renamed Sites to Shoots. Along that same these instead of creating sites we'll be sprouting shoots. I did not rename anything in the database, middleware, or controller. I just changed the views to reflect the botanical theme. 

For Day 5 my goal was to transform shoots from identical placeholders into dynamic pages by allowing users to define a description and a theme color during the "sprouting" process. 

Today's Definition of Done:
- The sites table includes description (text) and theme_color (string) columns.
- The "Sprout a New Shoot" form on the dashboard includes inputs for these two new fields.
- Submitting the form correctly saves the description and color to the database
- Visiting a shoot's subdomain displays that shoot's description instead of a hardcoded message.

### Work Session
Starting off today I knew I needed to add two new colums to the site table so back to `php artisan` for another migration.

``` bash
$ php artisan make:migration add_content_to_sites_table --table=sites

   INFO  Migration [database/migrations/2026_01_16_034514_add_content_to_sites_table.php] created successfully.
```

Inside the migration I added a nullable description of type text and a string for theme color that defaults to green. 

``` php
public function up(): void
{
    Schema::table('sites', function (Blueprint $table) {
        $table->text('description')->nullable(); 
        $table->string('theme_color')->default('green');
    });
}

public function down(): void
{
    Schema::table('sites', function (Blueprint $table) {
        $table->dropColumn(['description', 'theme_color']);
    });
}
```

Then I updated the Site model to add the new description and theme_color columns to the `$fillable` array to ensure I could save them using the form.

``` php
protected $fillable = [
    'name',
    'slug',
    'custom_domain',
    'description',
    'theme_color',
];
```

After that I was ready to update the form, so back to `resources/views/dashboard.blade.php`.

![New View with Description and Theme Color](/images/learning-laravel/day-05-new-view-w-description-and-theme-color.png)

And the last part of this step was to run the migration so I could move on to the controller.

``` bash
$ php artisan migrate

   INFO  Running migrations.

  2026_01_16_034514_add_content_to_sites_table .......................................................... 10.53ms DONE
```

After the migration succeeded I was ready to update the `store` login in `SiteController` so my new values could be stored in their respective columns.

``` php
public function store(Request $request)
{
    $request->merge([
        'slug' => Str::slug($request->slug),
    ]);

    $validated = $request->validate([
        'name' => 'required|string|max:255',
        'slug' => 'required|string|unique:sites,slug|max:255',
        'description' => 'nullable|string|max:1000',
        'theme_color' => 'required|string|in:blue,green,purple',
    ]);

    $request->user()->sites()->create($validated);

    return redirect()->route('dashboard')->with('success', 'Shoot sprouted successfully!');
}
```

And finally I needed to include the description and the theme_color in the json output in `Tenant\SiteController`.

``` php
public function index()
{
    $site = app('current_site');
    return response()->json([
        'shoot_name' => $site->name,
        'description' => $site->description ?? 'No description yet.',
        'theme_color' => $site->theme_color,
        'owner' => $site->user->name,
    ]);
}
```

And to prove to myself that it was working I created another new site called `Joes Site`.

![Joes Site w/ custom description and theme color](/images/learning-laravel/day-05-joes-site.png)

###  Recap 
I added the `description` and `theme_color` columns to the `sites` table and updated the `Site` model and the view for the form on the dashboard. then I ran the migration to apply the database changes and updated the `SiteController` to allow the new form inputs to be saved to the correct columns in the `sites` table and the `Tenant\SiteController` to ensure the page is dynamically rendering based on content in the database. 

The commit for todays session is [commit a7a1488](https://github.com/fjoenichols/RhizomeCMS/commit/a7a14889bf2baa8d88cebeaeb28d030814c1fed3).
