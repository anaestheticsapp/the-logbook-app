---
title: Logbook File Format
date: 2020-10-19
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
* Example applications that can open ```.logbook``` files:
  * The Logbook App [Preview Video](https://www.youtube.com/watch?v=iSJ5rMXmSbk)

# Object structure
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

If one of the intermediate nodes is missing (ie "general_anaesthesia" or "category"), JavaScript would throw an "Uncaught TypeError" error when trying to access the induction property. To write reliable code, it is good practice to add conditional checks for each property:

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

## Minimal data set
* Mandatory data fields
* Contains data required for the ARCP in the UK and must be present in every .logbook file
* Properties are required unless stated otherwise

```js
/**
 * @type {object}
 * @property {object} timing - date and session
 * @property {object} event - details for anaesthetic, criticalcare, clinic, pain, ... cases
 * @property {object} patient - age, age_units, asa
 * @property {object} training - supervision, supervisor, teaching
 * @property {array} procedures - list of procedures
 * @property {array} incidents - list of incidents (all strings)
 * @property {string} notes - general notes
 * @property {object} metadata - logbook origin, version, unique case id, timestamps (start and inserted)
 */
```

### Timing
* Most modern logbooks store ```sessions``` rather than ```start``` and ```end``` times
* ```Start``` and ```end``` times from older logbooks can be converted to ```session``` and ```duration```
* Medberry only uses three sessions ("Day", "Evening", "Night") whilst most other logbooks use four sessions ("Morning", "Afternoon", "Evening", "Night")

```js
/**
 * @type {object}
 * @name timing
 * @property {string} date - ISO 8601 format "YYYY-MM-DD"
 * @property {string} session - "Morning", "Afternoon", "Evening", "Night"
 * @property {number} duration - in minutes (optional)
 */
```

### Event
* This requires more thought

```js
/**
 * @type {object}
 * @name event
 * @property {string} activity - theatre | procedure | clinic | admission | daily review | ward review | cardiac arrest | trauma team | ward round | intra-hospital transfer | inter-hospital transfer | discussion with relatives | end of life care/donation
 * @property {string} category - anaesthesia | icm | pain | aim | em | phem
 * @property {object} details - see below
 */
```
#### Event > Details
```js
/**
 * @variation details(1) - category: anaesthesia and activity: theatre
 * @property {string} name - custom names are possible
 * @property {string} speciality - speciality of case
 * @property {string} priority - "Elective", "Urgent", "Expedited", "Immediate" - NCEPOD Classification
 * @property {string} destination - "Day Case", "Ward", "POCU", "Critical Care"
 * @property {string} secondary_name - (optional)
 * @property {string} secondary_speciality - (optional)
 * @property {string} tertiary_name - (optional)
 * @property {string} tertiary_speciality - (optional)
 *
 * @variation details(2) - activity: clinic
 * @property {string} name - C-PEX, Pre-op, Chronic Pain
 *
 * @variation details(3) - category: icm
 * @property {string} name - diagnosis such as "Pneumonia", "Diabetic ketoacidosis"
 * @property {string} speciality - (optional)
 * @property {string} referral - ie. "Hypotension" (optional)
 * @property {array} support - see below (optional)
 */
```

### Patient
* A lot of logbooks use age (+/- age units). These should be included in the patient object to prevent any loss of data and to allow re-importing logbook cases into the respective logbook.

```js
/**
 * @type {object}
 * @name patient
 * @property {number} age - no decimals allowed
 * @property {string} age_units - only the following options are allowed: "Days", "Months", "Years"
 * @property {number} age_category - range 1-5
 * @property {number} asa - range 1-6 (6 being "Donor") [ASA Classification](https://www.asahq.org/standards-and-guidelines/asa-physical-status-classification-system)
 */
```

### Training
* The RCoA LLP and AnaestheticsApp Logbook use teaching and supervision when logging theatre cases

```js
/**
 * @type {object}
 * @name training
 * @property {string} supervision - only the following options are allowed: "Immediate", "Local", "Direct", "Remote", "Solo" [NOTES TO PROVIDE CLARIFICATION OF ACSA STANDARDS](https://www.rcoa.ac.uk/sites/default/files/documents/2019-08/ACSA-SelfAssessment-2019.pdf)
 * @property {string} supervisor - ie "Consultant"; custom options are allowed
 * @property {string} supervisor_id - optional
 * @property {string} supervisor_name - optional
 * @property {string} teaching - ie "Medical Student"; custom options are allowed
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

### Metadata
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
 * @property {string} id - case id that must be unique
 * @property {string} name - logbook website, without "https://", ie "anaesthetics.app" or "lifelong.rcoa.ac.uk"
 * @property {number} version - version of logbook data schema, increment when object schema is modified
 * @property {object} timestamp - timestamps in milliseconds
 * @property {number} timestamp.start - timestamp when case was started
 * @property {number} timestamp.inserted - timestamp when case was inserted
 * @property {number} timestamp.edited - timestamp when case was edited (optional)
 * @property {object} setting
 * @property {string} setting.country - ISO 3166-1 alpha-2 country codes, such as "gb", "de", "at", "ug" (optional)
 * @property {string} setting.rotation - ie "North West" (optional)
 * @property {string} setting.location - ie "Manchester Royal Infirmary" (optional)
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
 * @property {object} location - country, region, hospital for a case
 */

/**
 * @type {object}
 * @name support - nested in event (category = icm)
 * @property {array} Respiratory - options "HFNO", "NIV", "IPPV", "ECMO"
 * @property {array} Cardiovascular - options "Intra-aortic balloon pump", "Pacing", "Ventricular assist device"
 * @property {array} Neuro - options "EEG", "ICP monitoring"
 * @property {array} Renal - empty array, currently no further options available
 * @property {array} Liver - empty array, currently no further options available
 */
```
