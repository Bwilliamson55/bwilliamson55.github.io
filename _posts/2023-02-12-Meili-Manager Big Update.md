---
title: Meili-Manager Update!
date: 2023-02-12 20:30
categories: [General, Videos]
author: bwilliamson
tags: [meilisearch, vue, js, quasar]
---

This is Meili-Manager, a Quasar app built to help manage your Meilisearch instance(s). It's been a little side project of mine for a few weekends.

I've made some posts about it already, but this post is around the time that **most** of the features I think necessary are now active. Really cool stuff like a full blown JSON editor for documents, and API key management.

# How to run the thing

You can run this locally or hosted. Currently the demo is at [https://meili-manager.vercel.app/#/](https://meili-manager.vercel.app/#/)

The repository can be found here: [https://github.com/Bwilliamson55/meili-manager](https://github.com/Bwilliamson55/meili-manager) with better documentation of the details.

# Features

Current feature set as of this posting:

_No more manual API calls to change settings_

- Indexes
  - List, Create, Edit, Delete
  - Statistics and status
- Settings
  - Per index, full settings object available to edit
  - Intuitive web form rather than raw JSON
- Search
  - Interactive Vue instantsearch widgets in each index view
    - Stats
    - Search Query
    - Sort Options
    - Filters
    - Refinements
    - Hits
- Keys
  - Create, edit, update, and delete API keys
- Tasks
  - View and search through the latest 1000 tasks in real time

# Walkthrough of app in current state

<div style="position: relative; padding-bottom: 5%; height: 30vh;"><iframe src="https://www.loom.com/embed/3e36443768e3453996f99ab596234174" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe></div>

As always, I hope you found this interesting :-)
