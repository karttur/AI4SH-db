---
layout: post
title: "Schema: macrofauna"
categories: ai4sh-db
excerpt: "AI4SH schema macrofauna"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-03-10 11:27'
modified: '2024-06-19'
comments: true
share: true

---

## Outline

Macrofauna observations will neither be taken at all pilots, nor at all sites/points at the pilots where it will actually be included. Whether or not to do a macrofauna analysis is defined in the _samples_ schema.

The default method for macrofauna analysis is to extract a soil monolith of 25 * 25 cm and hand sort the macrofauna separately from three different layers:

- litter (on top of the actual soil profile),
- organic (O-layer), and
- top soil (A-layer).

To accommodate the taxa from the different layers, the schema includes three replicas of the table for registering observed taxa:

- observation_litter_layer
- observation_organic_layer
- observation_topsoil_layer

Each table contain columns for _taxa_, _life_cycle_stage_code_, _n_specimen_ (number of specimens) and _g_total_ (total weight of taxa in grams), and the _sampleid_ (linking to the sample event and further up the chain to the sample point).

The landscape (above ground) characteristics and the cultivation methods and its history are covered in the _sites_ schema. These are linked to the schema _macrofauna_ where each record inherits the sample id (sampleid) from the schema _samples_. The analyser of the macrofauna must be registered separately, again by linking to the _users_ schema.

## Idea and objective

The schema _macrofauna_ is one of several sibling schemas for registering in-situ data analysis results. For all of these siblings, information on the sample point (pilot site), are linked via the schema _sites_. Information on the sampling event is linked via the _samples_ schema.

To insert a macrofauna observation into the observation tables, the taxa to insert must be registered in the _taxa_stage_ table. All insertions should use the latin name (regardless of the taxonomic level) and no manual entry should be allowed. The only way to enter a taxa should be via a drop down menu listing all allowed entries. To support the field observer the table storing all the allowed taxa could also have vernacular names. But only to support searching for the observation - once entered into the database it should automatically be the latin name regardless of how the observer identified the taxa.

New taxa should only be allowed for expert users to add.

## Macrofauna wiki page

[https://animaldiversity.org](https://animaldiversity.org)

### DBML

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'AI4SH schema for macrofauna measurements'
}

Table users.user {
  userid SERIAL
}

Table samples.sample_event {
  sampleid SERIAL
  siteid INTEGER [pk]
}

Table sampling_technique {
  sampleid INTEGER [pk]
  userid INTEGER
  sample_dim_cm_x smallint [default: 25]
  sample_dim_cm_y smallint [default: 25]
  sample_dim_cm_z smallint [default: 30]
  extraction_method TEXT [default: "hand_sorting"]
  litter_layer boolean [default: True]
  organic_layer boolean [default: False]
  topspil_layer boolean [default: True]
}

Table extraction_method {
  extraction_method varchar(32) [pk]
}
// extraction_method alternatives: "hand_sorting", "mustard_oil", "mustard_powder"

Table observation_litter_layer {
  sampleid INTEGER [pk]
  taxa TEXT [pk]
  life_cycle_stage_code char[1] [pk]
  n_specimen smallint
  g_total real  
}

Table observation_organic_layer {
  sampleid INTEGER [pk]
  taxa TEXT [pk]
  life_cycle_stage_code char[1] [pk]
  n_specimen smallint
  g_total real
}

Table observation_topsoil_layer {
  sampleid INTEGER [pk]
  taxa TEXT [pk]
  life_cycle_stage_code char[1] [pk]
  n_specimen smallint
  g_total real
}

Table life_cycle_stage {
  life_cycle_stage_code char[1] [pk]
  life_cycle_stage varchar(16)
}
// life_cycle_stage alternatives:  A=adult, C=cocoon, L = Larvae, J=juvenile, N= Nymph, E= Egg, P=Pupa,


Table taxa_stage {
  taxa TEXT [pk]
  taxa_alternative TEXT [pk]
  life_cycle_stage_code char(1) [default: 'A', pk]
  common_english_name TEXT[]
  taxonomic_level TEXT [pk]
  class TEXT
  order TEXT
  family TEXT
  genus TEXT [default: 'NA']
  species TEXT [default: 'NA']
  info TEXT
  url TEXT
}

Table taxonomic_level {
  taxonomic_level TEXT
}
// taxonomic_levels alternatives:  Species, Genus, Family, Order, Class, Phylum, Kingdom, Domain

REF: samples.sample_event.sampleid - sampling_technique.sampleid

REF: users.user.userid - sampling_technique.userid

REF: sampling_technique.sampleid - observation_litter_layer.sampleid

REF: sampling_technique.sampleid - observation_organic_layer.sampleid

REF: sampling_technique.sampleid - observation_topsoil_layer.sampleid

Ref: sampling_technique.extraction_method - extraction_method.extraction_method

Ref: "public"."observation_litter_layer"."taxa" < "public"."taxa_stage"."taxa"

Ref: "public"."observation_litter_layer"."life_cycle_stage_code" < "public"."taxa_stage"."life_cycle_stage_code"

Ref: "public"."observation_organic_layer"."taxa" < "public"."taxa_stage"."taxa"

Ref: "public"."observation_organic_layer"."life_cycle_stage_code" < "public"."taxa_stage"."life_cycle_stage_code"

Ref: "public"."observation_topsoil_layer"."taxa" < "public"."taxa_stage"."taxa"

Ref: "public"."observation_topsoil_layer"."life_cycle_stage_code" < "public"."taxa_stage"."life_cycle_stage_code"

Ref: "public"."taxa_stage"."life_cycle_stage_code" < "public"."life_cycle_stage"."life_cycle_stage_code"

Ref: "public"."taxa_stage"."taxonomic_level" < "public"."taxonomic_level"."taxonomic_level"

```

### Figure

<figure>
<a href="../../images/DBML_schema-macrofauna.png">
<img src="../../images/DBML_schema-macrofauna.png"></a>
<figcaption>Macrofauna DBML database structure</figcaption>
</figure>
