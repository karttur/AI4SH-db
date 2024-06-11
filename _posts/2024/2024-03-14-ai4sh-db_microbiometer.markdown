---
layout: post
title: "Schema: microbiometer"
categories: ai4sh-db
excerpt: "AI4SH schema microbiometer"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-03-14 11:27'
modified: '2024-03-20'
comments: true
share: true

---

## Outline

The schema _microbiometer_ only contains a single table, for recording the results of the [Microbiometer<sup>TM</sup>](https://microbiometer.com) soil sampling kit.

The soil sampling, including extraction method, depth, classification, photos etc are recorded in the schema _samples_. The landscape (above ground) characteristics and the cultivation methods and its history are covered in the _sites_ schema.

## Microbiometer

The [Microbiometer<sup>TM</sup>](https://microbiometer.com) is a commercial soil sampling kit for estimating the soil microbial biomass and fungal to bacterial ratio.
The measurements takes about 20 minutes and requires a smartphone, and the Microbiometer soil test kit.

The procedure for estimating bacteria and fungi includes mixing a small soil sample with a reagent and then use a template colour chart for evaluation using an app that interpreters the colour change of your soil sample through the camera of a smartphone. The full procedure takes about 20 minutes. If you subscribe and register an account you can save the results in an online database. For used with AI4SH the results need to be accessible for use with other in-situ data, and the AI4SH in-situ database thus contains a schema for storing microbiometer data. Storing the microbiometer data with its own app includes sample, sample point and sample site data, which are part of other schemas in the AI4SH in-situ database.

MicroBiometerTM  expresses test results as a number between 0 and 100.

### DBML

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'AI4SH schema for microbiometer measurements'
}

Table users.user {
  userid UUID
}

Table samples.sample_event {
  sampleuuid UUID
  siteid UUID [pk]
}

Table microbiometer {
  sampleuuid UUID [pk]
  sampler UUID
  bacteria_ratio float
  fungi_ratio float
  extra_reagent_required BOOLEAN [DEFAULT false]
  description TEXT
}

REF: samples.sample_event.sampleuuid - microbiometer.sampleuuid
REF: users.user.userid - microbiometer.sampler

```

### Figure

<figure>
<a href="../../images/DBML_schema-microbiometer.png">
<img src="../../images/DBML_schema-microbiometer.png"></a>
<figcaption>microbiometer DBML database structure (sketch)</figcaption>
</figure>
