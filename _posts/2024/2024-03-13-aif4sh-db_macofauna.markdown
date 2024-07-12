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
date: '2024-03-13 11:27'
modified: '2024-06-19'
comments: true
share: true

---

## Outline

Macrofauna observations will neither be taken at all pilots, nor at all sites/points at the pilots where it will actually be included. Whether or not to do a macrofauna analysis at a particular sample point is defined in the _samples_ schema.

The default method for macrofauna analysis is to extract a soil monolith of 25 * 25 cm and hand sort the macrofauna separately from two different layers:

- litter (on top of the actual soil profile),
- top soil (0-20 cm).

To accommodate the taxa from the different layers, the schema includes two replicas of the table for registering observed taxa:

- observation_litter
- observation_topsoil

Each table contain columns for _taxa_, _life_cycle_stage_code_, _n_specimen_ (number of specimens) and _g_total_ (total weight of taxa in grams), and the _sampleid_ (linking to the sample event and further up the chain to the sample point).

The identification of taxa is by default manual, but semi-automated or fully automated methods using 2D or 3D images is tested as part of AI4SH. Thus each taxa result must be accompanied with the identification method applied. As the image based identification is under development, the support table for defining the identification method, _identification_method_, includes columns for defining both the camera and light setup and the substrate used as background.

The landscape (above ground) characteristics and the cultivation methods and its history are covered in the _sites_ schema. These are linked to the schema _macrofauna_ where each record inherits the sample id (sampleid) from the schema _samples_. The analyser of the macrofauna must be registered separately, again by linking to the _users_ schema.

## Idea and objective

The schema _macrofauna_ is one of several sibling schemas for registering in-situ data analysis results. For all of these siblings, information on the sample point (pilot site), are linked via the schema _sites_. Information on the sampling event is linked via the _samples_ schema.

To insert a macrofauna observation into the observation tables, the taxa to insert must be registered in the _taxa_stage_ table. All insertions should use the latin name (regardless of the taxonomic level) and no manual entry should be allowed. The only way to enter a taxa should be via a drop down menu listing all allowed entries. To support the field observer the table storing all the allowed taxa could also have vernacular names. But only to support searching for the observation - once entered into the database it should automatically be the latin name regardless of how the observer identified the taxa.

The initial default taxa to use are listed separately below. New taxa should only be allowed for expert users to add.

To allow the comparison of manual, semi-automated and fully automated macrofauna identification methods, the table _identification_methods_ is used for defining the precise identification method. Each recorded taxa in the two result tables (_observation_litter_ and _observation_topsoil_) must be linked to an identification method. This allows cross-comparison of different image based setups, both against each other and vis-a-vis the manual identification, at a taxa level.

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
  sample_dim_cm_z smallint [default: 20]
  extraction_method TEXT [default: "hand_sorting"]
  litter_layer boolean [default: True]
  topsoil_layer boolean [default: True]
}

Table extraction_method {
  extraction_method TEXT [pk]
}
// extraction_method alternatives: "hand_sorting", "mustard_oil", "mustard_powder"

Table identification_method {
  approach TEXT [pk]
  substrate TEXT [pk]
  camera [pk]
  setup [pk]
  lightsource [pk]
  methodcode SERIAL
}
// approach alternatives: "manual", "image"
// substrate alterantives: "NA", "whitepaper", "whitecloth", "mirror", "glass", "shoebox"
// camera alternatives: "NA", "singlemobile", "dualmobile", "singlesystemcamera", "dualsystemcamera", "3Dscan"
// setup alternatives: "NA", "handheld", "singletripod", "dualtripod", "3Dscan"
// lightsource alternatives: "NA", "outdoo", 'indoor', "flash", "singlelamp", "duallamp", "3Dscan"


Table observation_litter {
  sampleid INTEGER [pk]
  methodcode INTEGER [pk]
  taxa TEXT [pk]
  life_cycle_stage_code char[1] [pk]
  n_specimen smallint
  g_total real  
}

Table observation_topsoil {
  sampleid INTEGER [pk]
  methodcode INTEGER [pk]
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

REF: sampling_technique.sampleid - observation_litter.sampleid

REF: sampling_technique.sampleid - observation_topsoil.sampleid

Ref: sampling_technique.extraction_method - extraction_method.extraction_method

Ref: identification_method.methodcode - observation_litter.methodcode

Ref: identification_method.methodcode - observation_topsoil.methodcode

Ref: "public"."observation_litter"."taxa" < "public"."taxa_stage"."taxa"

Ref: "public"."observation_litter"."life_cycle_stage_code" < "public"."taxa_stage"."life_cycle_stage_code"

Ref: "public"."observation_topsoil"."taxa" < "public"."taxa_stage"."taxa"

Ref: "public"."observation_topsoil"."life_cycle_stage_code" < "public"."taxa_stage"."life_cycle_stage_code"

Ref: "public"."taxa_stage"."life_cycle_stage_code" < "public"."life_cycle_stage"."life_cycle_stage_code"

Ref: "public"."taxa_stage"."taxonomic_level" < "public"."taxonomic_level"."taxonomic_level"

```

Initial taxa to use for the macrofauna registration, their taxonomic levels and common English names. Note that some entries are overlapping in such way that a lower hierarchical level (e.g. family) is a breakout from a higher level (e.g order).

```
// taxa: Dermaptera
// alternative_taxa: acari
// code: -
// taxa_system_level: order
// common_english_name: ['earwigs']
// phylum: Arthropoda
// class: insecta
// order: Dermaptera
// family: NA

// taxa: Acarina
// alternative_taxa: acari
// code: AR
// taxa_system_level: order
// common_english_name: ['mites','ticks']
// class: arachnida
// order: acarina

// taxa Araneidae
// alternative_taxa: NA
// code: -
// taxa_system_level: family
// common_english_name: ['orb-weaver spiders']
// class: arachnida
// order: araneidae
// family: araneidae

// taxa: blattaria
// alternative_taxa: blattodea
// code: BL
// taxa_system_level: order
// common_english_name: ['cockroach']
// phylum: Arthropoda
// class: insecta
// order: Blattaria
// family: NA

// taxa chilopoda
// alternative_taxa: NA
// code: CH
// taxa_system_level: class
// common_english_name: ['centipedes']
// class: chilopoda
// order: NA
// family: NA

// taxa: Coleoptera
// alternative_taxa: NA
// code: CO
// taxa_system_level: order
// common_english_name: ['beetle']
// phylum: Arthropoda
// class: insecta
// order: Coleoptera
// family: NA

// taxa: Collembola
// alternative_taxa: NA
// taxa_system_level: class
// common_english_name: ['springtales']
// phylum: Arthropoda
// class: collembola
// order: NA
// family: NA

// taxa: Dictyoptera
// alternative_taxa: NA
// taxa_system_level: genus
// common_english_name: ['net-winged beetles']
// phylum: Arthropoda
// class: insecta
// order: Coleoptera
// family: Lycidae
// Genus: Dictyoptera

// taxa: Diplopoda
// code: DI
// alternative_taxa: NA
// taxa_system_level: class
// common_english_name: ['millipede']
// phylum: Arthropoda
// class: Diplopoda
// order: NA
// family: NA
// Genus: NA

// taxa: Diploura
// alternative_taxa: Diplura
// taxa_system_level: order
// common_english_name: ['two-pronged bristletails']
// phylum: Arthropoda
// class: Diplura
// order: NA
// family: NA
// Genus: NA

// taxa: Diptera
// alternative_taxa: NA
// code: DI
// taxa_system_level: order
// common_english_name: ['fly']
// phylum: Arthropoda
// class: insecta
// order: Diptera
// family: NA
// Genus: NA

// taxa: Gasteropoda
// alternative_taxa: Gastropoda
// code: GA
// taxa_system_level: class
// common_english_name: ['slugs', 'snails']
// phylum: Mollusca
// class: Gasteropoda
// order: NA
// family: NA
// Genus: NA

// taxa Hemiptera
// alternative_taxa: NA
// code: HM
// taxa_system_level: order
// common_english_name ['true bugs']
// phylum: Arthropoda
// class: insecta
// order: Hemiptera
// family: NA
// Genus: NA

// taxa Hymenoptera
// alternative_taxa: NA
// code: -
// taxa_system_level: order
// common_english_name ['sawfiles','wasps','bees','ants']
// phylum: Arthropoda
// class: insecta
// order: Hymenoptera
// family: NA
// Genus: NA

// taxa Formicidae
// alternative_taxa: NA
// code: AN
// taxa_system_level: order
// common_english_name ['ants']
// phylum: Arthropoda
// class: insecta
// order: Hymenoptera
// family: Formicidae
// Genus: NA

// taxa Lepidoptera
// alternative_taxa: NA
// code: LE
// taxa_system_level: order
// common_english_name ['butterflies']
// phylum: Arthropoda
// class: insecta
// order: Lepidoptera
// family: NA
// Genus: NA

// taxa: Termite
// alternative_taxa: NA
// code: TE
// taxa_system_level: family
// common_english_name: ['termite']
// phylum: Arthropoda
// class: insecta
// order: Blattodea
// family: Termite
// Genus: NA
```

### Figure

<figure>
<a href="../../images/DBML_schema-macrofauna.png">
<img src="../../images/DBML_schema-macrofauna.png"></a>
<figcaption>Macrofauna DBML database structure</figcaption>
</figure>
