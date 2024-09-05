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
date: '2024-03-03 11:27'
modified: '2024-07-13'
comments: true
share: true
---

## Outline

The schema **sites** is intended for registering the AI4SH pilots, sites and sample points. Note that the schema can also be used for registering additional (non-official AI4SH) pilots, sites and sample points. To cover the different types of pilots, three hierarchical levels are suggested:

1. _pilot_ (whether a single or multiple location(s)) (grand parent),
2. _site_ (contiguous sampling area with multiple sample point, whether field[s] or farm[s]) (parent), and
3. _samplepoint_ (the actual sample points) (children).

Each sample point (child) must belong to a site (parent) that must belong to a pilot (grand parent). Note that some pilots will only have a single site, whereas others might have 10, or even more.

The table _point_insitu_methods_ is intended for defining the default in-situ observations that are supposed to be done at each sample point. Thus all columns in this table, except the pointid, are boolean variables. The table _point_insitu_methods_ should ideally be filled before any field work commence. Table 1 lists all the in-situ methods that are considered and available as part of WP4 of AI4SH.

_Table 1. In-situ data parameters and methods_

___________________________________________________________

| parameter | target | method | comment |
| :--------- | :--------- | :--------- | :--------- |
| wet_chemistry | soil constituents | wet chemistry | in-lab<sup>1</sup> (following LUCAS) |
| eDNA | soil biodiversity and function | DNA sequencing | in-lab<sup>1</sup> (specialist) |
| macrofauna | soil biodiversity | monolith excavation | in-field<sup>2,3</sup> (specialist, citizen scientist) |
| microbiometer | soil metabolism | commercial kit | in-kitchen<sup>4</sup> (citizen scientist) |
| moulder | aggregate stability | mobile phone app | in-kitchen<sup>4</sup> (citizen scientist) |
| fieldobs | soil density & moisture | soil cylinder | in-field<sup>4</sup> (citizen scientist) |
| soil_spectra_NO | soil constituents | spectroscopy | in-field<sup>3</sup> (citizen scientist) |
| soil_spectra_MX | soil constituents | spectroscopy | in-kitchen<sup>4</sup> (citizen scientist) |
| soil_spectra_DS | soil constituents | spectroscopy | in-lab<sup>1,3</sup> (specialist, citizen scientist)|
| enzymatic activity | extracellular enzymatic activity | SEAR<sup>5</sup> | in-kitchen<sup>4</sup> (citizen scientist) |
| penetrometer | soil moisture, salinity & nutrients | electronic | in-field<sup>3</sup> (citizen scientist) |
| pH | soil pH | ISE<sup>6</sup> | in-field<sup>3</sup> (citizen scientist) |
| infiltration | hydraulics | BEST<sup>7</sup> | in-field<sup>3</sup> (citizen scientist) |


___________________________________________________________

<sup>1</sup> <span style="font-size:0.85em;">= Field work includes sampling, packing, storing and shipping.</span><br>
<sup>2</sup> = <span style="font-size:0.85em;">Field work includes on-site analysis requiring specialist.</span><br>
<sup>3</sup> = <span style="font-size:0.85em;">Field work includes on-site analysis and can be done by citizen scientist.</span><br>
<sup>4</sup> = <span style="font-size:0.85em;">Field work includes sampling and kitchen analysis that can be done by citizen scientist.</span><br>
<sup>5</sup> = <span style="font-size:0.85em;">Soil Enzymatic Activity Reader - see schema on [enzymes](../ai4sh-db_enzymes).</span><br>
<sup>6</sup> = <span style="font-size:0.85em;">Ion Selective Electrode - see schema on [ise](../ai4sh-db_ise).</span><br>
<sup>7</sup> = <span style="font-size:0.85em;">Beerkan Estimation of Soil Transfer parameters [infiltration](../ai4sh-db_infiltration).</span><br>

## Idea and objective

The schema **sites** is primarily for registering AI4SH official pilots, sites and sample points. But it is possible to add additional data with the boolean variable pilot.ai4shpilot set to false.

The registering of the pilots, field/farms and sample points requires recognising the responsible user(s).

### DBML

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs
// Tool: https://dbdiagram.io/d

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'AI4SH schema for sites'
}

Table users.user {
  userid SERIAL
}

Table pilot {
  pilotid SERIAL
  contact_person INTEGER
  userid INTEGER
  pilotname TEXT [pk]
  country_iso2 char(2) [pk]
  pilot_area ST_Area
  description TEXT
  url TEXT
  ai4shpilot BOOLEAN
}

Table site {
 siteid SERIAL
 pilotid INTEGER [pk]
 sitename TEXT [pk]
 userid INTEGER
 site_area ST_Area
 description TEXT
 url TEXT
}

Table samplepoint {
  pointid SERIAL
  sitename TEXT [pk]
  siteid INTEGER [pk]
  latitude NUMERIC
  longitude NUMERIC
  elevation NUMERIC
  position_error NUMERIC
  point_moved BOOLEAN
}

// This table should be filled for each pilot site after the wish-list created by WP6
// The default setting translates to each point sampling event were the sampler can
// override the default setting (skipping or adding in-situ method for that point)
Table point_insitu_methods {
  pontid INTEGER [pk]
  wet_chemistry BOOLEAN
  eDNA BOOLEAN
  macrofauna BOOLEAN
  microbiometer BOOLEAN
  moulder BOOLEAN
  fieldobs BOOLEAN
  soil_spectra_NO BOOLEAN
  soil_spectra_MX BOOLEAN
  soil_spectra_DS BOOLEAN
  penetrometer_NPKPHCTH BOOLEAN
  enzymes BOOLEAN
  ise_pH BOOLEAN
  infiltration BOOLEAN
}

Ref: "users"."user"."userid" - "public"."pilot"."contact_person"

Ref: "users"."user"."userid" - "public"."pilot"."userid"

Ref: "public"."site"."siteid" < "public"."samplepoint"."siteid"

Ref: "public"."samplepoint"."pointid" - "public"."point_insitu_methods"."pontid"

Ref: "public"."pilot"."pilotid" < "public"."site"."pilotid"

Ref: "public"."site"."userid" - "users"."user"."userid"
```

### Figure

<figure>
<a href="../../images/DBML_schema-pilots-sites.png">
<img src="../../images/DBML_schema-pilots-sites.png"></a>
<figcaption>Pilots and sites DBML database structure. </figcaption>
</figure>
