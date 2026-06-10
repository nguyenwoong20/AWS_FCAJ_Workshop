---
title: "Blog 2: DynamoDB Data Modeling for a Movie App"
date: 2026-06-10
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

{{% notice note %}}
✏️ Draft — post to the [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj) and paste the post link here.
{{% /notice %}}

**Posted at:** [LINK TO FACEBOOK POST]

# "Where is my WHERE clause?" — DynamoDB Data Modeling for a Movie App

Coming from MongoDB, my first DynamoDB question was: *how do I query movies by category if the key is the movie id?* Here's what I learned designing the tables for my Cinemax app.

**Rule 1: design for your access patterns, not your data.** I listed what the app actually does: get a movie by id, browse newest movies in a category, search by title, list a user's bookmarks. Each pattern decides a key or an index.

**Rule 2: the GSI is your "second WHERE clause".** The main table uses `id` as partition key (get-by-id is O(1)). For "browse Action movies, newest first" I added a Global Secondary Index: partition key `category`, sort key `createdAt`. One `Query` call, already sorted, no scan.

**Rule 3: composite keys model relationships.** Bookmarks don't need their own id. The table is just partition key `userId` + sort key `movieId`. "All bookmarks of user X" is one Query; "did X bookmark movie Y" is one GetItem; adding the same bookmark twice simply overwrites itself. Three behaviors for free from key design.

**Rule 4: accept that Scan is the exception.** Title search uses a Scan with a filter — fine at my catalog's size, wrong at a million items. The honest fix at scale would be OpenSearch or a dedicated search index, and knowing *where your design stops scaling* is part of the design.

DynamoDB punishes you for skipping the modeling step and rewards you absurdly when you do it: single-digit-millisecond queries on the free tier, with zero servers.

#AWS #DynamoDB #NoSQL #DataModeling #FirstCloudJourney
