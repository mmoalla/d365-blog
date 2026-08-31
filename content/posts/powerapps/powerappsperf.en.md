---
title: "Power Apps performance: practical techniques to fix slow canvas apps"
date: 2026-08-31
draft: false
categories: ["Power Platform"]
tags: ["Power Apps", "Performance", "Canvas Apps", "Power Fx", "Dataverse"]
summary: "Slow startup, laggy screens, delayed actions — canvas app performance problems tend to come from a handful of recurring patterns. Here's how to diagnose them and the fixes that actually move the needle."
---
 
## Context
 
Canvas apps are easy to build fast, but that same flexibility makes it easy to accumulate performance degradation.
This article walks through the recurring causes of slow canvas apps, roughly in order of impact, along with the concrete fix for each.
 
## Prerequisites
 
- An existing canvas app you want to optimize (or one you're actively building)
- Access to Power Apps Studio with maker permissions.

## Step 1: Optimise loading sumultinously Datasets
Data is loaded by connecting to the data source connector. Sequential loading using for example multiple `ClearCollect` function in the `OnStart` property significantly slows down the application.
To reduce load times, the **Concurrent** function is used; this allows data (such as multiple collections) to be loaded simultaneously rather than sequentially.
{{< img src="images/powerapps/image1.png" >}}
 
## Step 2: Cache table data
Use the **ClearCollect** and **Set** function to locally cache table data and avoid loading it into galleries, forms, dropdown controls and so on, every time a screen is opened.
{{< img src="images/powerapps/image2.png" >}}
{{< img src="images/powerapps/image3.png" >}}

## Step 3: Limiting the size of a collection
Limit the number of columns in a collection and select that you use in the app
{{< img src="images/powerapps/image4.png" >}}

## Step 4: Bulk update
{{< img src="images/powerapps/image5.png" >}}
 
## Step 5: minimize what happens at startup
Everything in `App.OnStart` runs before the user sees anything. The leaner that formula, the faster the perceived startup.
 
- Move constants out of variables set in `OnStart` and into **Named Formulas** instead — they're computed lazily, only when actually referenced, rather than unconditionally on every launch.
 
## Common pitfalls
 
- **Optimizing controls before checking data patterns**: a screen with 15 controls and one bad `ForAll` loop is usually slower than a screen with 60 well-behaved controls — profile first.
- **Not noticing delegation warnings**: an app that "works fine" in testing with 50 rows can silently return wrong results in production with 5,000 rows, purely because a filter wasn't delegable.

## Further reading
- [Tips and best practices to improve performance of canvas apps — Microsoft Learn](https://learn.microsoft.com/power-apps/maker/canvas-apps/performance-tips)
- [Identify and mitigate canvas app performance issues — Microsoft Learn](https://learn.microsoft.com/power-platform/architecture/key-concepts/performance/top-issues)
- [Optimize canvas apps that require complex business logic — Microsoft Learn](https://learn.microsoft.com/power-platform/architecture/reference-architectures/optimize-performance-canvas-apps)