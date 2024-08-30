---
layout: post
title: "Schema: samples"
categories: ai4sh-db
excerpt: "AI4SH schema samples"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-03-11 11:27'
modified: '2024-07-13'
comments: true
share: true

---

## Outline

The schema **samples** is meant to cover the actual soil sampling and the samples, but not the in-situ analysis results. The data to register in **samples** include sampling method, number and depth of sub-samples taken at each sample point and macro-characteristics of the directly (with human senses) observed (below ground) soil profiles. The landscape (above ground) characteristics, the cultivation methods and its history are registered in the _landscape_ and _landstate_ schemas. The analysis results from the in-situ method measurements are in separate schemas for each method.

The _samples_ schema is linked to the schema for **users** (for registering the person[s] performing the sampling) and to the schema **sites**. From the **sites** schema each sampling event will relate to information on:

- in-situ methods to apply (table _sites.point_insitu_methods_), and
- point id (table _sites.samplepoint_) that links to both the site and the pilot.

Starting each sampling event the responsible person must enter the basic metadata on "whodunit" in the table _sample_event_ - this should generate a unique SERIAL (integer) id for this sampling event (_sampleid_), that is then registered with each of the in-situ methods as the sampling and analysis progresses.

Each sample point consists of 5 (five) excavation pits - one central and one in each main geographic direction:

- North (N),
- East (E),
- South (S), and
- West (W).

In the schema, the sampler should register which of these 5 pits (sub-points) where actually used for sampling of both the topsoil (0-20 cm) and the subsoil (20-50 cm). There should also be photos taken from the central point in approximately each of the main directions. Also this should be confirmed in the database, registering the name of the corresponding photo and its azimuth (that can deviaate and also allows additional photos to be registered).

The excavation tool/method (table: _soil_excavataion_tool_) is in general the same for all types of in-situ analysis methods, except for the macrofauna. The macrofauna in-situ observation is either done from a monolith (see schema [macrofauna](../ai4sh-db_macrofauna)) or by pouring dissolved mustard powder on the soil surface and capturing the earthworms that crawl up. Thus the excavation of any macrofauna must be registered separately (support table for tools/methods: _macrofauna_excavataion_tool_).

## Idea and objective

The objective of the the **samples** schema is to hold the basic information on each sample event per sample point. It is in a way the intermediate link between the schema **sites** holding information related to the sample points (sites and pilots) and the in-situ analysis results derived from the different methods - where each method results are registered in a separate schema.

As with other schemas and tables in this proposed database, each sample event is registered with i) the user (userid) who is performing this particular sampling, and ii) the sample point (pointid) where it is happening. When registering the sampling event, it is given its own (SERIAL) sampleid. The sampling event id is then used as the link to the analysis results.

Via the sample point id (pointid), linking to the site id (siteid) and the pilot id (pilotid), the type of in-situ observations to be done should be known before any field sampling.

The schema is built up after the field and sampling protocols developed with AI4SH.

### DBML

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'AI4SH schema for samples'
}

Table users.user {
  userid INTEGER
}

Table sites.samplepoint {
  pointid INTEGER
}

Table sites.insitu_methods {
  pontid INTEGER [pk]
  wet_chemistry BOOLEAN
  eDNA BOOLEAN
  macrofauna BOOLEAN
  microbiometer BOOLEAN
  moulder BOOLEAN
  soil_spectra_NO BOOLEAN
  soil_spectra_MX BOOLEAN
  soil_spectra_DS BOOLEAN
  penetrometer_NPKPHCTH BOOLEAN
  ise_pH BOOLEAN
}

Table sample_event {
 pointid INTEGER [pk]
 sampledatetime timestamp [pk]
 sampleid SERIAL
 userid INTEGER
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
 method TEXT [DEFAULT: cylinder]
}

Table bulkdensity {
 sampleid INTEGER [pk]
 topsoil BOOLEAN [pk]
 repetition SMALLINT [pk, DEFAULT:1]
 density REAL
}

Table sampling {
  sampleid INTEGER [pk]
  soil_moisture_percent SMALLINT
  soil_moisture_nominal varchar(8)
  topsoil_subsample_C BOOLEAN
  topsoil_subsample_N BOOLEAN
  topsoil_subsample_E BOOLEAN
  topsoil_subsample_S BOOLEAN
  topsoil_subsample_W BOOLEAN
  sobsoil_subsample_C BOOLEAN
  subsoil_subsample_N BOOLEAN
  subsoil_subsample_E BOOLEAN
  subsoil_subsample_S BOOLEAN
  subsoil_subsample_W BOOLEAN
  subsample_C_max_depth_cm SMALLINT
  subsample_N_max_depth_cm SMALLINT
  subsample_E_max_depth_cm SMALLINT
  subsample_S_max_depth_cm SMALLINT
  subsample_W_max_depth_cm SMALLINT
  soil_excavation_tool varchar(16)
  macrofauna_excavation_tool varchar(16)
}

Table soil_moisture_nominal {
  soil_moisture_nominal varchar(8)
}
// soil_moisture_nominal: "too wet", "too dry", "optimal"

Table soil_excavation_tool {
  soil_excavation_tool varchar(8)
}
// soil_excavation_tool alterantives: "spade", "auger+type1", "auger+type2"

Table macrofauna_excavation_tool {
  macrofauna_excavation_tool varchar(8)
}
// macrofauna_excavation_tool: "monolith", "spade",  "auger+type1", "auger+type2", "mustard-oil", "mustard-water"
// Only applicable if the point is registered for a macrofauna analysis

Ref: "users"."user"."userid" - "public"."sample_event"."userid"

Ref: "public"."sample_event"."sampleid" - "public"."sampling"."sampleid"

Ref: "public"."sample_event"."soil_class_code" - "public"."soil_class"."soil_class_code"

Ref: "public"."sampling"."soil_excavation_tool" - "public"."soil_excavation_tool"."soil_excavation_tool"

Ref: "public"."sample_event"."sampleid" - "public"."sample_photo"."sampleid"

Ref: "public"."sampling"."soil_moisture_nominal" - "public"."soil_moisture_nominal"."soil_moisture_nominal"

Ref: "public"."sample_event"."pointid" - "sites"."samplepoint"."pointid"

Ref: "public"."sampling"."macrofauna_excavation_tool" - "public"."macrofauna_excavation_tool"."macrofauna_excavation_tool"

Ref: "sites"."insitu_methods"."pontid" - "public"."sample_event"."pointid"
```

### Figure

<figure>
<a href="../../images/DBML_schema-sampling.png">
<img src="../../images/DBML_schema-sampling.png"></a>
<figcaption>Sampling DBML database structure</figcaption>
</figure>
