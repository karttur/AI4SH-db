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
date: '2024-03-10 11:27'
modified: '2024-06-03'
comments: true
share: true

---

## Outline

The schema _samples_ is meant to cover the actual soil sampling and the samples, but not the in-situ analysis results. The data to register in _samples_ include sampling method, number and depth of sub-samples taken at each sample point and macro-characteristics of the directly (with human senses) observed soil profiles. The landscape (above ground) characteristics, the cultivation methods and its history are registered in the _sites_ schema (but could be moved or split out in a separate schema). The analysis results from the in-situ method measurements are suggested to be in separate schemas for each method.

The _samples_ schema is linked to the schema for _users_ (for registering the person[s] performing the sampling) and to the schema _sites_. From the _sites_ schema each sampling event will relate to information on:

- in-situ methods to apply (table _pilots.pilot_insitu_methods_),and
- point id (table _pilots.samplepoint_) that links to both the site and the pilot.

Starting each sampling event the responsible person must enter the basic metadata on "whodunit" in the table _sample_event_ - this should generate a universally unique id (UUID) for this sampling event, that is then registered with each of the in-situ methods, and the associated DB schema, as the sampling and analysis progresses.

Each sample point consists of 5 (five) excavation pits - one central and one in each main geographic direction:

- North (N),
- East (E),
- South (S), and
- West (W).

In the schema, the sampler should register which of these 5 pits (sub-points) where actually used for sampling of both the topsoil (0-20 cm) and the subsoil (20-50 cm). There should also be photos taken from the central point in each of the main directions. Also this should be confirmed in the database, preferably including a register of the name of the corresponding photo.

## Idea and objective

The objective of the the _samples_ schema is to hold the basic information on each sample event per sample point. It is in a way the intermediate link between the schema _sites_ holding information related to the sample points (sites and pilots) and the in-situ analysis results derived from the different methods - where each method results are registered in a separate schema.

As with other schemas and tables in this proposed database, each sample event is registered with i) the user (UUID) who is performing this particular sampling, and ii) the sample point (UUID) where it is happening. When registering the sampling event, it is given its own Universally Unique id (UUID). The sampling event UUID is then used as the link to the analysis results.

Via the sample point UUID, linking to the site UUID and the pilot UUID, the type of in-situ observations to be done should be known - this should be registered as a boolean list allowing the sampler to tick off the observations and sampling required.

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
  userid UUID
}

Table pilots.pilot_insitu_methods {
  pilotid UUID [pk]
  wet_chemistry BOOLEAN
  eDNA BOOLEAN
  macrofauna_full BOOLEAN
  macrofauna_simple BOOLEAN
  microbiometer BOOLEAN
  moulder BOOLEAN
  soil_spectra_NO BOOLEAN
  soil_spectra_MX BOOLEAN
  soil_spectra_DS BOOLEAN
  infiltration_rate BOOLEAN
  penetrometer_moisture BOOLEAN
  penetrometer_moisture_salinity BOOLEAN
  penetrometer_moisture_salinity_NPK BOOLEAN
  penetrometer_complete BOOLEAN
  ise_pH BOOLEAN
}

Table pilots.site {
 siteid UUID
 pilotid UUID [pk]
}

Table pilots.samplepoint {
  pointid UUID
  siteid UUID [pk]
}

Table sample_event {
 pointid UUID [pk]
 sampledatetime timestamp [pk]
 sampleuuid UUID
 sampleruuid UUID
 soil_class_code char[5]
 description TEXT
}

Table soil_class {
 soil_class_code char[5] [pk]
 soil_class TEXT
 description TEXT
}

Table sample_photo {
 sampleuuid UUID [pk]
 profile_photo TEXT
 towards_N_photo TEXT
 towards_E_photo TEXT
 towards_S_photo TEXT
 towards_W_photo TEXT
}

Table sampling {
  sampleuuid UUID [pk]
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
}

Table soil_moisture_nominal {
  soil_moisture_nominal varchar(8)
}
// soil_moisture_nominal: "too wet", "too dry", "optimal"

Table soil_excavation_tool {
  soil_excavation_tool varchar(8)
}
// soil_moisture_nominal: "spade", "auger+type1", "auger+type2"

// The table insitu_methods is inhereted from the pilot site schema
// It should be filled automatically when the sample_event is creates.
// But then allow the sampler (user) to add or remove methods applied
Table insitu_methods {
  sampleuuid UUID [pk]
  wet_chemistry BOOLEAN
  eDNA BOOLEAN
  macrofauna_full BOOLEAN
  macrofauna_simple BOOLEAN
  microbiometer BOOLEAN
  moulder BOOLEAN
  soil_spectra_NO BOOLEAN
  soil_spectra_MX BOOLEAN
  soil_spectra_DS BOOLEAN
  infiltration_rate BOOLEAN
  penetrometer_moisture BOOLEAN
  penetrometer_moisture_salinity BOOLEAN
  penetrometer_moisture_salinity_NPK BOOLEAN
  penetrometer_complete BOOLEAN
  ise_pH BOOLEAN
}

Ref: "users"."user"."userid" - "public"."sample_event"."sampleruuid"

Ref: "public"."sample_event"."sampleuuid" - "public"."sampling"."sampleuuid"

Ref: "public"."sample_event"."soil_class_code" - "public"."soil_class"."soil_class_code"

Ref: "public"."sampling"."soil_excavation_tool" - "public"."soil_excavation_tool"."soil_excavation_tool"

Ref: "public"."sample_event"."sampleuuid" - "public"."sample_photo"."sampleuuid"

Ref: "public"."sampling"."soil_moisture_nominal" - "public"."soil_moisture_nominal"."soil_moisture_nominal"

Ref: "public"."sample_event"."sampleuuid" - "public"."insitu_methods"."sampleuuid"

Ref: "public"."sample_event"."pointid" - "pilots"."samplepoint"."pointid"

Ref: "pilots"."samplepoint"."siteid" - "pilots"."site"."siteid"

Ref: "pilots"."site"."pilotid" - "pilots"."pilot_insitu_methods"."pilotid"

Ref: "pilots"."pilot_insitu_methods"."wet_chemistry" - "public"."insitu_methods"."wet_chemistry"

Ref: "pilots"."pilot_insitu_methods"."eDNA" - "public"."insitu_methods"."eDNA"

Ref: "pilots"."pilot_insitu_methods"."macrofauna_full" - "public"."insitu_methods"."macrofauna_full"

Ref: "public"."insitu_methods"."macrofauna_simple" - "pilots"."pilot_insitu_methods"."macrofauna_simple"

Ref: "public"."insitu_methods"."microbiometer" - "pilots"."pilot_insitu_methods"."microbiometer"

Ref: "public"."insitu_methods"."moulder" - "pilots"."pilot_insitu_methods"."moulder"

Ref: "public"."insitu_methods"."soil_spectra_NO" - "pilots"."pilot_insitu_methods"."soil_spectra_NO"

Ref: "public"."insitu_methods"."soil_spectra_MX" - "pilots"."pilot_insitu_methods"."soil_spectra_MX"

Ref: "public"."insitu_methods"."soil_spectra_DS" - "pilots"."pilot_insitu_methods"."soil_spectra_DS"

Ref: "public"."insitu_methods"."infiltration_rate" - "pilots"."pilot_insitu_methods"."infiltration_rate"

Ref: "public"."insitu_methods"."penetrometer_moisture" - "pilots"."pilot_insitu_methods"."penetrometer_moisture"

Ref: "public"."insitu_methods"."penetrometer_moisture_salinity" - "pilots"."pilot_insitu_methods"."penetrometer_moisture_salinity"

Ref: "public"."insitu_methods"."penetrometer_moisture_salinity_NPK" - "pilots"."pilot_insitu_methods"."penetrometer_moisture_salinity_NPK"

Ref: "public"."insitu_methods"."penetrometer_complete" - "pilots"."pilot_insitu_methods"."penetrometer_complete"

Ref: "public"."insitu_methods"."ise_pH" - "pilots"."pilot_insitu_methods"."ise_pH"
```

### Figure

<figure>
<a href="../../images/DBML_schema-samplings.png">
<img src="../../images/DBML_schema-samplings.png"></a>
<figcaption>Samplings DBML database structure</figcaption>
</figure>
