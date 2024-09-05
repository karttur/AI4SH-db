---
layout: post
title: "Schemas"
categories: ai4sh-db
excerpt: "AI4SH database schemas."
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-02-27 11:27'
modified: '2024-09-02'
comments: true
share: true

---

## Outline

The first post in this series is an overview of all the schemas in the AI4SH database for in-situ data. It shows schemas and tables, rather than tables and fields as all the following posts. Not all tables are shown. Support, or catalog tables and alternative naming/labelling tables, are omitted from the overview.

The schemas, totalling 17, can be divided into 7 groups:

1. **users** - schema of all users that link to all other schemas for identifying the contact and responsible persons,
2. **sites** and **landscape** - for describing the sample sites and their permanent (landscape) characteristics,
3. **sample_event** - for each actual sampling taking place in-situ,
4. **landstate**, **fieldobs** and **weather** - direct observations of the state of the landscape, soil and weather at each sampling event,
5. **infiltration**, **penetrometer**, **ise** and **macrofauna** - in-situ methods that must (or should) be applied directly in the field,
6. **microbiometer**, **moulder**, **enzymes** and **spectra** - methods that are performed directly after field sampling, in e.g. a field lab or kitchen environment
7. **wetlab** and **edna** - methods that require samples to be preserved and sent off to specialist laboratories.

Some methods can be used both directly in the field, in a "kitchen" setting or in professional labs. The microbiometer test can be done direcly in the field or in a home-lab. Ion Selective Electrodes (ISE) and Soil spectroscopy can be used in all three settings (using a variety of different instruments).

## Idea and objective

The idea is that all types of in-situ data should be recorded in a single database and possible to extract in a coherent format for exploration and analysis.

### DBML

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs
// Tool: https://dbdiagram.io/d

// Support, or catalog tables and alterantive naming/labeling tables are not shown

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'AI4SH database schemas'
}

Table users {
  user Table
  orgnaisation TAble
}

Table sites {
 pilot Table
 site Table
 samplepoint Table
 point_insitu_methods Table
}

Table landscape {
 landscape Table
 major_landform Table
 hillslope_position Table
}

Table landstate {
 land_use_cover Table
 dominant_land_use Table
 samplepoint_state Table
 cultivation Table
 crop Table
}

Table weather {
sky_condition Table
ground_condition Table
}

Table sample_event {
sample_event Table
soil_class Table
sample_photo Table
sampling Table
soil_moisture_nominal Table
}

Table fieldobs {
sample_event Table
soil_class Table
sample_photo Table
sampling Table
soil_moisture_nominal Table
}

Table macrofauna {
  sampling_technique Table
  extraction_method Table
  identification_method Table
  observation Table
  monolith_photo Table
  monolith_depth Table
  taxa_photo Table
}

Table microbiometer {
  microbiometer Table
}

Table moulder {
  moulder Table
}

Table edna {
  samplelogistics Table
  metabarcodingpipeline Table
  ednaanalysismeta Table
  taxaheterogenity Table
  functionabundance Table
  laboratory Table
}

Table spectra {
  spectrometer Table
  scanmeta Table
  reflectancescan Table
  sampleprep Table
}

Table wetlab {
  labanalysismethod Table
  labanalysismeta Table
  labanalysisresults Table
  laboratory Table
}

Table penetrometer {
  penetrometer Table
  penetrometertypes Table
  penetrometercalib Table
  probemeta Table
  penetrometerobs Table
}

Table enzymes {
  eeameta Table
  eea Table
  eeaquality Table
}

Table ise {
  iseprobe Table
  isemeta Table
  iseanalysisresults Table
  isecalibration Table
}

Table infiltration {
  iseprobe Table
  isemeta Table
  iseanalysisresults Table
  isecalibration Table
}
```

### Figure

<figure>
<a href="../../images/DBML_schemas.png">
<img src="../../images/DBML_schemas.png"></a>
<figcaption>DBML database overview structure of schemas and tables</figcaption>
</figure>
