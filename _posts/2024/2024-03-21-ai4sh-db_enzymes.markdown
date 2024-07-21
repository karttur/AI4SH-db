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
modified: '2024-07-13'
comments: true
share: true

---

## Outline

The schema **enzymes** contains tables for recorded enzymatic activity using the [Soil Enzyme Activity Reader (SEAR)](https://www.digit-soil.com/webinar-registration) developed by [Digit-Soil AG](https://www.digit-soil.com). Each read records the activity of 5 different enzymes, with 5 replicas of each enzyme.

The schema **enzymes** is built along the same principles as the other schemas for laboratory derived soil properties, [wetlab](../ai4sh-db_wetlab/) and [edna](../ai4sh-db_edna/). The sampleid and userid are derived from other schemas and registered together with other metadata for each SEAR analysis in the table _eeameta_. Each SEAR analysis is given a uique (SERIAL) id, that is then used for linking to the results (table: _eea_ [extracellular enzymatic activity]).

The actual SEAR results are thus stored in the table _eea_, with each individual enzyme in a separate record. The enzyme is registered using a 3 letter code, expanded in the support table _enzymecode_.

### DBML

```
/// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'AI4SH schema for enzyme data'
}

Table users.user {
  userid SERIAL
}

Table samples.sample_event {
  sampleid SERIAL
}

Table eeameta {
  eeaanalysisid SERIAL [pk]
  userid INTEGER
  sampleid INTEGER
  analysisdate date
  sampletemperature REAL  
}

Table enzymecode {
  enzymecode char(3) [pk]
  enzymefull TEXT
  info TEXT
}
// enzymecodes
//  GLA: Chitinase/ β-glucosaminidase,
//  GLU: β-Glucosidase,
//  PHO: phosphatases (Phosphomonesterases),
//  XYL: β-Xylosidase,
//  LEU: Leucine- aminopeptidase.

Table eea {
  eeaanalysisid INTEGER [pk]
  enzymecode char(3) [pk]
  measurement_pmol_min REAL
  measurement_std_err REAL
  is_valid BOOLEAN
  not_valid_reason TEXT
}

Ref: "samples"."sample_event"."sampleid" - "public"."eeameta"."sampleid"

Ref: "public"."eeameta"."eeaanalysisid" - "public"."eea"."eeaanalysisid"

Ref: "public"."enzymecode"."enzymecode" - "public"."eea"."enzymecode"

Ref: "users"."user"."userid" - "public"."eeameta"."userid"
```

### Figure

<figure>
<a href="../../images/DBML_schema-enzyme.png">
<img src="../../images/DBML_schema-enzyme.png"></a>
<figcaption>Enzyme DBML database structure</figcaption>
</figure>
