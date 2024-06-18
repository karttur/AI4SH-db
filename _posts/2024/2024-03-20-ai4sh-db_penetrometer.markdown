---
layout: post
title: "Schema: penetrometer"
categories: ai4sh-db
excerpt: "AI4SH schema penetrometer data"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-03-20 11:27'
modified: '2024-06-03'
comments: true
share: true

---

## Outline

Penetrometers are field probes that are pushed into the soil. Modern penetrometers record different soil properties using microelectronic controllers and can indirectly observe physical, chemical and biological properties. There are even penetrometers with built-in spectral sensors that can record the soil profile spectra as the penetrometer is pushed into the soil. In AI4SH only a simple type of steel pinned penetrometers are used (figure 1).

<figure>
<img src="../../images/penetrometers.png">
<figcaption>Field penetrometers for soil properties. The frontmost probe can sense all the parameters of the other three combined (blue body: Nitrogen, Phosphorus and Potassium [NPK]; silver-grey body: pH; orange body: temperature, soil moisture, electric conductivity, salinity and epsilon).</figcaption>
</figure>

The soil properties that can be observed with the AI4SH adopted soil penetrometers include:

- soil moisture,
- temperature,
- electric conductivity,
- salinity,
- epsilon,
- nitrogen (N),
- phosphorus (P),
- potassium  (K), and
- pH.

A main reason for applying soil penetrometers as part of AI4SH is that soil moisture is a required parameters for interpreting and modelling soil spectra acquired from wet (non-dried) samples. As salinisation is a severe soil health problem, also the direct field observations of electric conductivity, salinity and epsilon are of interest. The probing of NPK and pH are more indicative, but as these chemical soil properties can be captured simultaneously with those of primary interest, they are also included in the database schema _penetrometer_.

As with spectrometers, each individual penetrometer has a slightly different response and thus needs an individual calibration. The table _penetrometer_ is for registering each individual instrument, the table _penetrometertypes_ for registering the physical quantities and units observed by each type of (brand+model) of penetrometer, and the table _penetrometercalib_ for storing the (optional) calibration settings.

The meta data for each penetrometer observation is stored in the table _probemeta_. The table is prepared for accepting multiple observations for the each single sample (field: _proberepeat_). Observations can be done either in the field or using mixed (wet) samples, this is recorded in the field _subsample_  with a single letter (M, C, N, E, S, W for Mixed, Central, North, East, South and West). The alterantive options for peneteromter observations leads to a metadata table with a large number of primary keys (pk):

- penetrometeruuid (the probe)
- sampleuuid (the sample event)
- topsoil (topsoil or subsoil)
- subsample (Mixed, Central, North, East, South or West)
- scanrepeat (sequence from 1 to maximum 9)

The actual observation data is saved in the table _penetrometerobs_. The link between _probemeta_ and _penetrometerobs_ is a UUID set for each individual observation (obsuuid). The link to the penetrometer is also via a UUID (penetrometeruuid). As for other schemas, the user responsible is linked to each observation via the useruuid.  

### DBML

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'AI4SH schema for penetrometer data'
}

Table users.user {
  userid UUID
}

Table samples.sample_event {
  sampleuuid UUID
}

Table penetrometer {
  brand TEXT [pk]
  model TEXT [pk]
  serialnumber TEXT [pk]
  penetrometeruuid UUID
}

Table penetrometertypes {
  brand TEXT [pk]
  model TEXT [pk]
  quantity TEXT [pk]
  unit TEXT
}

Table penetrometercalib {
  penetrometeruuid UUID [pk]
  quantity TEXT [pk]
  gain REAL [DEFAULT: 1]
  offset REAL [DEFAULT: 0]
}

TABLE probemeta {
  penetrometeruuid UUID [pk]
  sampleuuid UUID [pk]
  topsoil Boolean [pk]
  subsample char(1) [pk]
  proberepeat char(1) [pk]
  useruuid UUID
  obsuuid UUID
}
// topsoil (true) represents 0-20 cm, subsoil (false) represents 20-50 cm.
// subsample can be M (mixed), C (central), N (north), E (east), S (south) of W (west).
// proberepeat is a simple numbering starting a 1 to a maximum 0f 9 repeats for the same sample

TABLE penetrometerobs {
  obsuuid UUID [pk]
  quantity TEXT [pk]
  obsmean REAL[]
  obsstd REAL[]  
}

REF: samples.sample_event.sampleuuid - probemeta.sampleuuid

REF: users.user.userid - probemeta.useruuid


REF: "public"."penetrometer"."penetrometeruuid" - "public"."probemeta"."penetrometeruuid"

REF: "public"."penetrometertypes"."brand" - "public"."penetrometer"."brand"

REF: "public"."penetrometertypes"."model" - "public"."penetrometer"."model"

REF: "public"."penetrometer"."penetrometeruuid" - "public"."penetrometercalib"."penetrometeruuid"

Ref: "public"."probemeta"."obsuuid" - "public"."penetrometerobs"."obsuuid"
```

### Figure

<figure>
<a href="../../images/DBML_schema-penetrometer.png">
<img src="../../images/DBML_schema-penetrometer.png"></a>
<figcaption>Penetrometer DBML database structure</figcaption>
</figure>
