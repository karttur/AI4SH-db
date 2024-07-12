---
layout: post
title: "Schema: sites"
categories: ai4sh-db
excerpt: "AI4SH schema sites contains hierarchical information on i) pilot sites, divided into ii) fields/farms and iii) individual sample points."
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

The schema _sites_ is intended for registering the AI4SH pilots, sites and sample points. Note that the schema can also be used registering additional (unofficial AI4SH) pilots, sites and sample points. To cover the different types of pilots, three hierarchical levels are suggested:

1. _pilot_ (whether a single or multiple location(s)) (grand parent),
2. _site_ (contiguous sampling area with multiple sample point, whether field[s] or farm[s]) (parent), and
3. _samplepoint_ (the actual sample points) (children).

Each sample point (child) must belong to a site (parent) that must belong to a pilot (grand parent). Note that some pilots will only have a single site, whereas others might have 10, or even more.

In addition, the _sites_ schema also contains the tables that describe both constant (e.g. topography, tables: _samplepoint_landscape_, _major_landform_ and _hillslope_position_) and variable (e.g. land use, tables: _land_use_cover_, _samplepoint_state_ and _crop_growth_stage_code_) characteristics for the (above ground) landscape surrounding each sample point. The landscape description in the database should represent an area of approximately 25 m<sup>2</sup> centred around the central pit of each sample point.

Also the weather conditions at time of sampling and over the 24 hours prior to the actual sampling is included in the _sites_ schema (table: _samplepoint_weather_state_).

The table *samplepoint_cultivation_history* contains boolean columns for _baresoil_ and _tillage_, numerical columns for _irrigation_ (mm),  *manure_fertilisation* (kg<sup>.</sup>ha<sup>-1</sup>), *chemical_fertilisation* (kg<sup>.</sup>ha<sup>-1</sup>), _pesticides_ (kg<sup>.</sup>ha<sup>-1</sup>) and _yield_ (kg<sup>.</sup>ha<sup>-1</sup>), and a text column for dominating _crop_. For interpreting and modelling some of the in-situ information retrieved, the detailed management at each sample point (the 25 m<sup>2</sup> mentioned above) is required going back five years. Thus, the table *samplepoint_cultivation_history* contains five (5) columns for each attribute, for the present year and each of the preceding 4 years.

The table _pilot_insitu_methods_ is intended for defining which in-situ observations that are supposed to be done at each pilot site. Thus all columns in this table, except the pilotid, are boolean variables. Table 1 lists all the in-situ methods that are considered and available as part of WP4 of AI4SH.

_Table 1. In-situ data parameters and methods_

___________________________________________________________

| parameter | target | method | comment |
| :--------- | :--------- | :--------- | :--------- |
| wet_chemistry | soil constituents | wet chemistry | in-lab<sup>1</sup> (following LUCAS) |
| eDNA | soil biodiversity and function | DNA sequencing | in-lab<sup>1</sup> (specialist) |
| macrofauna_full | soil biodiversity | hand sorted monolith | in-field<sup>2</sup> (specialist) |
| macrofauna_simple | earth worm count | mustard | in-field<sup>3</sup> (citizen scientist) |
| microbiometer | soil metabolism | commercial kit | in-kitchen<sup>4</sup> (citizen scientist) |
| moulder | aggregate stability | mobile phone app | in-kitchen<sup>4</sup> (citizen scientist) |
| soil_spectra_NO | soil constituents | spectroscopy | in-field<sup>3</sup> (citizen scientist) |
| soil_spectra_MX | soil constituents | spectroscopy | in-kitchen<sup>4</sup> (citizen scientist) |
| soil_spectra_DS | soil constituents | spectroscopy | in-lab<sup>1,3</sup> (specialist, citizen scientist)|
| penetrometer_moisture | soil moisture | electronic | in-field<sup>3</sup> (citizen scientist) |
| penetrometer_salinity | soil salinity | electronic | in-field<sup>3</sup> (citizen scientist) |
| penetrometer_NPK | soil nutrients | electronic | in-field<sup>3</sup> (citizen scientist) |
| penetrometer_all | soil moisture, salinity & nutrients | electronic | in-field<sup>3</sup> (citizen scientist) |
| ise_ph | soil pH | ISE | in-field<sup>3</sup> (citizen scientist) |

___________________________________________________________

<sup>1</sup> <span style="font-size:0.85em;">= Field work includes sampling, packing, storing and shipping,</span><br>
<sup>2</sup> = <span style="font-size:0.85em;">Field work includes on-site analysis requiring specialist,</span><br>
<sup>3</sup> = <span style="font-size:0.85em;">Field work includes on-site analysis and can be done by citizen scientist,</span><br>
<sup>4</sup> = <span style="font-size:0.85em;">Field work includes sampling and analysis in kitchen environment that can be done by citizen scientist.</span>

**Note** that the sample and sampling as such is an a separate schema, and the observation/analysis results are in yet other schemas.

It would perhaps be better to split the schema into two separate schemas, or move some tables to the schema _sample_.

## Idea and objective

General information of pilots, fields/farms (sites) and even the sample-points should be possible to fill in already before conducting any in-situ observation. Including the stratified random distribution of sample points assuring a statistically valid result representing the theoretical distribution of a-priori known (and mapped) landscape conditions. When this data is entered each pilot, field/farm and sample point will get a SERIAL (automatic) id number, This id is then used as the link for all subsequent data capture related to pilots, fields/farms and sample points.

If would be possible to also fill some default data in the tables representing steady state properties (e.g. topographic features in the table _samplepoint_landscape_) from spatial data, and just let the sampler confirm this data in the field.

The registering of the pilots, field/farms and sample points, in turn, requires ids (the unique integer id assigned by the SERIAL data type) recognising the user(s) that are responsible.

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
  nr_topsoil_subsamples SMALLINT
  nr_subsoil_subsamples SMALLINT
  auger_depth_max_cm SMALLINT [DEFAULT: -99]
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

// crop_growth_stage_code alternatives: "dormant", "sprouting", "vegetative", "blooming", "harvest ready" "harvest remnant", "other"

Table erosion_conservation_measure_code {
    erosion_conservation_measure_code char(2)
    erosion_conservation_measure TEXT
    characteristics TEXT
}
// erosion_conservation_measures alternatives: "NA", "vegetative", "structural", "other"

Table samplepoint_cultivation_history { // for each sample occasion at samplepoint
  pointid INTEGER [pk]
  date DATE [pk]
  yr_0_baresoil_over_annual_cycle BOOLEAN [DEFAULT: false]
  yr_m1_baresoil_over_annual_cycle BOOLEAN [DEFAULT: false]
  yr_m2_baresoil_over_annual_cycle BOOLEAN [DEFAULT: false]
  yr_m3_baresoil_over_annual_cycle BOOLEAN [DEFAULT: false]
  yr_m4_baresoil_over_annual_cycle BOOLEAN [DEFAULT: false]
  yr_0_tillage BOOLEAN [DEFAULT: true]
  yr_m1_tillage BOOLEAN [DEFAULT: true]
  yr_m2_tillage BOOLEAN [DEFAULT: true]
  yr_m3_tillage BOOLEAN [DEFAULT: true]
  yr_m4_tillage BOOLEAN [DEFAULT: true]
  yr_0_irrigation_mm SMALLINT [DEFAULT: 0]
  yr_m1_irrigation_mm SMALLINT [DEFAULT: 0]
  yr_m2_irrigation_mm SMALLINT [DEFAULT: 0]
  yr_m3_irrigation_mm SMALLINT [DEFAULT: 0]
  yr_m4_irrigation_mm SMALLINT [DEFAULT: 0]
  yr_0_manure_fertilisation_kg_ha SMALLINT [DEFAULT: 0]
  yr_m1_manure_fertilisation_kg_ha SMALLINT [DEFAULT: 0]
  yr_m2_manure_fertilisation_kg_ha SMALLINT [DEFAULT: 0]
  yr_m3_manure_fertilisation_kg_ha SMALLINT [DEFAULT: 0]
  yr_m4_manure_fertilisation_kg_ha SMALLINT [DEFAULT: 0]
  yr_0_chemical_fertilisation_kg_ha SMALLINT [DEFAULT: 0]
  yr_m1_chemical_fertilisation_kg_ha SMALLINT [DEFAULT: 0]
  yr_m2_chemical_fertilisation_kg_ha SMALLINT [DEFAULT: 0]
  yr_m3_chemical_fertilisation_kg_ha SMALLINT [DEFAULT: 0]
  yr_m4_chemical_fertilisation_kg_ha SMALLINT [DEFAULT: 0]
  yr_0_pesticides_kg_ha SMALLINT [DEFAULT: 0]
  yr_m1_pesticides_kg_ha SMALLINT [DEFAULT: 0]
  yr_m2_pesticides_kg_ha SMALLINT [DEFAULT: 0]
  yr_m3_pesticides_kg_ha SMALLINT [DEFAULT: 0]
  yr_m4_pesticides_kg_ha SMALLINT [DEFAULT: 0]
  yr_0_crop TEXT
  yr_m1_crop TEXT
  yr_m2_crop TEXT
  yr_m3_crop TEXT
  yr_m4_crop TEXT
  yr_0_yield_kg_ha SMALLINT [DEFAULT: 0]
  yr_m1_yield_kg_ha SMALLINT [DEFAULT: 0]
  yr_m2_yield_kg_ha SMALLINT [DEFAULT: 0]
  yr_m3_yield_kg_ha SMALLINT [DEFAULT: 0]
  yr_m4_yield_kg_ha SMALLINT [DEFAULT: 0]
}

Ref: "users"."user"."userid" < "public"."pilot"."contact_person"

Ref: "users"."user"."userid" < "public"."pilot"."userid"

Ref: "users"."user"."userid" < "public"."site"."userid"

Ref: "users"."user"."userid" < "public"."samplepoint_state"."userid"

Ref: "users"."user"."userid" < "public"."samplepoint_landscape"."userid"

Ref: "public"."samplepoint_landscape"."major_landform_code" < "public"."major_landform"."major_landform_code"

Ref: "public"."samplepoint_landscape"."hillslope_position_code" < "public"."hillslope_position"."hillslope_position_code"

Ref: "public"."samplepoint"."pointid" < "public"."samplepoint_landscape"."pointid"

Ref: "public"."samplepoint_landscape"."pointid" < "public"."samplepoint_state"."pointid"

Ref: "public"."samplepoint_state"."erosion_conservation_measure_code" < "public"."erosion_conservation_measure_code"."erosion_conservation_measure_code"

Ref: "public"."samplepoint_state"."pointid" < "public"."samplepoint_weather_state"."pointid"

Ref: "public"."samplepoint_state"."pointid" < "public"."land_use_cover"."pointid"

Ref: "public"."samplepoint_state"."crop_growth_stage_code" < "public"."crop_growth_stage_code"."crop_growth_stage_code"

Ref: "public"."samplepoint_weather_state"."sky_condition_code" < "public"."sky_condtion"."sky_condition_code"

Ref: "public"."land_use_cover"."dominant_land_use_code" < "public"."dominant_land_use"."dominant_land_use_code"

Ref: "public"."samplepoint_landscape"."pointid" < "public"."samplepoint_cultivation_history"."pointid"

Ref: "public"."samplepoint_weather_state"."pointid" < "public"."point_insitu_methods"."pointid"
```

### Figure

<figure>
<a href="../../images/DBML_schema-pilots.png">
<img src="../../images/DBML_schema-pilots.png"></a>
<figcaption>Pilots and sites DBML database structure. The table "samplepoint_cultivation_history" is shortened (only 3 years of history shown but table above contains 5 years of history for each attribute).</figcaption>
</figure>
