---
layout: post
title: "Schema: eDNA"
categories: ai4sh-db
excerpt: "AI4SH schema eDNA"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-03-16 11:27'
modified: '2024-06-24'
comments: true
share: true
---

## eDNA

eDNA will be centrally analysed within AI4SH and the data entered after the laboratory analysis is completed. The costs are comparably high and eDNA analysis will be limited to a few pilots and not all points at these selected pilots.

eDNA is a method for estimating biodiversity and the occurrence of both different taxa and dominating biological function (e.g. aerobic/anaerobic, processes in the nitrogen cycle). The eDNA analysed as part of AI4SH will be divided in the following taxonomic domains:

- Bacteria,
- Archaea, and
- Eukarya.

The following Eukarya taxa (subgroups) will be analysed:
- Fungus (yeasts, moulds and mushrooms subgroup),
- Protists (polyphyletic subgroup),
- Tardigrade (micro animal subgroup),
- Naematode (roundworm subgroup),
- Arthropod (exoskeleton clad animal subgroup),
- Annelid (segmented worm subgroup)
- Rotifer (wheel animal subgroup)

As a baseline the listed taxonomic groups above will be recorded using two indexes:
- richness (total number of species), and
- Shannon's diversity index.

In addition bacteria and fungi will be divided into functional guilds.

Bacterial guilds:
- chemoheterotrophs,
- N-fixers, and
- human pathogenes.

Fungal guilds:
- Ectomycorrhizal,
- Arbuscular mycorrhizal,
- saprotrophs, and
- plant pathogens.

In addition the overall community dissimilarity will be recorded.

## Outline

The tables of the schema _edna_ follows the principles of the [schema _wetlab_](../ai4sh-db_wetlab).

The sample to analyse is always the mixed sample from each sample event, where either only the topsoil (0-20 cm) or both the topsoil and subsoil (20-50 cm) are analysed. If both are analysed, two metadata records (table _ednaanalysismeta_) must be created for the same sample. The metadata should also include the name and contact of the (certified/centralised) eDNA laboratory that analysed the sample. The schema thus also contains a table for registering the laboratories used by AI4SH (_laboratory_).

Because the eDNA analysis includes both taxa ("form") and guild ("function"), there are two separate support tables that list the possible forms (table: _taxa_) and functions (table: _guild_) to register. These must be filled beforehand by the specialists responsible. Only entries recorded in the support tables will then be allowed in the two tables holding the final analysis results, _taxaheterogenity_ and _guildabundance_.

The method applied for the identification of taxa and guilds, metabarcoding, include a number of steps once the sample is received by the laboratory:
- extraction,
- amplification,
- purification,
- sequencing, and
- informatics mining.

The last step, informatics mining, requires a database against which the identified sequences are compared and the taxa identified. The method steps above and the database employed all influence the identified taxa, and should be registered with each analysis. In the schema _edna_ this is done in the table _ednaanalysismethod_. Each combination of the five (5) process step plus DNA library used must be registered. When registering, the combination is given a UUID that must then follow each sample where this precise sequence of methods is applied. If one process (or the DNA library) is changed, a new UUID must be registered and linked to the samples using this alternative (updated) chain of process methods.

## Idea and objective

Both the taxa (along with richness and Shannon's diversity) and the functional guilds are to some extent arbitrarily selected and defined. To allow other taxa and guilds to be added, the schema _edna_ is built up around support tables for _taxa_ and _guilds_. As stated above, the main table for recording biodiversity, _taxaheterogenity_, and functional guilds, _guildabundance_, can only be inserted with taxa and guilds defined in the support tables. This ensures that all added records are consistent while also allowing for novel taxa and guilds to be added.

### DBML

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'AI4SH schema for eDNA'
}

Table users.user {
  userid UUID
}

Table samples.sample_event {
  sampleuuid UUID
  siteid UUID [pk]
}

Table ednaanalysismethod {
  extraction TEXT [pk]
  amplification TEXT [pk]
  purification TEXT [pk]
  sequencing TEXT [pk]
  dnareflibrary TEXT [pk]
  miningmethod TEXT [pk]
  ednamethoduuid UUID
}

TABLE ednaanalysismeta {
  ednamethoduuid UUID [pk]
  laboratorieuuid UUID [pk]
  sampleuuid UUID [pk]
  topsoil Boolean [pk]
  analysisdate date
  miningdate date
  useruuid UUID
  labanalysisuuid UUID
}
// topsoil (true) represents 0-20 cm, subsoil (false) represents 20-50 cm.

Table taxa {
  taxa TEXT [pk]
  domain TEXT
  subgroup TEXT
  info TEXT
}
// support table listing taxa to add

TABLE taxaheterogenity {
  labanalysisuuid UUID [pk]
  taxa TEXT [pk]
  richness REAL
  shannon REAL
}

Table guild {
  guild TEXT
  taxa TEXT
  guildtaxa TEXT [pk]
  info TEXT
}
// support table with drop down alternatives for guild to add

TABLE guildabundance {
  labanalysisuuid UUID [pk]
  guildtaxa TEXT [pk]
  abundance REAL
}

Table laboratory {
  labname TEXT [pk]
  labadress TEXT [pk]
  labcountry char(2)
  laburl TEXT
  labcontact TEXT
  laboratorieuuid UUID
}

Ref: "public"."laboratory"."laboratorieuuid" - "public"."ednaanalysismeta"."laboratorieuuid"

Ref: "samples"."sample_event"."sampleuuid" - "public"."ednaanalysismeta"."sampleuuid"

Ref: "users"."user"."userid" - "public"."ednaanalysismeta"."useruuid"

Ref: "public"."taxa"."taxa" - "public"."taxaheterogenity"."taxa"

Ref: "public"."guild"."guildtaxa" - "public"."guildabundance"."guildtaxa"

Ref: "public"."ednaanalysismeta"."labanalysisuuid" - "public"."taxaheterogenity"."labanalysisuuid"

Ref: "public"."ednaanalysismeta"."labanalysisuuid" - "public"."guildabundance"."labanalysisuuid"

Ref: "public"."ednaanalysismethod"."ednamethoduuid" - "public"."ednaanalysismeta"."ednamethoduuid"
```

### Figure

<figure>
<a href="../../images/DBML_schema-edna.png">
<img src="../../images/DBML_schema-edna.png"></a>
<figcaption>eDNA DBML database structure</figcaption>
</figure>
