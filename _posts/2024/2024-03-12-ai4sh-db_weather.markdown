---
layout: post
title: "Schema: weather"
categories: ai4sh-db
excerpt: "AI4SH schema for weather conditions just before and during sample events."
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-03-12 11:27'
modified: '2024-07-13'
comments: true
share: true

---

## Outline

The schema **weather** is required for capturing the conditions just before and during sampling, primarily affecting the upper soil layer moisture and temperature conditions and thus the biological activity and the presence of different macrofauna taxa. Also the sky and ground conditions at the time of soil sampling is recorded in the schema.

Both _sky_conditions_ and _ground_condtions_ are recorded via catalogs of predefined conditions.

## Idea and objective

The weather and related ground conditions can vary strongly over both time and space. Observations require on-site presence. The attributes are fairly easy to observe and register, with the filling of sky and ground conditions depending on predefined catalogues and lists to chose from.

### DBML

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs
// Tool: https://dbdiagram.io/d

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'AI4SH schema for weather'
}

Table users.user {
  userid SERIAL
}

Table samplepoint {
  pointid INTEGER
}

Table weather {
  pointid INTEGER [pk]
  datetime timestamp [pk]
  userid INTEGER
  sky_condition_code char(2)
  rain_mm_1h SMALLINT
  rain_mm_24h SMALLINT
  temperature_1h_mean SMALLINT
  temperature_24_max SMALLINT
  temperature_24_min SMALLINT
  ground_condition_code CHAR(2)
}

Table sky_condtion {
    sky_condition_code char(2) [pk]
    sky_condition VARCHAR(8)
  }
  // sky_condtion alternatives: "clear", "cloudy", "changing", "rainy"

Table ground_condtion {
    ground_condition_code char(2) [pk]
    ground_condition VARCHAR(8)
  }
  // ground_condtion alternatives: "dry", "wet", "dew", "flooded", "snow", "ice", "hail"

Ref: "public"."samplepoint"."pointid" < "public"."samplepoint_weather"."pointid"

Ref: "users"."user"."userid" - "public"."samplepoint_weather"."userid"

Ref: "public"."sky_condtion"."sky_condition_code" - "public"."samplepoint_weather"."sky_condition_code"

Ref: "public"."ground_condtion"."ground_condition_code" - "public"."samplepoint_weather"."ground_condition_code"
```

### Figure

<figure>
<a href="../../images/DBML_schema-weather.png">
<img src="../../images/DBML_schema-weather.png"></a>
<figcaption>Weather DBML database structure.</figcaption>
</figure>
