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

The schema _sites_ is intended for registering the AI4SH pilot sites and sample points. To cover the different types of pilots within AI4SH, three hierarchical levels are suggested:

1. _pilot_ (whether a single or multiple location(s)) (grand parent),
2. _site_ (contiguous sampling area with multiple sample point, whether field[s] or farm[s]) (parent), and
3. _samplepoint_ (the actual sample points) (children).

Each sample point (child) must belong to a site (parent) that must belong to a pilot (grand parent). Note that some pilots will only have a single site, whereas others might have 10, or even more.

In addition, the _sites_ schema also contains the tables that describe both constant (e.g. topography, tables: _samplepoint_landscape_, _major_landform_ and _hillslope_position_) and variable (e.g. land use, tables: _land_use_cover_, _samplepoint_state_ and _crop_growth_stage_code_) characteristics for the (above ground) landscape surrounding each sample point. The landscape description in the database should represent an area of approximately 25 m<sup>2</sup> centred around the central pit.

Also the weather conditions at time of sampling and over the 24 hours prior to the actual sampling is included in the _sites_ schema (table: _samplepoint_weather_state_).

The table *samplepoint_cultivation_history* contains boolean columns for _baresoil_ and _tillage_, numerical columns for _irrigation_ (mm),  *manure_fertilisation* (kg<sup>.</sup>ha<sup>-1</sup>), *chemical_fertilisation* (kg<sup>.</sup>ha<sup>-1</sup>), _pesticides_ (kg<sup>.</sup>ha<sup>-1</sup>) and _yield_ (kg<sup>.</sup>ha<sup>-1</sup>), and a text column for dominating _crop_. For interpreting and modelling some of the in-situ information retrieved, the detailed management at each sample point (the 25 m<sup>2</sup> mentioned above) is required going back five years. Thus, the table *samplepoint_cultivation_history* contains five (5) columns for each attribute, for the present year and each of the preceding 4 years.

The table _pilot_insitu_methods_ is intended for defining which in-situ observations that are supposed to be done at each pilot site. Thus all columns in this table, except the pilot universally unique id (UUID), are boolean variables. Table 1 lists all the in-situ methods that are considered and available as part of WP4 of AI4SH.

Table 1. In-situ data parameters and methods

___________________________________________________________

| parameter | target | method | comment |
| :--------- | :--------- | :--------- | :--------- |
| wet_chemistry | soil constituents | wet chemistry | in-lab<sup>1</sup> (following LUCAS) |
| eDNA | soil biodiversity and function | DNA sequencing | in-lab<sup>1</sup> (specialist) |
| macrofauna_full | soil biodiversity | hand sorted monolith | in-field<sup>2</sup> (specialist) |
| macrofauna_simple | earth worm count | mustard | in-field<sup>3</sup> (citizen scientist) |
| microbiometer | soil metabolism | commercial kit | in-kitchen<sup>4</sup> (citizen scientist) |
| moulder | aggregate stability | mobile phone app | in-kitchen<sup>4</sup> (citizen scientist) |
| soil_spectra_NO | soil constituents | scanning | in-field<sup>3</sup> (citizen scientist) |
| soil_spectra_MX | soil constituents | scanning | in-kitchen<sup>4</sup> (citizen scientist) |
| soil_spectra_DS | soil constituents | scanning | in-lab<sup>1,3</sup> (specialist, citizen scientist)|
| infiltration_rate | infiltration rate | constant head | in-field<sup>3</sup> (citizen scientist) |
| penetrometer_moisture | soil moisture | electronic | in-field<sup>3</sup> (citizen scientist) |
| penetrometer_salinity | soil salinity | electronic | in-field<sup>3</sup> (citizen scientist) |
| penetrometer_NPK | soil nutrients | electronic | in-field<sup>3</sup> (citizen scientist) |
| penetrometer_all | soil moisture, salinity & nutrients | electronic | in-field<sup>3</sup> (citizen scientist) |
| ise-ph | soil pH | ISE | in-field<sup>3</sup> (citizen scientist) |

___________________________________________________________

<sup>1</sup> <span style="font-size:0.85em;">= Field work includes sampling, packing, storing and shipping,</span><br>
<sup>2</sup> = <span style="font-size:0.85em;">Field work includes on-site analysis but requires specialist,</span><br>
<sup>3</sup> = <span style="font-size:0.85em;">Field work includes on-site analysis and can be done by citizen scientist,</span><br>
<sup>4</sup> = <span style="font-size:0.85em;">Field work includes sampling and analysis in kitchen environment that can be done by citizen scientist.</span>

**Note** that the sample and sampling as such is an a separate schema, and the observation/analysis results are in yet other schemas.

It would perhaps be better to split the schema into two separate schemas, or move some tables to the schema _sample_.

## Idea and objective

General information of pilots, fields/farms and even the sample-points should be possible to fill in already before conducting any in-situ observation. Including the stratified random distribution of sample points assuring a statistically valid result representing the theoretical distribution of a-priori known (and mapped) landscape conditions. When this data is entered each pilot, field/farm and sample point will get a Universally Unique Id (UUID), This UUID is then used as the link for all sub-sequent data capture related to pilots, fields/farms and sample points.

If would be possible to also fill some default data in the tables representing steady state properties (e.g. topographic features in the table _samplepoint_landscape_) from spatial data, and just let the sampler confirm this data in the field.

The registering of the pilots, field/farms and sample points, in turn, requires UUIDs recognising the user(s) that are responsible.

### DBML

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'AI4SH schema for sites'
}

Table users.user {
  userid UUID
}

Table pilot {
  pilotid UUID
  contact_person UUID
  responsible_person UUID
  pilotname TEXT [pk]
  country_iso2 char(2) [pk]
  pilot_area ST_Area
  description TEXT
  url TEXT
}

Table site {
 siteid UUID
 pilotid UUID [pk]
 sitename TEXT [pk]
 responsible_person UUID
 site_area ST_Area
 description TEXT
 url TEXT
}

Table samplepoint {
  pointid UUID
  sitename TEXT [pk]
  siteid UUID [pk]
  latitude NUMERIC
  longitude NUMERIC
  elevation NUMERIC
  position_error NUMERIC
  point_moved BOOLEAN
}

Table samplepoint_landscape { // constant
  pointid UUID [pk]
  siteid UUID
  observation_date DATE
  responsible_person UUID
  major_landform_code char(2)
  hillslope_position_code char(2)
  upward_slope SMALLINT
  downward_slope SMALLINT
}

Table major_landform {  // constant
  major_landform_code char(2) [pk]
  major_landform TEXT
  characteristics TEXT
}
// major_landform alternatives: "level", "sloping", "steep", "composite"

Table hillslope_position {
  hillslope_position_code char(2) [pk]
  hillslope_position TEXT
  characteristics TEXT
}

// hillslope_position alternatives: "plain", "slope shoulder", "midslope", "footslope", "upland valley", "mesa", "peak/major ridge", "valley in plain", "canyon", "large valley", "local ridge in slope", "local ridge in plain"

// only updated if changed from previous recording
Table land_use_cover {
  pointid UUID [pk]
  date DATE [pk]
  protected BOOLEAN
  managed BOOLEAN
  organic_farming BOOLEAN
  dominant_land_use_code char(2)
}

Table dominant_land_use {
    dominant_land_use_code char(2)
    dominant_land_use TEXT
    characteristics TEXT
}

// dominant_land_use alternatives: "annual crop", "perennial crop", "annual agroforestry", "pasture/rangeland", "other grassland" "fallow", "woodlot", "broadleaf forest", "needle leaf forest", "evergreen forest", "deciduous forest", "protected", "garden", "urban", "tundra", "desert", "other"

Table samplepoint_state { // for each sample occasion at samplepoint
  pointid UUID [pk]
  date DATE [pk]
  responsible_person UUID
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
  pointid UUID [pk]
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

Table samplepoint_weather_state { // for each sample occasion
  pointid UUID [pk]
  date DATE [pk]
  sky_condition_code char(2)
  rain_mm_1h SMALLINT
  rain_mm_24h SMALLINT
  temperature_1h_mean SMALLINT
  temperature_24_max SMALLINT
  temperature_24_min SMALLINT
}

Table sky_condtion {
  sky_condition_code char(2) [pk]
  sky_condition VARCHAR(8)
}
// erosion_conservation_measures alternatives: "clear", "cloudy", "changing", "rainy"

// This table should be filled for each pilot site after the wish-list created by WP6
// The default setting translates to each point sampling event were the sampler can
// override the default setting (skipping or adding in-situ method for that point)
Table pilot_insitu_methods {
  pilotid UUID [pk]
  wet_chemistry BOOLEAN
  eDNA BOOLEAN
  macrofauna_full BOOLEAN
  macrofauna_simple BOOLEAN
  micobiometer BOOLEAN
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

Ref: "users"."user"."userid" < "public"."pilot"."contact_person"

Ref: "users"."user"."userid" < "public"."pilot"."responsible_person"

Ref: "users"."user"."userid" < "public"."site"."responsible_person"

Ref: "users"."user"."userid" < "public"."samplepoint_state"."responsible_person"

Ref: "users"."user"."userid" < "public"."samplepoint_landscape"."responsible_person"

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
```

### Figure

<figure>
<a href="../../images/DBML_schema-pilots.png">
<img src="../../images/DBML_schema-pilots.png"></a>
<figcaption>Pilots and sites DBML database structure. The table "samplepoint_cultivation_history" is shortened (only 3 years of history shown but table above contains 5 years of history for each attribute).</figcaption>
</figure>
