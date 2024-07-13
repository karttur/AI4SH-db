---
layout: post
title: "Schema: users"
categories: ai4sh-db
excerpt: "AI4SH database schema users, only a sketch to indicate that it is required."
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-03-01 11:27'
modified: '2024-07-13'
comments: true
share: true

---

## Outline

The schema **users** is only schematic, indicating its existence; registered users are linked to tables in all other schemas of the suggested database.

## Idea and objective

The idea is that all the researchers, field workers and laboratory staff etc., involved should register as _users_ and get assigned a unique id. The unique id is assigned using the postgreSQL data type SERIAL - an automatically incremented integer number with each added database record. For each entry in the database the person who did the work and is responsible (which can differ) should then be inserted using the userid as link.

### DBML

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'AI4SH schema for users'
}

Table user {
  userid SERIAL
  firstname TEXT [pk]
  middlename TEXT [pk]
  lastname TEXT [pk]
  organisation TEXT [pk]
}

Table organisation {
 organisation TEXT [pk]
 country_iso2 char(2) [NOT NULL]
}

Ref: user.organisation > organisation.organisation
```

### Figure

<figure>
<a href="../../images/DBML_schema-users.png">
<img src="../../images/DBML_schema-users.png"></a>
<figcaption>Users DBML database structure (sketch)</figcaption>
</figure>
