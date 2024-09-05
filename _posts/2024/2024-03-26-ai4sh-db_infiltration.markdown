---
layout: post
title: "Schema: infiltration"
categories: ai4sh-db
excerpt: "AI4SH schema infiltration rate"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-03-26 11:27'
modified: '2024-08-29'
comments: true
share: true

---

## Outline

The schema **infiltration** contains tables for the in-field single ring (Beerkan method, Lassabatère et al., 2006) water infiltration rate test, commonly abbreviated as BEST – Beerkan Estimation of Soil Transfer parameters.

BEST is a physical, yet simple method; the infiltration test is performed using water vessels filled to a default volume, sequentially poured into a single ring with water depth restricted to a maximum of 1 cm. The infiltration rate is measured by the time it takes for each water volume in the sequence to be swallowed - until a stable time interval is reached. The method also requires information of soil particle size distribution, initial and final soil water content and dry soil bulk density.

From the BEST test the water retention curve, including field capacity and wilting point can be determined, and the hydraulic conductivity estimated. The retention curve (the relation between water content and water potential) differs between wetting and a drying cycles (an effect called hysteresis). The BEST method only gives the wetting cycle water retention curve.

The schema **infiltration** contains tables for filling in test data directly in the field, with the alternative to automatically calculate the soil hydraulic properties from the captured data. It is also possible to note the infiltration rates in another media and then use a third party solution to calculate the soil hydraulic properties and fill in only the results.

The schema **infiltration** is built along the same principles as the other schemas for derived soil properties. The sampleid and userid are derived from other schemas and registered together with other metadata for each infiltration in the table _infiltrationmeta_. Each infiltration test is given a unique (SERIAL) id, that is then used for linking to the (optional) test data (table: _infiltrationrate_) and the results (tables: _soilwaterretention_ and _hydraulicconductivity_).

The texture data is assumed to be the same before and after the infiltration test and resides in the schema [_wetlab_](../ai4sh-db_wetlab/). The sample point soil moisture content (default given as vol/vol) resides in the schema _samples_. For the infiltration rate test to be complete, also the soil moisture content after the test is just finished need to be added (table: _soilmoisture_ in this schema). The user can choose to also add the initial soil water content in this schema (table: _soilmoisture_). If not inserted, the initial soil water content is taken from the schema _samples_. The default method for defining soil moisture is to use a volume precise soil cylinder and weigh it both when wet and after drying. A simpler and quicker, but presumably less accurate, method is to use a soil [penetrometer](..ai4sh-db_penetrometer/). The method used should be registered in the _soilmoisture_ table. A predefined catalog lists all possible methods and only listed methods are accepted.

### DBML

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs
// Tool: https://dbdiagram.io/d

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'AI4SH schema for infiltration test'
}

Table users.user {
  userid SERIAL
}

Table samples.sample_event {
  sampleid SERIAL
}

TABLE infiltrationmeta {
  sampleid INTEGER [pk]
  userid INTEGER
  topsoil Boolean [pk, DEFAULT: TRUE]
  repetition SMALLINT [pk, DEFAULT:1]
  ringdiameter REAL
  waterunit REAL
  infiltrationid SERIAL
}
// ringdiameter in cm
// waterunit of pouring vessel in ml
// topsoil (true) represents 0-20 cm, subsoil (false) represents 20-50 cm.

Table soilmoisture {
  infiltrationid INTEGER [pk]
  postwetting BOOLEAN [pk, NOT NULL]
  soilmoisture REAL [NOT NULL]
  smunit TEXT [DEFAULT: "vol vol-1"]
  method TEXT [DEFAULT: "cylinder"]
}
// postwetting:
// TRUE for post wetting conditions
// FALSE for initial soil moisture conditions

TABLE smmethod {
  method TEXT [pk]
  description TEXT
  reference TEXT
  url TEXT
}
// e.g: cylinder100, cylinder250, penetrometer

TABLE infiltrationrate {
  infiltrationid INTEGER [pk]
  infiltrationsequence SMALLINT [NOT NULL]
  infiltrationtime REAL [NOT NULL]
}
// infiltrationtime in seconds

Table hydraulicconductivity {
  infiltrationid INTEGER [pk]
  saturatedconductivity REAL
}

Table soilwaterretention {
  infiltrationid INTEGER [pk]
  hysteresiswetting BOOLEAN [DEFAULT: TRUE]
  fieldcapacity REAL
  wilitingpoint REAL
}
// The BEST methdod gives the wetting cycle soil water retention
// If another method is used for drying cycle soil water retention
// hysteresiswetting should be set to FALSE

Table soilwaterretentioncurve {
 infiltrationid INTEGER [pk]
  hysteresiswetting BOOLEAN [DEFAULT: TRUE]
  watercontent REAL
  waterpotential REAL
}
// The BEST methdod gives the wetting cycle soil water retention
// If another method is used for drying cycle soil water retention
// hysteresiswetting should be set to FALSE

Ref: "samples"."sample_event"."sampleid" - "public"."infiltrationmeta"."sampleid"

Ref: "users"."user"."userid" - "public"."infiltrationmeta"."userid"

Ref: "public"."infiltrationmeta"."infiltrationid" - "public"."soilwaterretention"."infiltrationid"

Ref: "public"."soilwaterretention"."infiltrationid" - "public"."hydraulicconductivity"."infiltrationid"

Ref: "public"."infiltrationmeta"."infiltrationid" - "public"."soilmoisture"."infiltrationid"

Ref: "public"."soilmoisture"."infiltrationid" - "public"."infiltrationrate"."infiltrationid"

Ref: "public"."infiltrationrate"."infiltrationid" - "public"."soilwaterretentioncurve"."infiltrationid"

Ref: "public"."soilmoisture"."method" - "public"."smmethod"."method"
```

### Figure

<figure>
<a href="../../images/DBML_schema-infiltration.png">
<img src="../../images/DBML_schema-infiltration.png"></a>
<figcaption>Infiltration DBML database structure. The lower row of tables are optional - third party solutions are possible to use and then only the top rows need to be filled. The value of the data will, however, decrease significantly if that route is chosen.</figcaption>
</figure>
