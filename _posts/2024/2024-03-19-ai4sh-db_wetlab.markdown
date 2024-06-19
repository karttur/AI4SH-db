---
layout: post
title: "Schema: wetlab"
categories: ai4sh-db
excerpt: "AI4SH schema wetlab laboratory data"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-03-18 11:27'
modified: '2024-06-03'
comments: true
share: true

---

## Outline

The schema _wetlab_ contains tables for different laboratory analysis methods and results. The default soil properties included in the table are the physico-chemical quantities that are collected as part of the European Statistical Office (EUROSTAT) Land Use and Coverage Area frame Survey (LUCAS). The LUCAS sampling is structured in 5 different modules of which 3 are reflected in the AI4SH schema _wetlab_. The remaining 2 LUCAS modules are covered by the AI4SH schemas _macrofauna_ and _edna_, and _samples_, as outlined below.

| LUCAS module | AI4SH DB schema |
| :-----------  | :----------- |
| 1 Physico-chemical properties | wetlab |
| 2 Soil biodiversity | macrofauna/edna |
| 3 Bulk density | wetlab |
| 4 Field measurements | samples |
| 5 Pollution | organic pollutants, pesticides residues |

LUCAS module 1 Physico-chemical quantitative properties and analysis methods include:

| quantity | abbr. | unit | ISO | OSSL (usda) |
| :------- | :---- | :------- | :------- | :------- |
| CaCO<sub>3</sub> | CaCO<sub>3</sub> | g kg<sup>2</sup> | 10693:1995 | caco3_usda.a54_w.pct |
| Coarse fragments | CF | % | 11464:2006 | cf_usda.c236_w.pct |
| Clay | clay | % | - | clay.tot_usda.a334_w.pct |
| Silt | silt | % | - | silt.tot_usda.c62_w.pct |
| Sand | sand | % | - | sand.tot_usda.c60_w.pct |
| pH CaCl<sub>2</sub> | pH-CaCl<sub>2</sub>  | index | 10390:2005 | ph.cacl2_usda.a481_index |
| pH H<sub>2</sub>O | pH-H<sub>2</sub>O | index | 10390:2005 | ph.h2o_usda.a268_index |
| Organic carbon | OC | g kg<sup>-1</sup> | 10694:1995 | oc_usda.c729_w.pct |
| Total nitrogen | Ntot | g kg<sup>-1</sup> | 11261:1995 | n.tot_usda.a623_w.pct |
| Phosphorus content | Pext  | g kg<sup>-1</sup> | 11263:1194 | p.ext_usda.a274_mg.kg |
| Potassium  | Kext | g kg<sup>-1</sup> | USDA−NRCS | k.ext_usda.a725_cmolc.kg |
| Cation Exchange Capacity | CEC | cmol(+) kg<sup>-1</sup> | 11260:1994 | cec_usda.a723_cmolc.kg |
| Electical conductivity | EC | mS m<sup>-1</sup>  | 11265:1994 | ec_usda.a364_ds.m |

All properties that are to be laboratory analysed must be listed in the table _labanalysismethod_. The default methods to apply are the 13 physico-chemical methods listed in LUCAS module 1 (above).

The table _methodtransfer_ is a support table for linking the ISO coded methods as defined by LUCAS and the USDA equivalent codes - as expressed in the OSSL online database. Additional tables defining other transfer functions might be required in the future.

The sample to analyse is always the mixed sample from each sample event, where either only the topsoil (0-20 cm) or both the topsoil and subsoil (20-50 cm) are analysed. If both are analysed, two metadata records (table _labanalysismeta_) must be created for the same sample. The metadata should also include the name and contact of the (certified/centralised) laboratory that analysed the sample. The schema thus also contains a table for registering the laboratories used by AI4SH (_laboratory_).

The actual laboratory results, for quantities and units as defined in the table _labanalysismethod_, are registered in the table _labanalysisresults_. As the analysis actually done varies for each pilot/site/sample point, the table is built up for registering single analysis results against the _analysismeta_ (lined using the common field _labanalysisuuid_).

### DBML

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'AI4SH schema for wet laboratory data'
}

Table users.user {
  userid UUID
}

Table samples.sample_event {
  sampleuuid UUID
}

Table labanalysismethod {
  quantity TEXT [pk]
  unit TEXT
  isocode TEXT
  lucasmodule CHAR(1)
  default Boolean
}
// default is True for the LUCAS module 1 13 physico-chemical properties, and false for all other quantitites.

TABLE labanalysismeta {
  laboratorieuuid UUID [pk]
  sampleuuid UUID [pk]
  topsoil Boolean [pk]
  useruuid UUID
  labanalysisuuid UUID
}
// topsoil (true) represents 0-20 cm, subsoil (false) represents 20-50 cm.

TABLE labanalysisresults {
  labanalysisuuid UUID
  quantity TEXT
  value REAL
}

Table laboratory {
  labname TEXT [pk]
  labadress TEXT [pk]
  labcountry char(2)
  laburl TEXT
  labcontact TEXT
  laboratorieuuid UUID
}

Table methodtransfer {
  isocode TEXT [pk]
  osslcode  TEXT [pk]
  gain REAL [DEFAULT: 1]
  offset REAL [DEFAULT: 0]
}

Ref: "public"."laboratory"."laboratorieuuid" - "public"."labanalysismeta"."laboratorieuuid"

Ref: "samples"."sample_event"."sampleuuid" - "public"."labanalysismeta"."sampleuuid"

Ref: "users"."user"."userid" - "public"."labanalysismeta"."useruuid"

Ref: "public"."labanalysismeta"."labanalysisuuid" < "public"."labanalysisresults"."labanalysisuuid"

Ref: "public"."labanalysismethod"."isocode" - "public"."methodtransfer"."isocode"

Ref: "public"."labanalysismethod"."quantity" - "public"."labanalysisresults"."quantity"
```

### Figure

<figure>
<a href="../../images/DBML_schema-wetlab.png">
<img src="../../images/DBML_schema-wetlab.png"></a>
<figcaption>Wetlab DBML database structure</figcaption>
</figure>
