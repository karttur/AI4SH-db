---
layout: post
title: "Schema: wetlab"
categories: ai4sh-db
excerpt: "AI4SH schema wetlab laboratory data"
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

## Outline

The schema _wetlab_ ... The schema is modelled after the [Open Soil Spectral Library (OSSL)]() data for laboratoy ... OSSL include the European LUCAS data and the default defined standard methods included in the schema _wetlab_ is a repliation of this data.

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
}

Table analysismethod{

}

```

### Figure
