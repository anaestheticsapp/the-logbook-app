---
title: Logbook File Format - Draft 2
date: 2020-10-18
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
* Preview: [TheLogbook App](https://www.youtube.com/watch?v=iSJ5rMXmSbk)
* JSON schema: will follow once details are final

# Object structure

## Minimal data set
* Mandatory data fields
* Contains data required for the ARCP in the UK and must be present in every .logbook file
* Properties are required unless stated otherwise

```js
/**
 * @type {object}
 * @name encounter
 * @property {object} metadata - logbook origin, version, unique case id, timestamps (start, inserted)
 * @property {object} timing - date and session
 * @property {object} patient - age, age_units, asa
 * @property {array} events -
 * @property {array} procedures -
 * @property {array} incidents -
 * @property {object} training - supervision, supervisor, teaching
 * @property {string} notes - general notes
 */
```

### Metadata
```js
/**
 * @type {object}
 * @name metadata
 * @property {string} id - case id that must be unique, the AnaestheticsApp Logbook uses uuid v4
 * @property {string} logbook - website of the logbook, without "https://", ie "anaesthetics.app" or "lifelong.rcoa.ac.uk"
 * @property {number} version - version of logbook data schema, increment when object schema is modified
 * @property {object} setting - see below
 * @property {object} timestamp - see below
 */
```

#### Metadata > Setting
```js
/**
 * @type {object}
 * @name setting
 * @property {string} country - using ISO 3166-1 alpha-2 country codes, such as "gb", "de", "at", "ug"
 * @property {string} rotation - ie "North West"
 * @property {string} location - ie "Manchester Royal Infirmary"
 */
```

#### Metadata > Timestamp
```js
/**
 * @type {object}
 * @name timestamp
 * @property {number} start - timestamp in milliseconds when case was started (use session times below if the logbook does not use exact times)
 * @property {number} inserter - timestamp in milliseconds when case was inserted
 * @property {number} edited - timestamp in milliseconds when case was edited
 */
```

#### Corresponding ```session``` times for ```start```
 * Morning - 08:00:00
 * Afternoon - 13:00:00
 * Evening - 18:00:00
 * Night - 22:00:00

### Timing
```js
/**
 * @type {object}
 * @name timing
 * @property {string} date - ISO 8601 format "YYYY-MM-DD"
 * @property {string} session - "Morning", "Afternoon", "Evening", "Night"
 * @property {number} duration - in minutes (optional)
 */
```

### Patient
```js
/**
 * @type {object}
 * @name patient
 * @property {number} age - no decimals allowed
 * @property {string} age_units - only the following options are allowed: "Days", "Months", "Years"
 * @property {string} age_category -
 * @property {number} asa - range 1-6, 6 being "Donor"
 */
```

### Events
```js
/**
 * @type {object}
 * @name events
 * @property {string} category - anaesthesia | icm | pain | ...
 * @property {string} activity - theatre | clinic | ward round | transfer | admission | discussion with relatives | end of life care/donation
 * @property {object} details - see below
 */
```
#### Event > Details
```js
/**
 * @variation details(1) - anaesthesia
 * @property {string} priority - "Elective", "Urgent", "Expedited", "Immediate"
 * @property {string} destination - "Day Case" if applicable, other values are optional, non-standard and include "Ward", "POCU", "Critical Care"
 * @property {string} primary_speciality - speciality of case
 * @property {string} primary_operation - custom names are possible
 * @property {string} secondary_speciality - (optional)
 * @property {string} secondary_operation - (optional)
 * @property {string} tertiary_speciality - (optional)
 * @property {string} tertiary_operation - (optional)
 *
 * @variation details(2) - icm
 * @property {string} speciality - (optional)
 * @property {string} diagnosis - allow custom diagnosis
 * @property {string} referral - ie. "Hypotension" (optional)
 * @property {array} support - see below (optional)
 */
```

### Procedures
```js
/**
 * @type {object}
 * @property {string} category - airway, lines, regional, general
 * @property {string} name - name of regional, procedure, airway, mode of anaesthesia
 * @property {array} technique - "Landmark", "Ultrasound", "Nerve Stimulator", "Catheter"
 * @property {array} supervision - "Immediate", "Local", "Distant", "Solo"
 */
```

### Training
```js
/**
 * @type {object}
 * @name training
 * @property {string} supervision - only the following options are allowed: "Immediate", "Local", "Direct", "Remote", "Solo"/"None"
 * @property {string} supervisor_id -
 * @property {string} supervisor_grade - ie "Consultant"; custom options are allowed
 * @property {string} teaching - ie "Medical Student"; custom options are allowed
 */
```