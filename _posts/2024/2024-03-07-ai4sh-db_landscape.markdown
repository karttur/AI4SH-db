---
layout: post
title: "Schema: landscape"
categories: ai4sh-db
excerpt: "AI4SH schema landscape contains tables for recording the landscape and the sample point topographic setting. The registered properties are considered permanent over the AI4SH project duration."
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-03-07 11:27'
modified: '2024-07-13'
comments: true
share: true

---

## Outline

The **landscape** schema contains the tables that describe the more constant above ground characteristics around each sample point. The landscape description in the database should represent an area of approximately 25 m<sup>2</sup> centred around the central pit of each sample point.

The tables _major_landform_ and _hillslope_position_ are catalogues of accepted different landscape elements and positions. Thus the definition of both major landform and hillslope position can only be done by selecting one item from these predefined catalogues.

## Idea and objective

Permanent landscape characteristics around each sample-points are regarded as static over the duration of the AI4SH project. The data in the **landscape** schema is thus not associated with any particular sampling event.

### DBML

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'AI4SH schema for landscape'
}

Table users.user {
  userid SERIAL
}

Table sites.samplepoint {
  pointid SERIAL
}

Table landscape { // constant
  pointid INTEGER [pk]
  observation_date DATE
  userid INTEGER
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

Ref: "users"."user"."userid" - "public"."landscape"."userid"

Ref: "public"."landscape"."major_landform_code" - "public"."major_landform"."major_landform_code"

Ref: "public"."landscape"."hillslope_position_code" - "public"."hillslope_position"."hillslope_position_code"

Ref: "sites"."samplepoint"."pointid" - "public"."landscape"."pointid"
```

### Figure

<figure>
<a href="../../images/DBML_schema-landcapse.png">
<img src="../../images/DBML_schema-landscape.png"></a>
<figcaption>Landscape DBML database structure.</figcaption>
</figure>
