+++
title = "Day 14: Livewire Autosave"
date = 2026-01-28T19:40:00-07:00
categories = ["21-Day Sprints"]
sprints = ["Learning Laravel"]
tags = ["laravel", "php", "tech"]
+++

### Summary
Well I wanted to make things a bit more reactive on the webapp side so I decided to implement some livewire components to do page autosave when the fields are being edited. What I thought was going to be pretty straight forward turned into a bit of a fiasco for a very silly reason.

### Work Session
Setting up livewire to use was fairly straight forward just using composer to do the livewire install. 

Then I created a livewire component for PageEditor. This is where all the problems began, but I didn't know it yet. 

I added views, the livewire component at the top of the route page, and then when I clicked on the "Edit" button instead of my view being rendered I saw raw php. Weird.  I spent about 30 minutes fumbling around getting various 500 errors until I saw it. I forgot the `<? php` header on the PageEditor components. Hah! 

Anyways after a bunch of frustration over the stupidest thing the livewire component to update pages works and there is no more save button. It just autosaves.

###  Recap 
The commit for todays session is [commit 0dde6d9](https://github.com/fjoenichols/RhizomeCMS/commit/0dde6d97afec9dd49a1867add38d2ac3b7bbb1b6).
