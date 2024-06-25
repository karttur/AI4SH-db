---
layout: post
title: "Schema: enzymes"
categories: ai4sh-db
excerpt: "AI4SH schema enzyme activity data"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-03-21 11:27'
modified: '2024-06-26'
comments: true
share: true

---

## Outline

The schema _enzymes_ contains tables for recorded enzymatic activity using the [Soil Enzyme Activity Reader (SEAR)](https://www.digit-soil.com/webinar-registration) developed by [Digit-Soil AG](https://www.digit-soil.com). Each read records the activity of 5 different enzymes, with 5 replicas of each enzyme???

The schema _enzymes_ is built along the same principles as the other schemas for laboratory derived soil properties, [wetlab](../ai4sh-db_wetlab/) and [edna](../ai4sh-db_edna/). The sampleid and userid are derived from other schemas and registered together with other metadata for each SEAR analysis in the table _eeameta_. Each SEAR analysis is given a Universal Unique ID (UUID), that is then used for linking to the results (table: _eea_) and quality (table: _eeaquality_.)

The actual SEAR results are thus stored in the table _eea_, with each individual enzyme in a separate record. The enzyme is registered using a 3 letter code, expanded in the support table _enzymecode_.

### DBML

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'AI4SH schema for enzyme data'
}

Table users.user {
  userid UUID
}

Table samples.sample_event {
  sampleuuid UUID
}

Table eeameta {
  sampleuuid UUID [pk]
  topsoil Boolean [pk]
  storageconditions TEXT
  analysisdate date
  useruuid UUID
  eeaanalysisuuid UUID  
}

Table enzymecode {
  enzymecode char(3) [pk]
  enzymefull TEXT
  info TEXT
}

Table eea {
  eeaanalysisuuid UUID [pk]
  enzymecode char(3) [pk]
  measurement_pmol_min real
  is_valid Boolean
  not_valid_reason TEXT
}

Table eeaquality {
  sampleuuid UUID [pk]
  enzymecode char(3) [pk]
  rejected_technical_replicates_fraction REAL
  standard_deviation REAL
  raw_r2 REAL
  adjusted_r2 REAL  
}

Ref: "samples"."sample_event"."sampleuuid" - "public"."eeameta"."sampleuuid"

Ref: "users"."user"."userid" - "public"."eeameta"."useruuid"

Ref: "public"."eeameta"."eeaanalysisuuid" - "public"."eea"."eeaanalysisuuid"

Ref: "public"."enzymecode"."enzymecode" - "public"."eea"."enzymecode"

Ref: "public"."eeaquality"."enzymecode" - "public"."enzymecode"."enzymecode"

Ref: "public"."eeaquality"."sampleuuid" - "public"."eeameta"."sampleuuid"
```

### Figure

<figure>
<a href="../../images/DBML_schema-enzyme.png">
<img src="../../images/DBML_schema-enzyme.png"></a>
<figcaption>Enzyme DBML database structure</figcaption>
</figure>
