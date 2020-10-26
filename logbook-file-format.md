---
title: Logbook File Format
date: 2020-10-25
summary: Proposed data structure for the .logbook file format
meta: File extension and format for .logbook files
tags: logbook
img: ''
---

# Standard for Logbook Files
* File extension: ```.logbook```
* File format: JSON
* Mime type: ```text/json```
* Use case: anaesthesia logbooks
* Applications that can open ```.logbook``` files:
  * The Logbook App [Preview Video](https://www.youtube.com/watch?v=iSJ5rMXmSbk)

# JSON Style Guidelines

## Deep vs Flat Objects
There is no right or wrong answer, but one of the problems when looking for a property that's deep in a tree-like structure is that one often has to check whether intermediate nodes exist. Using the proposed ```event``` object as an example:

```js
const obj = {
  id: 0,
  event_type: 'procedure',
  event: {
    procedure: {
      category: {
        general_anaesthesia: {
          induction: 'Intravenous',
          maintenance: 'TIVA/TCI'
        }
      }
    }
  }
};
```

To access the ```induction``` property, one would have to do the following:

```js
// this is what we want to do
const induction = obj.event.procedure.category.general_anaesthesia.induction;
```

If one of the intermediate nodes is missing (ie "general_anaesthesia" or "category"), JavaScript would throw an ```Uncaught TypeError``` when trying to access the induction property. To write reliable code, it is good practice to add conditional checks for each property:

```js
// please don't make me do this
const induction = obj.event
  && obj.event.procedure
  && obj.event.procedure.category
  && obj.event.procedure.category.general_anaesthesia ? obj.event.procedure.category.general_anaesthesia.induction : null;
```

This adds a lot of complexity to the code, especially when writing a function to generate summary reports for different procedures. There is an easy solution to this, which is using ```optional chaining```. Unfortunately, this is a fairly new feature and only supported by the most recent modern browsers (Chromium 80, iOS 13.7, Firefox 74). This limits it's use case in web apps, especially on iOS as Apple only allows the use of the Safari browser engine.

```js
// this is nice but not widely supported
const induction = obj.event?.procedure?.category?.general_anaesthesia?.induction;
```

## Property names
* Avoid the use of reserved JavaScript keywords (relevant here are "case", "private", "public").
* The first character must be a letter or an underscore (_).
* Subsequent letters can include any letter, number, or underscore.
* Variable names are case sensitive.
* Words should be joined and capitalised (ie ageUnits) rather than separated with an underscore (age_units).
* Array types should have plural property names, other fields are singular.
* Naming conflicts are avoided by incrementing the ```metadata.version``` value.

## Property values
* Must be unicode booleans, numbers, strings, objects, arrays, or null.
* Properties should be removed if a property value is empty or null.
  * Top-level object properties should not be dropped.
  * Does not apply to properties such as ```age``` or ```version```.

# Minimal Data Set
* Mandatory data fields
* Contains data required for the ARCP in the UK and must be present in every .logbook file
* Properties are required unless stated otherwise

```js
/**
 * @type {object}
 * @property {object} activity - data for theatre, icm, clinic, pain, ...
 * @property {object} event - procedures, regional, anaesthesia, incidents
 * @property {object} metadata - data required to interpret and handle the logbook case
 * @property {string} note - general notes
 * @property {object} patient - age, asa
 * @property {object} setting - country, rotation, location (hospital)
 * @property {object} timing - date and session
 * @property {object} training - supervision, supervisor, teaching
 */
```

## Activity

```js
/**
 * @type {object}
 * @name activity
 *
 * @variation activity(1)
 * @property {string} context - theatre
 * @property {string} destination - "Day Case", "Ward", "POCU", "Critical Care" (optional - except when day case)
 * @property {string} priority - NCEPOD Classification "Elective", "Urgent", "Expedited", "Immediate"
 *
 * @variation activity(2)
 * @property {string} context - icm
 * @property {string} diagnosis - ie "Pneumonia"
 * @property {string} referral - ie. "Hypotension" (optional)
 * @property {array} support - see below (optional)
 * @property {string} type - admission | daily review | ward review | cardiac arrest | trauma team | ward round | intra-hospital transfer | inter-hospital transfer | discussion with relatives | end of life care/donation
 *
 * @variation activity(3)
 * @property {string} context - clinic
 * @property {string} type - C-PEX, Pre-op, Chronic Pain
 *
 *
 * @variation activity(4)
 * @property {string} context - phem | pain | procedure | session/event
 */
```

## Event
```js
/**
 * @type {object}
 * @name event
 * @property {object} anaesthesia -
 * @property {array} incidents -
 * @property {array} procedures -
 * @property {array} regional -
 * @property {array} support - optional for icm
 * @property {array} surgeries -
 */
```

### Anaesthesia
```js
/**
 * @type {object}
 * @name anaesthesia
 * @property {array} airway - "LMA", "ETT", "DLT"
 * @property {string} name - "GA", "Sedation"
 * @property {array} options - "Volatile", "TIVA", "RSI"
 */
```

### Procedure / Regional
```js
/**
 * @type {object}
 * @property {string} name - name of regional or procedure
 * @property {array} options - "Landmark", "Ultrasound", "Nerve Stimulator", "Catheter", "Observed"
 * @property {string} category - airway, access, drains, other, custom, axial, lower limb, upper limb (optional)
 * @property {string} outcome - "Failed" (optional)
 * @property {string} notes - "Failed" (optional)
 * @property {string} supervision - "Immediate", "Local", "Distant", "Solo" (optional)
 */
```

### Incidents
```js
/**
 * @type {object}
 * @property {string} name - name of incident
 */
```

### Surgeries
```js
/**
 * @type {object}
 * @property {string} name -
 * @property {string} speciality -
 */
```

### Support
```js
/**
 * @type {object}
 * @name support
 * @property {array} Respiratory - "HFNO", "NIV", "IPPV", "ECMO"
 * @property {array} Cardiovascular - "Intra-aortic balloon pump", "Pacing", "Ventricular assist device"
 * @property {array} Neuro - "EEG", "ICP monitoring"
 * @property {array} Renal - empty array, currently no further options available
 * @property {array} Liver - empty array, currently no further options available
 */
```

## Metadata
* Data stored here is required to interpret and handle the logbook case - the user should not be allowed to directly edit any properties
* The ```start``` timestamp is made up of the date and session time
  * Morning - 08:00:00
  * Afternoon - 13:00:00
  * Evening - 18:00:00
  * Night - 22:00:00
* Whilst the ISO 8601 format is used for the ```date``` property above, timestamps should be stored as Unix Timestamps in milliseconds. Using timestamps makes it easier to work with dates in JavaScript without being dependent on large third party libraries like ```moment.js```.
* The ID **must be collision free**, this is important when importing third-party logbooks. The AnaestheticsApp Logbook uses uuid v4. NanoID is faster and would be a good alternative. Which method is being used is up to the developer.

```js
/**
 * @type {object}
 * @name metadata
 * @property {string} env - development | stable (optional)
 * @property {string} id - case id that must be unique
 * @property {string} name - logbook website, without "https://", ie "anaesthetics.app" or "lifelong.rcoa.ac.uk"
 * @property {object} timestamp - timestamps in milliseconds
 * @property {number} timestamp.start - timestamp when case was started
 * @property {number} timestamp.inserted - timestamp when case was inserted
 * @property {number} timestamp.edited - timestamp when case was edited (optional)
 * @property {number} version - represents version of logbook data schema, increment when object schema is modified. No decimals as every change is a breaking change
 */
```

## Patient
* A lot of logbooks use age (+/- age units). These should be included in the patient object to prevent any loss of data and to allow re-importing logbook cases into the respective logbook.

```js
/**
 * @type {object}
 * @name patient
 * @property {number} age - no decimals allowed
 * @property {string} ageUnits - only the following options are allowed: "Days", "Months", "Years"
 * @property {number} ageCategory - range 1-5 (optional)
 * @property {number} asa - range 1-6 (6 being "Donor") [ASA Classification](https://www.asahq.org/standards-and-guidelines/asa-physical-status-classification-system)
 * @property {string} sex - "Male", "Female"
 */
```

## Setting
```js
/**
 * @type {object}
 * @name setting
 * @property {string} country - ISO 3166-1 alpha-2 country codes, such as "gb", "de", "at", "ug" (optional)
 * @property {string} location - ie "Manchester Royal Infirmary" (optional)
 * @property {string} rotation - ie "North West" (optional)
 */
```

## Timing
* Most modern logbooks store ```sessions``` rather than ```start``` and ```end``` times
* ```Start``` and ```end``` times from older logbooks can be converted to ```session``` and ```duration```
* Medberry only uses three sessions ("Day", "Evening", "Night") whilst most other logbooks use four sessions ("Morning", "Afternoon", "Evening", "Night")

```js
/**
 * @type {object}
 * @name timings
 * @property {string} date - ISO 8601 format "YYYY-MM-DD"
 * @property {string} session - "Morning", "Afternoon", "Evening", "Night"
 * @property {number} duration - in minutes (optional)
 */
```

## Training
* The RCoA LLP and AnaestheticsApp Logbook use teaching and supervision when logging theatre cases
* Levels of supervision are based on [NOTES TO PROVIDE CLARIFICATION OF ACSA STANDARDS](https://www.rcoa.ac.uk/sites/default/files/documents/2019-08/ACSA-SelfAssessment-2019.pdf)

```js
/**
 * @type {object}
 * @name training
 * @property {string} supervision - "Immediate", "Local", "Direct", "Remote", "Solo"
 * @property {string} supervisor - ie "Consultant"; custom options are allowed
 * @property {string} supervisorName - optional
 * @property {string} teaching - ie "Medical Student"; custom options are allowed
 */
```

## Optional properties being used by third-party logbooks
### AnaestheticsApp Logbook
```js
/**
 * further properties found in the main object
 * @type {object}
 * @property {array} tags - array of strings with options "reflect", "draft", "followup"
 * @property {array} labels - array of strings with custom options
 */
```
