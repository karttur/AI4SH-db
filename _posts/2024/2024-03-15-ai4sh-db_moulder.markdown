---
layout: post
title: "Schema: moulder"
categories: ai4sh-db
excerpt: "AI4SH schema moulder"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-03-15 11:27'
modified: '2024-07-13'
comments: true
share: true

---

## Outline

The schema **moulder** only contains a single table, for recording the results of the [Moulder: Soil aggregates](https://play.google.com/store/apps/details?id=slaker.sydneyuni.au.com.slaker) open source app. Results from the older versions of the app, called [slakes](https://play.google.com/store/apps/details?id=slaker.sydneyuni.au.com.slaker) can also be registered in the schema.

The soil sampling, including extraction method, depth, classification, photos etc., are recorded in the schema **samples**. The landscape (above ground) characteristics and the cultivation methods and its history are covered in the **sites**, **landscape** and **landsites** schemas.

## Moulder (slakes)

[Moulder: Soil aggregates](https://play.google.com/store/apps/details?id=slaker.sydneyuni.au.com.slaker) is an open source mobile app available on [Google Play](https://play.google.com/store/apps/details?id=slaker.sydneyuni.au.com.slaker) and [Apple App Store](https://apps.apple.com/us/app/slakes-soil-health-test/id6449554197).

The measurement of the soil aggregate stability rely on the mobile phone camera. The light settings to separate the soil aggregrates from the background can be tricky and you might need to try different light conditions before the app is able to separate the soil aggregates from the (white) background.

The app records the results in a folder under <span class='file'>documents</span> on your phone. The path to the folder of your results depend on whether you used moulder or slakes:

```
./documents/slakes/SoilHealthTestResultsExport_YYYY-MM-DD_HH-MM_SS
```

Each experiment generate a set of csv files with the results and also the images used for the analysis:
```
|____ImageData.csv
|____Sample_hdd_YYYY-MM-DD_hh-mm_ss
| |____Initial_mask.jpg
| |____Final_crop.jpg
| |____Initial_bw.jpg
| |____Second_crop.jpg
| |____Second_bw.jpg
| |____Description.csv
| |____Initial_crop.jpg
| |____Final_mask.jpg
| |____Initial_original.jpg
| |____Second_mask.jpg
| |____Final_original.jpg
| |____Second_original.jpg
| |____Final_bw.jpg
|____SampleSummary.csv
```

<figure class="third">
<img src="../../images/Initial_original.jpg">
<img src="../../images/Second_original.jpg">
<img src="../../images/Final_original.jpg">

<img src="../../images/Initial_crop.jpg">
<img src="../../images/Second_crop.jpg">
<img src="../../images/Final_crop.jpg">

<img src="../../images/Initial_bw.jpg">
<img src="../../images/Second_bw.jpg">
<img src="../../images/Final_bw.jpg">

<img src="../../images/Initial_mask.jpg">
<img src="../../images/Second_mask.jpg">
<img src="../../images/Final_mask.jpg">

<figcaption> Images captured and processed by the app <span class='app'>slakes</span>; top row: original images, 2nd row: cropped images, 3rd row: black and white cropped images, and last row: masked aggregates. left column: initial sample in dry conditions, middle column: soaked sample , last column: soaked sample at the end of the experiment</figcaption>
</figure>

<span class='apss'>slakes</span> generates 3 csv documents:

- SampleSummary.csv
- ImageData.csv
- Sample_hdd_YYYY-MM-DD_hh-mm_ss/Descriptions.csv

The only data that is transferred to the database is the Aggregate Stability Index (ASI) from the file <span class='file'>SampleSummary.csv</span>:

```
Aggregate Stability Index Values

Sample Name,Time Stamp,Aggregate Stability Index,Error State
sample-name,03/22/2024 09:55,0.48,None
```

or in more readable tabular form:

| Sample Name | Time stamp | Aggregate Stability Index | Error State |
| "sample-name" | 03/22/2024 09:55 | 0.48 | None |

### DBML

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs
// Tool: https://dbdiagram.io/d

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'AI4SH schema for moulder measurements'
}

Table users.user {
  userid SERIAL
}

Table samples.sample_event {
  sampleid SERIAL
  siteid INTEGER [pk]
}

Table moulder {
  sampleid INTEGER [pk]
  topsoil Boolean [pk]
  userid INTEGER
  app_version_moulder BOOLEAN
  aggregate_stability_index float
  description TEXT
}
// topsoil (true) represents 0-20 cm, subsoil (false) represents 20-50 cm.

REF: samples.sample_event.sampleid - moulder.sampleid
REF: users.user.userid - moulder.userid
```

### Figure

<figure>
<a href="../../images/DBML_schema-moulder.png">
<img src="../../images/DBML_schema-moulder.png"></a>
<figcaption>Moulder DBML database structure</figcaption>
</figure>
