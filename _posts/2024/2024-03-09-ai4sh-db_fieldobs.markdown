---
layout: post
title: "Schema: fieldobs"
categories: ai4sh-db
excerpt: "AI4SH schema fieldobs"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-03-09 11:27'
modified: '2024-07-08'
comments: true
share: true

---

## Outline

The schema **fieldobs** includes tables for direct observations in field, including photographs, soil bulk density and soil moisture. This data is most commonly captured by the user that collects the samples. Bulk density and soil moisture require taking a soil cylinder filled with soil to a "kitchen" laboratory for weighing and drying. This is also usually done in direct association with the sampling - if for no other reason then a limited number of soil cylinders.

The _fieldobs_ schema is linked to the schema for **users** (for registering the person[s] performing the sampling) and to the schema **sample_event**.

In the schema, the sampler should register the photos taken from the central point in approximately each of the main directions (N, E, S, W). Also this should be confirmed in the database, registering the name of the corresponding photo and its azimuth (that can deviate and also allows additional photos to be registered).

## Idea and objective

The **fieldobs** schema covers general information captured directly, or almost directly, while sampling. The tools required for capturing the **fieldobs** data are the classical soil tools: spade, hammer and soil cylinder. Complemented with a camera (smartphone) for photos.

### DBML

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs
// Tool: https://dbdiagram.io/d

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'AI4SH schema for fieldobs'
}


Table sampleevent.sampling {
  sampleid INTEGER
}

Table fieldobs {
 sampleid INTEGER [pk]
 soil_class_code char[5]
 description TEXT
}

Table soil_class {
 soil_class_code char[5] [pk]
 soil_class TEXT
 description TEXT
}

Table sample_photo {
 sampleid INTEGER [pk]
 azimuth SMALLINT [pk]
 photoid TEXT
 photourl TEXT
}

Table soilmoisture {
 sampleid INTEGER [pk]
 topsoil BOOLEAN [pk]
 repetition SMALLINT [pk, DEFAULT:1]
 soilmoisture REAL
 smunit TEXT [DEFAULT: vol vol-1]
 method TEXT [DEFAULT: cylinder100]
}

Table bulkdensity {
 sampleid INTEGER [pk]
 topsoil BOOLEAN [pk]
 repetition SMALLINT [pk, DEFAULT:1]
 density REAL
 method TEXT [DEFAULT: cylinder100]
}


Table soil_moisture_nominal {
  soil_moisture_nominal varchar(8)
}
// soil_moisture_nominal: "too wet", "too dry", "optimal"


Ref: "public"."fieldobs"."sampleid" - "public"."sampling"."sampleid"

Ref: "public"."fieldobs"."soil_class_code" - "public"."soil_class"."soil_class_code"

Ref: "public"."sampling"."soil_excavation_tool" - "public"."soil_excavation_tool"."soil_excavation_tool"

Ref: "public"."fieldobs"."sampleid" - "public"."sample_photo"."sampleid"

Ref: "public"."sampling"."soil_moisture_nominal" - "public"."soil_moisture_nominal"."soil_moisture_nominal"

Ref: "public"."fieldobs"."pointid" - "sites"."samplepoint"."pointid"

Ref: "public"."sampling"."macrofauna_excavation_tool" - "public"."macrofauna_excavation_tool"."macrofauna_excavation_tool"

Ref: "sites"."insitu_methods"."pontid" - "public"."fieldobs"."pointid"
```

### Figure

<figure>
<a href="../../images/DBML_schema-sampling.png">
<img src="../../images/DBML_schema-sampling.png"></a>
<figcaption>Sampling DBML database structure</figcaption>
</figure>
