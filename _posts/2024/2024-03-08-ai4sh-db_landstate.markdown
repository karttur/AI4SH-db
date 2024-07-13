---
layout: post
title: "Schema: landstate"
categories: ai4sh-db
excerpt: "AI4SH schema landstate for recording the land use/management and crop conditions both historically and at the time of each sampling event."
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-03-08 11:27'
modified: '2024-07-13'
comments: true
share: true

---

## Outline

The **landstate** schema contains the tables that describe land use/management, crop and erosion states for each sample point at the time of each sampling event. It can also be used for describing historical crop management and yield. The state description in the database should represent an area of approximately 25 m<sup>2</sup> centred around the central pit of each sample point. Most observations requires visual (human) in-field observation. The observations are linked to each sample point (schema.table: _sites.pointid_), and via the observation date also to a particular sample event (schema.table: _sample.sampleid_).

The land use/cover is registered using three Boolean variables (protected, managed and organic_farming) plus selecting a single dominant land use from a predefined catalog, _dominant_land_use_ (see DBML code below for alternatives). In addition to dominant land use, the presence of different vegetation groups should be recorded as boolean variables. The stage of the main crop is also registered, again only allowing stages predefined in the support table (catalog) _crop_growth_stage_code_ - see below in the DBML code. Also any signs of erosion and erosions conservation measures (predefined catalogue: _erosions_censervation_measure_code_) should be recorded.

As collected samples for further (laboratory) analysis by default must exclude rocks, stones and usually also gravel, the percentage of the landscape surface covered by rocks, stones and gravel must be estimated in the field by the sampling team. These observations are entered as numbers (between 0 and 100) directly in the table _samplepoint_state_.

The table _cultivation_ should be filled at each sample event, but can also be used for recording historical data. It contains boolean columns for _baresoil_ and _tillage_, numerical columns for _irrigation_ (mm),  *manure_fertilisation* (kg<sup>.</sup>ha<sup>-1</sup>), *chemical_fertilisation* (kg<sup>.</sup>ha<sup>-1</sup>), _pesticides_ (kg<sup>.</sup>ha<sup>-1</sup>) and three columns for yield (kg<sup>.</sup>ha<sup>-1</sup>). There are three columns for crop types and their respective yield:

- maincrop (main_yield (kg<sup>.</sup>ha<sup>-1</sup>)),
- catchcrop (catch_yield (kg<sup>.</sup>ha<sup>-1</sup>)), and
- treecrop (tree_yield (kg<sup>.</sup>ha<sup>-1</sup>)).

The three different crop types can occur together (in agroforestry farm with catch crops interspersed with the main crop). Thus all three should be possible to register for a single sample point. All the crop types allowed in the table _cultivation_ must be pre-registered in the catalogue _crop_, also defining defining which type (main, catch or tree) any particular crop can be registered as.

For interpreting and modelling some of the in-situ information retrieved, the detailed management at each sample point (the 25 m<sup>2</sup> mentioned above) is required going back five years. Thus, the table _cultivation_ requires registering the _cultivationyear_ and start and end of the growing season(s) (_cultivationstartdate_  and _cultivationenddate_). If the (main) crop is intercropped with a catch crop (or agroforestry) and the catch crop should be registered.

## Idea and objective

The landstate properties vary over time and with growing season. Most of the observations require on-site visual inspection. Most of the attributes are fairly easy to observe and register, with the filling of the database largely depending on predefined catalogues and lists to chose from. Historical attributes are not mandatory.

### DBML

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'AI4SH schema for landstate'
}

Table users.user {
  userid SERIAL
}

Table samplepoint {
  pointid INTEGER
}

Table land_use_cover {
  pointid INTEGER [pk]
  date DATE [pk]
  protected BOOLEAN
  managed BOOLEAN
  organic_farming BOOLEAN
  dominant_land_use_code char(2)
}

Table dominant_land_use {
    dominant_land_use_code char(2) [pk]
    dominant_land_use TEXT
    characteristics TEXT
}

// dominant_land_use alternatives: "annual crop", "perennial crop", "annual agroforestry", "pasture/rangeland", "other grassland" "fallow", "woodlot", "broadleaf forest", "needle leaf forest", "evergreen forest", "deciduous forest", "protected", "garden", "urban", "tundra", "desert", "other"

Table samplepoint_state { // for each sample occasion at samplepoint
  pointid INTEGER [pk]
  date DATE [pk]
  userid INTEGER
  trees_presence BOOLEAN
  shrubs_presence BOOLEAN
  graminoids_presence BOOLEAN
  forbs_presence BOOLEAN
  lichses_mosses_presence BOOLEAN
  crop_growth_stage_code char(2)
  visible_sheet_erosion BOOLEAN [DEFAULT: false]
  visible_rill_erosion BOOLEAN [DEFAULT: false]
  visible_gully_erosion BOOLEAN [DEFAULT: false]
  visible_wind_erosion BOOLEAN [DEFAULT: false]
  erosion_conservation_measure_code CHAR(2) [DEFAULT: 'NA']
  rock_cover_percent SMALLINT
  stone_cover_percent SMALLINT
  gravel_cover_percent SMALLINT
}

Table crop_growth_stage_code {
    crop_growth_stage_code char(2)
    crop_growth_stage TEXT
    characteristics TEXT
}

// crop_growth_stage_code alternatives: "no crop", "dormant", "sprouting", "vegetative", "blooming", "harvest ready" "harvest remnant", "other"

Table erosion_conservation_measure_code {
    erosion_conservation_measure_code char(2)
    erosion_conservation_measure TEXT
    characteristics TEXT
}
// erosion_conservation_measures alternatives: "NA", "vegetative", "structural", "other"

Table cultivation { // for each sample occasion at samplepoint
  pointid INTEGER [pk]
  cultivationyear SMALLINT [pk]
  cultivationseasonstartdate DATE [pk]
  cultivationseasonenddate DATE [pk]
  baresoil_over_whole_season BOOLEAN [DEFAULT: false]
  tillage BOOLEAN [DEFAULT: true]
  irrigation_mm SMALLINT [DEFAULT: 0]
  manure_fertilisation_kg_ha SMALLINT [DEFAULT: 0]
  chemical_fertilisation_kg_ha SMALLINT [DEFAULT: 0]
  pesticides_kg_ha SMALLINT [DEFAULT: 0]
  maincrop TEXT
  catchcrop TEXT
  main_yield_kg_ha SMALLINT [DEFAULT: 0]
  catch_yield_kg_ha SMALLINT [DEFAULT: 0]
}

Ref: "users"."user"."userid" - "public"."samplepoint_state"."userid"

Ref: "public"."samplepoint_state"."erosion_conservation_measure_code" - "public"."erosion_conservation_measure_code"."erosion_conservation_measure_code"

Ref: "public"."samplepoint_state"."pointid" - "public"."land_use_cover"."pointid"

Ref: "public"."samplepoint_state"."crop_growth_stage_code" - "public"."crop_growth_stage_code"."crop_growth_stage_code"

Ref: "public"."land_use_cover"."dominant_land_use_code" - "public"."dominant_land_use"."dominant_land_use_code"

Ref: "public"."samplepoint"."pointid" - "public"."samplepoint_state"."pointid"

Ref: "public"."land_use_cover"."pointid" - "public"."cultivation"."pointid"
```

### Figure

<figure>
<a href="../../images/DBML_schema-landstate.png">
<img src="../../images/DBML_schema-landstate.png"></a>
<figcaption>Landstate DBML database structure.</figcaption>
</figure>
