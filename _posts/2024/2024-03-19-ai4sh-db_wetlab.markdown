---
layout: post
title: "Schema: wet-chemistry"
categories: ai4sh-db
excerpt: "AI4SH schema wet-chemisty laboratory data"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-03-18 11:27'
modified: '2024-06-03'
comments: true
share: true

---
# UNDER CONSTRUCTION


## Outline


### DBML

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'AI4SH schema for wet chemistry laboratory data'
}

Table users.user {
  userid UUID
}

Table samples.sample_event {
  sampleuuid UUID
  siteid UUID [pk]
}


```

### Figure
