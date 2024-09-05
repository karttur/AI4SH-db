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
modified: '2024-07-13'
comments: true
share: true
---

## Outline 1

eDNA will be centrally analysed within AI4SH and the analysis data entered after the laboratory analysis is completed. The AI4SH database will only hold deduced taxa and predicted functions, the complete sequence data will be stored with the [European Nucleotide Archive](https://www.ebi.ac.uk/ena/browser/home) or an equivalent facility. The method applied for eDNA analysis is called metabarcoding and is built up as a pipeline of several steps - all of which need to be recorded in the AI4SH database. As the costs are comparably high, eDNA analysis will be limited to a few pilots and not all points at these selected pilots.

## eDNA

eDNA is a method for estimating biodiversity and the occurrence of both different taxa and dominating biological functions (e.g. aerobic/anaerobic, processes in the nitrogen cycle). The eDNA analysed as part of AI4SH will be divided in the following taxonomic domains:

- Bacteria,
- Archaea, and
- Eukarya.

Eukarya will be analysed at higher taxonomic resolution, including for instance the sub-groups:
- Fungus (yeasts, moulds and mushrooms subgroup),
- Protists (polyphyletic subgroup),
- Tardigrade (micro animal subgroup),
- Naematode (roundworm subgroup),
- Arthropod (exoskeleton clad animal subgroup),
- Annelid (segmented worm subgroup)
- Rotifer (wheel animal subgroup)

As a baseline the listed taxonomic groups will be recorded using two indexes:
- richness (total number of species), and
- Shannon's diversity index.

In addition bacteria and fungi (and potentially other taxonomic groups) will be divided into predictive functional categories.

Bacterial predicted functions:
- chemoheterotrophs,
- N-fixers, and
- human pathogenes.

Fungal predicited functions:
- Ectomycorrhizal,
- Arbuscular mycorrhizal,
- saprotrophs, and
- plant pathogens.

Most of the listed eDNA attributes above require customised pipelines for the metabarcoding. This causes the meatabarcoding pipeline to be linked to the results instead of the sample (i.e. several pipelines will be applied to each sample).

In addition the overall community dissimilarity will be recorded as a metadata (as it does not reveal any taxa or function).

## Outline 2

The tables of the schema **edna** follows the principles of the schema [**wetlab**](../ai4sh-db_wetlab).

The sample to analyse is always the mixed sample from each sample event, where either only the topsoil (0-20 cm) or both the topsoil and subsoil (20-50 cm) are analysed. If both are analysed, two metadata records (table _ednaanalysismeta_) must be created for the same sampleevent. The metadata should also include the name and contact of the (certified/centralised) eDNA laboratory that analysed the sample, and the archive and accession code/id for the original sequence data.

The schema thus also contains tables for registering the laboratories used by AI4SH (_laboratory_) and the nucleotide sequence archives (_nucleotidesequencearchive_) where the full sequence data are stored.

Because the eDNA analysis includes both taxa and function, there are two separate support tables that list the possible forms (table: _taxa_) and functions (table: _function_) to register. The support tables must be filled beforehand by the specialists responsible. Only entries recorded in the support tables will then be allowed in the two tables holding the final analysis results, _taxaheterogenity_ and _functionabundance_. Each entry into the support tables (_taxa_ and _function_) must also record the status of each entry - revealing if it is Compulsory (C), Recommended (R), Optional(O), Experimental(E) or Pending(P).

As stated above, the methods composing the metabarcoding pipeline differs dependent on taxa and functions. Each unique combination of methods building up a pipeline must be registered in the table _metabarcodingpipeline_.

The generic steps forming the metabarcoding pipeline include:
- extraction,
- amplification,
- purification,
- sequencing, and
- bioinformatic treatment.

For each of the five steps only a code is entered in the table _metabarcodingpipeline_, with all methodological details cataloged in five support tables (see DBML code below).

The last step, bioinformatic treatment, requires a database against which the identified sequences are compared and the taxa/functions identified. The method steps above and the database employed all influence the identified taxa, and should be registered with each analysis result. Each combination of the five (5) process step plus the nucleotide library used must be registered. When registering, the combination is given a unique SERIAL id that must then follow each result (taxa and function) where this precise pipeline is applied. If one process (or the nucleotide library) is changed, a new SERIAL id must be registered and linked to the result using this alternative (updated) chain of process methods. It is thus possible to analyse the same sample using two different pipelines (e.g. by just running the same sequence against an updated version of the nucleotide library) and then register two different results. Thus it is also possible to compare the results of employing 2 different pipelines, e.g. for evaluating different methods.

As the transport and storage of soil samples influence the results, there is a special table for registering the sample logistics at the laboratory side of the logistics chain, _samplelogistics_. The handling associated with the field sampling side is recorded in the schema **samples**.

## Idea and objective

Both the taxa (along with richness and Shannon's diversity) and the functions are to some extent arbitrarily selected and defined. To allow other taxa and functions to be added, the schema **edna** is built up around support tables for _taxa_ and _functions_. As stated above, the main table for recording biodiversity, _taxaheterogenity_, and functions, _functionabundance_, can only be inserted with taxa and functions defined in the support tables. This ensures that all added records are consistent while also allowing for novel taxa and functions to be added.

### DBML

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs
// Tool: https://dbdiagram.io/d

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'AI4SH schema for eDNA'
}

Table users.user {
  userid SERIAL
}

Table samples.sample_event {
  sampleid SERIAL
}

Table samplelogistics {
  sampleid INTEGER
  preservation TEXT
  transport TEXT
  transportduration SMALLINT
  storage TEXT
  storageduration SMALLINT
}

Table metabarcodingpipeline {
  extractioncode CHAR[2] [pk]
  amplificationcode CHAR[2] [pk]
  purificationcode CHAR[2] [pk]
  sequencingcode CHAR[2] [pk]
  sequencereferencelibrarycode CHAR[2] [pk]
  bioinformatictreamentmentcode CHAR[2] [pk]
  ednapipelineid SERIAL
}

Table extraction {
  extractioncode CHAR[2] [pk]
  method TEXT
  url TEXT
}

Table amplification {
  amplificationcode CHAR[2] [pk]
  method TEXT
  url TEXT
}

Table purification {
  purificationcode CHAR[2] [pk]
  method TEXT
  url TEXT
}

Table sequencing {
  sequencingcode CHAR[2] [pk]
  method TEXT
  url TEXT
}

Table sequencereferencelibrary {
  sequencereferencelibrarycode CHAR[2] [pk]
  version TEXT
  url TEXT
}

Table bioinformatictreamentment {
  bioinformatictreamentmentcode CHAR[2] [pk]
  version TEXT
  url TEXT
}

TABLE nucleotidesequencearchive {
  databasename TEXT [pk]
  proprietor TEXT [pk]
  databaseurl TEXT [pk]
  info TEXT
  nucleotidesequencearchive SERIAL
}
// AI4SH will primarily use

TABLE ednaanalysismeta {
  laboratorieid INTEGER [pk]
  sampleid INTEGER [pk]
  topsoil Boolean [pk]
  analysisdate date
  userid INTEGER
  nucleotidesequencearchive INTEGER
  accessionid TEXT
  accessionurl TEXT
  labanalysisid SERIAL
  dissimilarity REAL
}
// topsoil (true) represents 0-20 cm, subsoil (false) represents 20-50 cm.

Table taxa {
  taxa TEXT [pk]
  domain TEXT
  subgroup TEXT
  statuscode CHAR(1)
  info TEXT
}
// support table listing taxa to add

Table status {
  statuscode CHAR(1)
  status TEXT
}
//  Support table for support tables taxa and function
// C:Compulsory; R:Recommended; O:Optional; E:Experimental; P:Pending;

TABLE taxaheterogenity {
  labanalysisid INTEGER [pk]
  ednapipelineid INTEGER [pk]
  bioinformatictreatmentdate date [pk]
  taxa TEXT [pk]
  richness REAL
  shannon REAL
}

Table function {
  function TEXT
  taxa TEXT
  functiontaxa TEXT [pk]
  statuscode CHAR(1)
  info TEXT
}
// support table with drop down alternatives for function to add

TABLE functionabundance {
  labanalysisid INTEGER [pk]
  ednapipelineid INTEGER [pk]
  bioinformatictreatmentdate date [pk]
  functiontaxa TEXT [pk]
  abundance REAL
}

Table laboratory {
  labname TEXT [pk]
  labadress TEXT [pk]
  labcountry char(2)
  laburl TEXT
  labcontact TEXT
  laboratorieid SERIAL
}

Ref: "public"."laboratory"."laboratorieid" - "public"."ednaanalysismeta"."laboratorieid"

Ref: "samples"."sample_event"."sampleid" - "public"."ednaanalysismeta"."sampleid"

Ref: "users"."user"."userid" - "public"."ednaanalysismeta"."userid"

Ref: "public"."taxa"."taxa" - "public"."taxaheterogenity"."taxa"

Ref: "public"."function"."functiontaxa" - "public"."functionabundance"."functiontaxa"

Ref: "public"."ednaanalysismeta"."labanalysisid" - "public"."taxaheterogenity"."labanalysisid"

Ref: "public"."ednaanalysismeta"."labanalysisid" - "public"."functionabundance"."labanalysisid"

Ref: "public"."nucleotidesequencearchive"."nucleotidesequencearchive" - "public"."ednaanalysismeta"."nucleotidesequencearchive"

Ref: "public"."function"."statuscode" - "public"."taxa"."statuscode"

Ref: "public"."taxa"."statuscode" - "public"."status"."statuscode"

Ref: "public"."samplelogistics"."sampleid" - "samples"."sample_event"."sampleid"

Ref: "public"."metabarcodingpipeline"."ednapipelineid" - "public"."taxaheterogenity"."ednapipelineid"

Ref: "public"."metabarcodingpipeline"."ednapipelineid" - "public"."functionabundance"."ednapipelineid"

Ref: "public"."extraction"."extractioncode" - "public"."metabarcodingpipeline"."extractioncode"

Ref: "public"."purification"."purificationcode" - "public"."metabarcodingpipeline"."purificationcode"

Ref: "public"."amplification"."amplificationcode" - "public"."metabarcodingpipeline"."amplificationcode"

Ref: "public"."sequencing"."sequencingcode" - "public"."metabarcodingpipeline"."sequencingcode"

Ref: "public"."sequencereferencelibrary"."sequencereferencelibrarycode" - "public"."metabarcodingpipeline"."sequencereferencelibrarycode"

Ref: "public"."bioinformatictreamentment"."bioinformatictreamentmentcode" - "public"."metabarcodingpipeline"."bioinformatictreamentmentcode"
```

### Figure

<figure>
<a href="../../images/DBML_schema-edna.png">
<img src="../../images/DBML_schema-edna.png"></a>
<figcaption>eDNA DBML database structure</figcaption>
</figure>
