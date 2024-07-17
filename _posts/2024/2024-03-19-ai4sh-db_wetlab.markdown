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
modified: '2024-07-13'
comments: true
share: true

---

## Outline

The schema **wetlab** contains tables for different laboratory analysis methods and results. The default soil properties included in the table are the physico-chemical quantities that are collected as part of the European Statistical Office (EUROSTAT) Land Use and Coverage Area frame Survey (LUCAS). The LUCAS sampling is structured in 5 different modules of which 3 are reflected in the AI4SH schema **wetlab**. The remaining 2 LUCAS modules are covered by the AI4SH schemas **macrofauna** and **edna**, and **samples**, as outlined below.

___________________________________________________________

| LUCAS module | AI4SH DB schema |
| :-----------  | :----------- |
| 1 Physico-chemical properties | wetlab |
| 2 Soil biodiversity | macrofauna/edna |
| 3 Bulk density | wetlab |
| 4 Field measurements | samples |
| 5 Pollution | organic pollutants, pesticides residues |

___________________________________________________________

LUCAS module 1 Physico-chemical quantitative properties and analysis methods include:

___________________________________________________________

| quantity | abbr. | unit | ISO | quantcode |
| :------- | :---- | :------- | :------- | :------- |
| CaCO<sub>3</sub> | CaCO<sub>3</sub> | g kg<sup>-1</sup> | 10693:1995 | caco3.10693:1995.gkg |
| Coarse fragments | CF | % | 11464:2006 | cf.11464:2006.pct |
| Clay | clay | % | - | clay..pct |
| Silt | silt | % | - | silt..pct |
| Sand | sand | % | - | sand..pct |
| pH CaCl<sub>2</sub> | pH-CaCl<sub>2</sub>  | index | 10390:2005 | ph-cacl2.10390:2005.index |
| pH H<sub>2</sub>O | pH-H<sub>2</sub>O | index | 10390:2005 | ph-h2o.10390:2005.index |
| Organic carbon | OC | g kg<sup>-1</sup> | 10694:1995 | oc.10694:1995.pct |
| Total nitrogen | Ntot | g kg<sup>-1</sup> | 11261:1995 | ntot.11261:1995.gkg |
| Phosphorus content | Pext  | g kg<sup>-1</sup> | 11263:1194 | p.11263:1194.kg |
| Potassium  | Kext | g kg<sup>-1</sup> | USDA−NRCS | k.USDA−NRCS.cmolckg |
| Cation Exchange Capacity | CEC | cmol(+) kg<sup>-1</sup> | 11260:1994 | cec.11260:1994.cmolckg |
| Electical conductivity | EC | mS m<sup>-1</sup>  | 11265:1994 | ec.11265:1994.mSm |

___________________________________________________________

The translation between the LUCAS (ISO) methods and the USDA methods as coded in the Open Soil Spectral Library (OSSL) are given below - NOTE ALL NEEDS CONFIRMATION.

___________________________________________________________

| AI4SH quantcode | OSSL (usda) code | comment |
|  :------- | :------- | :------- |
| caco3.10693:1995.gkg | caco3_usda.a54_w.pct | confirm! |
| cf.11464:2006.pct | cf_usda.c236_w.pct | confirm! |
| clay..pct | clay.tot_usda.a334_w.pct | confirm! |
| silt..pct | silt.tot_usda.c62_w.pct | confirm! |
| sand..pct | sand.tot_usda.c60_w.pct | confirm! |
| ph-cacl2.10390:2005.index | ph.cacl2_usda.a481_index | confirm! |
| ph-h2o.10390:2005.index | ph.h2o_usda.a268_index | confirm! |
| oc.10694:1995.pct | oc_usda.c729_w.pct | confirm! |
| ntot.11261:1995.gkg | n.tot_usda.a623_w.pct | confirm! |
| p.11263:1194.kg | p.ext_usda.a274_mg.kg | confirm! |
| k.USDA−NRCS.cmolckg | k.ext_usda.a725_cmolc.kg | confirm! |
| cec.11260:1994.cmolckg | cec_usda.a723_cmolc.kg | confirm! |
| ec.11265:1994.mSm | ec_usda.a364_ds.m | confirm! |

___________________________________________________________

Additional methods can be added by following the same principles of identifying the quantity, ISO-method and reporting units - and combining these into the unique _quantcode_. The table below illustrates how to define soil nitrogen species analysed from dried samples (ISO 14255:1998).

___________________________________________________________

| quantity | abbr. | unit | ISO | quantcode |
| :------- | :---- | :------- | :------- | :------- |
| N-NO<sub>3</sub> | N-NO<sub>3</sub> | mg kg<sup>-1</sup> | 14255:1998 | n-no3.14255:1998.mgkg |
| N-NH<sub>4</sub> | N-NH<sub>4</sub> | mg kg<sup>-1</sup> | 14255:1998 | n-nh4.14255:1998.mgkg |

___________________________________________________________

All properties that are to be laboratory analysed must be listed in the table _labanalysismethod_. The default methods to apply are the 13 physico-chemical methods listed in LUCAS module 1 (above).

The table _methodtransfer_ is a support table for linking the ISO coded methods as defined by LUCAS to any other coding system. By default the USDA coding system as expressed in the Open Soil Spectral Library (OSSL) should be added. But also other, country specific, standards can be entered.

The sample to analyse is always the mixed sample from each sample event, where either only the topsoil (0-20 cm) or both the topsoil and subsoil (20-50 cm) are analysed. If both are analysed, two metadata records (table _labanalysismeta_) must be created for the same sample. The metadata should also include the name and contact of the (certified/centralised) laboratory that analysed the sample. The schema thus also contains a table for registering the laboratories used by AI4SH (_laboratory_).

The actual laboratory results, for quantities and units as defined in the table _labanalysismethod_, are registered in the table _labanalysisresults_. As the analysis actually done varies for each pilot/site/sample point, the table is built up for registering single analysis results against the _analysismeta_ (linked using the common field _labanalysisid_).

### DBML

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'AI4SH schema for wet laboratory data'
}

Table users.user {
  userid SERIAL
}

Table samples.sample_event {
  sampleid SERIAL
}

Table labanalysismethod {
  quantity TEXT [pk]
  isocode TEXT [pk]
  unit TEXT  
  lucasmodule CHAR(1)
  quantcode TEXT
  default Boolean
}
// default is True for the LUCAS module 1 13 physico-chemical properties, and false for all other quantities.
// The quantcode is a combination of "quantity"."isocode"."unit"

TABLE labanalysismeta {
  laboratorieid INTEGER [pk]
  sampleid INTEGER [pk]
  topsoil Boolean [pk]
  analysisdate date
  userid INTEGER
  labanalysisid SERIAL
}
// topsoil (true) represents 0-20 cm, subsoil (false) represents 20-50 cm.

TABLE labanalysisresults {
  labanalysisid INTEGER
  quantcode TEXT
  value REAL
}

Table laboratory {
  labname TEXT [pk]
  labadress TEXT [pk]
  labcountry char(2)
  laburl TEXT
  labcontact TEXT
  laboratorieid SERIAL
}

Table methodtransfer {
  quantcode TEXT [pk]
  coountry TEXT [pk]
  countyrcode TEXT
  info TEXT
  gain REAL [DEFAULT: 1]
  offset REAL [DEFAULT: 0]
}

Ref: "public"."laboratory"."laboratorieid" - "public"."labanalysismeta"."laboratorieid"

Ref: "samples"."sample_event"."sampleid" - "public"."labanalysismeta"."sampleid"

Ref: "users"."user"."userid" - "public"."labanalysismeta"."userid"

Ref: "public"."labanalysismeta"."labanalysisid" - "public"."labanalysisresults"."labanalysisid"

Ref: "public"."labanalysismethod"."quantcode" - "public"."methodtransfer"."quantcode"

Ref: "public"."labanalysismethod"."quantcode" - "public"."labanalysisresults"."quantcode"
```

### Figure

<figure>
<a href="../../images/DBML_schema-wetlab.png">
<img src="../../images/DBML_schema-wetlab.png"></a>
<figcaption>Wetlab DBML database structure</figcaption>
</figure>
