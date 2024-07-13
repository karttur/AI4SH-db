---
layout: post
title: "Schema: spectra"
categories: ai4sh-db
excerpt: "AI4SH schema spectra"
tags:
  - db
  - setup
  - schema
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2024-03-17 11:27'
modified: '2024-07-13'
comments: true
share: true
---

## Outline

The schema **spectra** contains tables for:  

- individual instruments (_spectrometer_),
- metadata for each spectral scan (_scanmeta_), and
- spectral reflectance mean and standard deviation for each scan (_reflectancescan_).

Spectrometers are individually calibrated precision instruments. The spectral sensor units of spectrometers of the same brand and model do not record exactly the same wavelengths. Thus each individual sensor applied must be registered together with the wavelengths (expressed in nano-metres) that it records. This information is residing in the table _spectrometer_.

A unique feature of the spectral sampling is that it can be done in the field, at home (kitchen lab) and in laboratories. The AI4SH sampling framework is built to accommodate all three types of spectral scans. In the table _scanmeta_ the field _prepcode_ can take one of the following three values:

- NO (no prep = in field scanning),
- MX (mixed = in kitchen scanning of wet mixed subsamples), or
- DS (dried & sieved = in laboratory scanning).

Further, spectral scanning is a rapid procedure and it is common to do repeated scans for a single sample, and each scan of the same sample is given a sequential number in the field _scanrepeat_. The in-field scanning (_prepcode_ = _NO_) can also be done at either a single pit (i.e. the central position) or including the perimeter pits in North, West, South and East. In the table _scanmeta_ this is encoded in the field _subsample_ using a single letter (M, C, N, E, S, W for Mixed, Central, North, East, South and West) The many options for spectral scanning leads to a metadata table with a large number of primary keys (pk):

- spectrometerid (the sensor)
- sampleid (the sample event)
- topsoil (topsoil or subsoil)
- subsample (Mixed, Central, North, East, South or West)
- scanrepeat (sequence from 1 to maximum 9)
- prepcode (NO [none], MX [mixed] or DS [dried & sieved])

Soil spectral analysis uses a technique called diffuse reflectance spectroscopy, where the reflecactance of the soil is compared to a standardised white reference. The value of the reflectance for each wavelength of the recorded spectra should thus be between 0 and 1 (at least reflecting some light and not ore than than a white reference surface). Sometime the absorbance (1/reflectacne) is registered instead of reflectance.

In the AI4SH database the reflectance (not absorbance) must be registered. If the spectral sensor applied records by default requires a white reference and directly produces reflectance that is the data to register. If the spectrometer, however, captures the white reference and the soil characteristcs separately (i.e. as digital numbers), the reflectance must be calculated before entered.

The actual spectral data, i.e. the soil diffuse reflectance expressed as a fraction between 0 and 1 for each wavelength, is saved in the table _reflectancescan_. The link between _scanmeta_ and _reflectancescan_ is a unique (SERIAL) id set for each individual scan (scanid). The link to the spectrometer is also via a unique SERIAL id (spectrometerid). As for other schemas, the user responsible for the scanning is linked to each scan via the userid.

The schema **spectra** is written for registering unique wavelengths with each instrument and the reflectance as compared to a white reference. Each reading in each scan should consequently have a value between 0 and 1. For each scan the number of erroneous values are compiled and registered in the table _scanmeta_:

- non recoded values (_nafreq_),
- negative values (_negfreq_),
- values extending 1 (_extfreq_).

### DBML

```
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Table users.user {
  userid SERIAL
}

Table samples.sample_event {
  sampleid SERIAL
}

Table spectrometer {
  brand TEXT [pk]
  model TEXT [pk]
  serialnumber TEXT [pk]
  wavelenghts REAL[]
  spectrometerid SERIAL
}

TABLE scanmeta {
  spectrometerid INTEGER [pk]
  sampleid INTEGER [pk]
  topsoil Boolean [pk]
  subsample char(1) [pk]
  scanrepeat char(1) [pk]
  prepcode CHAR(2) [pk]
  userid INTEGER
  scanid SERIAL
  nafreq REAL
  negfreq REAL
  extfreq REAL
}
// topsoil (true) represents 0-20 cm, subsoil (false) represents 20-50 cm.
// subsample is only for prepcode NO and can be C (central), N (north), E (east), S (south) of W (west). Default is M (mixed)
// scanrepeat is a simple numbering starting a 1 to a maximum of 9 repeats for the same sample

TABLE reflectancescan {
  scanid INTEGER
  signalmean REAL[]
  signalstd REAL[]  
}

TABLE sampleprep {
  prepcode CHAR(2) [pk]
  sampleprep text
  info TEXT
  url TEXT
}
// sampleprep alternatives: "NO", "MX", "DS"

REF: samples.sample_event.sampleid - scanmeta.sampleid

REF: users.user.userid - scanmeta.userid

REF: scanmeta.scanid - reflectancescan.scanid

Ref: "public"."spectrometer"."spectrometerid" < "public"."scanmeta"."spectrometerid"

Ref: "public"."sampleprep"."prepcode" < "public"."scanmeta"."prepcode"
```

### Figure

<figure>
<a href="../../images/DBML_schema-spectra.png">
<img src="../../images/DBML_schema-spectra.png"></a>
<figcaption>Spectra DBML database structure</figcaption>
</figure>
