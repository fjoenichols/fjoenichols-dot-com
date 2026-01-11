+++
title = "Day 1: Scoping a Multi-Tenant CMS"
date = 2026-01-10T15:30:00-07:00
categories = ["21-Day Sprints"]
sprints = ["Learning Laravel"]
tags = ["laravel", "php", "tech"]
+++

## Phase 1: Prep Day (Day 1)

### 1. Pick your poison
**Learning Laravel.** I haven't actively used php on purpose for anything since Late php 4/Early php 5. I want to jump back in and refresh my php skills so I plan to do so by learning the Laravel framework.

### 2. Define the essence
To build a functional multi-tenant system, I need to focus on these sub-skills:

* **Authentication & Profile**: Implementing secure login (Laravel Breeze) so users can manage their own accounts.
* **CRUD**: Allowing users to Create, Read, Update, and Delete content.
* **Data Ownership**: Creating data relationships to ensure content is isolated to the correct customer.
* **Multi-Site Routing**: Learning how Laravel handles subdomains and maps custom external domains to specific customer data.

### 3. Name the prize
**The Goal**: A working Multi-Tenant CMS. 
A user should be able to sign up, get a dedicated subdomain or path, pick a theme, and manage their own unique set of posts/pages without seeing anyone else's data.

### 4. Clear the deck
* **Environment**: 
    * Install php, composer, laravel
    * Set up github repo
    * Create the laravel project
* **Naming**: The CMS will be called RhizomeCMS

### 5. Burn the ships
> I will spend 60 minutes a day for the next 20 days building a **Multi-tenant CMS** so I can learn **Laravel**.