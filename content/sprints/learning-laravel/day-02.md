+++
title = "Day 2: Models & Migrations"
date = 2026-01-12T23:30:00-07:00
categories = ["21-Day Sprints"]
sprints = ["Learning Laravel"]
tags = ["laravel", "php", "tech"]
+++

## Day 2: Models & Migrations
### Summary
Going into today's session I had the following goals:
- Create a Site model
- Create a sites table that can be "owned" by a user
- Create a site belonging to a user

I referenced the Laravel migrations doc at [laravel.com/docs/12.x/migrations](https://laravel.com/docs/12.x/migrations) and asked Gemini a few clarifying questions when I ran into issues that I couldn't quickly find an answer to. 

I was also aquainted with `php artisan` today, and it feels very much like working with Ruby on Rails scaffolding. 

### The Work Session
I created the model and a database migation for Site using php artisan.

``` bash
$ php artisan make:model Site -m

   INFO  Model [app/Models/Site.php] created successfully.

   INFO  Migration [database/migrations/2026_01_13_054336_create_sites_table.php] created successfully.
```

I really appreciated that it told me the path to the files it created making finding them really easy. 

When editing the migration file I learned about `nullable()` to make a database field nullable because I want users to have the option to use a custom domain but not force them to if having a subdomain is fine and `unique()` to make a field unique so site slugs are always unique. 

I ran into an issue where I didn't define the `user_id` column before attempting to make it a foreign key so I got an error trying to run the migration.

I was able to get past this by defining the column first and then creating a foreign key.

``` php
Schema::create('sites', function (Blueprint $table) {
    $table->id();
    $table->timestamps();
    $table->unsignedBigInteger('user_id');
    $table->foreign('user_id')->references('id')->on('users');
    $table->string('name');
    $table->string('slug');
    $table->string('custom_domain')->nullable();
});
```

Running the migration I attempted to use `php artisan tinker` to create a new site belonging to my user, but I was met with an error! This is where I should note that my foreign key was originally called `owner_id`, not `user_id`.

```
Illuminate\Database\QueryException  SQLSTATE[HY000]: General error: 1 table sites has no column named user_id (Connection: sqlite, Database: database/database.sqlite, SQL: insert into "sites" ("name", "slug", "user_id", "updated_at", "created_at") values (Mountain Lotus Digital, mountainlotus, 1, 2026-01-13 06:11:18, 2026-01-13 06:11:18)).
```

Through a bit of research I learned that Laravel by default will expect the foreign key column to be named <referenced-table>_id and since I was referencing the users table `user_id` was expected.  I saw there were ways to work around this by defining constraints and some other stuff but I figured just using `user_id` was the easiest (and probably correct) thing to do.

So I updated my migration file and ran `php artisan migrate:fresh` to start with a fresh database. 

I used artisan to check the Site model and make sure that `user_id` was present. It was!
```
$ php artisan model:show Site

  App\Models\Site ....................................................................................................
  Database .................................................................................................... sqlite
  Table ........................................................................................................ sites

  Attributes ............................................................................................. type / cast
  id increments, unique ................................................................................ integer / int
  created_at nullable ............................................................................ datetime / datetime
  updated_at nullable ............................................................................ datetime / datetime
  user_id .................................................................................................... integer
  name fillable .............................................................................................. varchar
  slug fillable .............................................................................................. varchar
  custom_domain nullable, fillable ........................................................................... varchar
```

Now I headed back into tinker to see if I could finally create a site that belonged to a user. Since I had done `migrate:fresh` I had to recreate my user and then create the site. 

```
$ php artisan tinker
Psy Shell v0.12.18 (PHP 8.4.1 — cli) by Justin Hileman
New PHP manual is available (latest: 3.0.1). Update with `doc --update-manual`
> $user = App\Models\User::create([
   'email' => '[redacted]',
    'password' => bcrypt('password'),
]);
= App\Models\User {#6146
    name: "Joe Nichols",
    email: "[redacted]",
    #password: "[redacted]",
    updated_at: "2026-01-13 06:17:10",
    created_at: "2026-01-13 06:17:10",
    id: 1,
  }

> $user->sites()->create([
me' => 'Mountain Lotus Digital',
    'slug' => 'mountainlotus',
]);
= App\Models\Site {#6892
    name: "Mountain Lotus Digital",
    slug: "mountainlotus",
    user_id: 1,
    updated_at: "2026-01-13 06:17:16",
    created_at: "2026-01-13 06:17:16",
    id: 1,
  }
```

Success! 

### Recap
To recap I learned to create the Site model, create a sites table, run migrations and use tinker to create a site that belongs to a user all in a Laravel way. 

Todays code commit is [commit b26d6a1](https://github.com/fjoenichols/RhizomeCMS/commit/b26d6a1a2fd801d60d329409375e364b92eb5c79).