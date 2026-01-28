+++
title = "Day 13: The Flu & A Pivot"
date = 2026-01-27T17:40:00-07:00
categories = ["21-Day Sprints"]
sprints = ["Learning Laravel"]
tags = ["laravel", "php", "tech"]
+++

### Summary
I had hoped to be a lot furhter along by now, but I ended up getting the flu on Saturday, slept most of the day Sunday and Monday, and I'm just now feeling like getting out of bed and doing stuff again so here's day 13 three days late. 

I made a pivot from sqlite to postgres so I can verify things work when I get ready to publish to cloud. 

### Work Session
It was a pretty easy day. I installe postgres on my linux box
``` bash
sudo apt update
sudo apt install postgresql postgresql-contrib php8.4-pgsql -y

sudo systemctl start postgresql
sudo systemctl enable postgresql
```

and created my database
``` bash
sudo -u postgres psql

CREATE DATABASE rhizome_cms;
CREATE USER [redacted] WITH PASSWORD [redacted];
GRANT ALL PRIVILEGES ON DATABASE rhizome_cms TO [redacted];
ALTER DATABASE rhizome_cms OWNER TO [redacted];
\q
```

Then I updated my .env file and ran `php artisan migrate`. 

The last thing I did was create a booted method in the Site model to automatically create a Home page when a site is created.
``` php
protected static function booted()
{
    static::created(function ($site) {
        $site->pages()->create([
            'title' => 'Home',
            'slug' => 'home',
            'content' => 'Welcome to your new site! This page was automatically generated to help you get started. You can edit this content anytime from your dashboard.',
        ]);
    });
}
```

###  Recap 
The commit for todays session is [commit 4ec019b](https://github.com/fjoenichols/RhizomeCMS/commit/4ec019b3d4cf64c2d3a122a2afc8a14f650c4b1d).
