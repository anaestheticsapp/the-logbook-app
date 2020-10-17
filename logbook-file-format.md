---
title: Logbook File Format
date: 2020-10-17
summary: Proposed data structure for the .logbook file format
meta: File extension and format for .logbook files
tags: logbook
img: ''
---

# Standard for logbook files
* File extension: ```.logbook```
* File format: JSON
* Mime type: ```text/json```
* Use case: anaesthesia logbooks

# Object structure

## Minimal data set
* Mandatory data fields
* Contains data required for the ARCP in the UK and must be present in every .logbook file
* Properties are requires unless stated

```js
/**
 * @type {object}
 * @property {object} encounter - date and session
 * @property {object} case - case details for anaesthetic, criticalcare, clinic, pain, ... cases
 * @property {object} patient - age, age_units, asa
 * @property {object} training - supervision, supervisor, teaching
 * @property {array} regional - list of regional procedures
 * @property {array} procedures - list of other procedures
 * @property {array} incidents - list of incidents (all strings)
 * @property {string} notes - general notes
 * @property {object} metadata - logbook origin, version, unique case id, timestamps (dt_start and dt_inserted)
 */
```

### Encounter
```js
/**
 * @type {object}
 * @name encounter
 * @property {string} date - ISO 8601 format "YYYY-MM-DD"
 * @property {string} session - "Morning", "Afternoon", "Evening", "Night"
 * @property {number} duration - in minutes (optional)
 */
```

### Case
```js
/**
 * @type {object}
 * @name case
 *
 * @variation case(1)
 * @property {string} type - anaesthetic
 * @property {string} speciality - speciality of case
 * @property {string} operation - custom names are possible
 * @property {string} priority - "Elective", "Urgent", "Expedited", "Immediate"
 * @property {string} destination - "Day Case" if applicable, other values are optional, non-standard and include "Ward", "POCU", "Critical Care"
 * @property {string} anaesthesia - mode of anaesthesia; ie. "GA ETT RSI TIVA"
 * @property {string} secondary_speciality - (optional)
 * @property {string} secondary_operation - (optional)
 * @property {string} tertiary_speciality - (optional)
 * @property {string} tertiary_operation - (optional)
 *
 * @variation case(2)
 * @property {string} type - criticalcare
 * @property {string} diagnosis - allow custom diagnosis
 * @property {string} event - for example "Admission"
 * @property {string} referral - ie. "Hypotension" (optional)
 * @property {string} speciality - (optional)
 * @property {array} support - see below (optional)
 *
 * @variation case(3) - logbooks have the option to use further types (these are optional, non-standard and may not be supported by other logbooks)
 * @property {string} type -  ie. clinic, pain, procedure, session, ...
 */
```

### Patient
```js
/**
 * @type {object}
 * @name patient
 * @property {number} age - no decimals allowed
 * @property {string} age_units - only the following options are allowed: "Days", "Months", "Years"
 * @property {number} asa - range 1-6, 6 being "Donor"
 */
```

### Training
```js
/**
 * @type {object}
 * @name training
 * @property {string} supervision - only the following options are allowed: "Immediate", "Local", "Direct", "Solo"
 * @property {string} supervisor - ie "Consultant"; custom options are allowed
 * @property {string} teaching - ie "Medical Student"; custom options are allowed
 */
```

### Regional and Procedures
```js
/**
 * @type {object}
 * @property {string} name - name of regional or procedure
 * @property {array} technique - "Landmark", "Ultrasound", "Nerve Stimulator", "Catheter"
 * @property {array} supervision - "Immediate", "Local", "Distant", "Solo"
 */
```

### Metadata
```js
/**
 * @type {object}
 * @name metadata
 * @property {string} id - case id that must be unique, the AnaestheticsApp Logbook uses uuid v4
 * @property {string} name - website of the logbook, without "https://", ie "anaesthetics.app" or "lifelong.rcoa.ac.uk"
 * @property {number} version - version of logbook data schema, increment when object schema is modified
 * @property {number} dt_start - timestamp in milliseconds when case was started (use session times below if the logbook does not use exact times)
 * @property {number} dt_insert - timestamp in milliseconds when case was inserted
 */
```

#### Corresponding ```session``` times for ```dt_start```
 * Morning - 08:00:00
 * Afternoon - 13:00:00
 * Evening - 18:00:00
 * Night - 22:00:00

## Further optional values being used by the AnaestheticsApp Logbook
```js
/**
 * further properties found in the main object
 * @type {object}
 * @property {array} tags - array of strings with options "reflect", "draft", "followup"
 * @property {array} labels - array of strings with custom options
 * @property {object} location - country, region, hospital for a case
 */

/**
 * @type {object}
 * @name support - nested in case (type = criticalcare)
 * @property {array} Respiratory - options "HFNO", "NIV", "IPPV", "ECMO"
 * @property {array} Cardiovascular - options "Intra-aortic balloon pump", "Pacing", "Ventricular assist device"
 * @property {array} Neuro - options "EEG", "ICP monitoring"
 * @property {array} Renal - empty array, currently no further options available
 * @property {array} Liver - empty array, currently no further options available
 */

/**
 * @type {object}
 * @name location
 * @property {string} country - using ISO 3166-1 alpha-2 country codes, such as "gb", "de", "at", "ug"
 * @property {string} region - ie "North West"
 * @property {string} hospital - ie "Manchester Royal Infirmary"
 */
```
