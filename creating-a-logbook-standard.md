---
title: Creating a standard for logbook files
date: 2020-10-03
summary: Logbook Standard
meta: Creating a standard for logbook files
tags: logbook
img: ''
---
It was only a few weeks ago when I received an email asking for help. A Trac user was no longer able to open his/her logbook and the ".db" backup file with more than 300 cases was unusable as no other logbook would accept this file type. Trac has been unsupported for a while now and it was only a matter of time before the app would stop working. If you are using Trac, I strongly recommend you backup your cases now! There are two options available to you - the first one is to export cases as a .csv file and the second one is to create a .db file (SQLite database). The latter is only supported by Trac - if you can no longer open the app, you can no longer restore your backup. There are ways around this, but neither of them are straightforward. The "easiest" way would be to
* install a program that can open SQLite files,
* export the database tables,
* manually join the database tables together and,
* convert the file to a logbook format of your choice.

# Why is it so difficult to import cases from other logbooks?
Transfering data from one logbook to another requires a data migration tool. At the moment, every logbook uses a different file format and a different data structure. This means that developers have to create a unique file convertor for each logbook. This is not only time consuming but also requires a lot of background knowledge about how other logbooks store data.

The three file types that are currently being used are .csv, .xml and .xlsx files.

CSV files (**Comma Seperated Values**) are popular because they are
* text-based and easy to read, and
* have the smallest file size of all formats

Values are seperated by commas as shown below. The best way to visualise the data is by using tables. Imagine that each line represents a row and values enclosed by commas represent columns.

```
20/03/2019,General,Laparotomy,M,GA ETT,Spinal,Arterial line,Urgent
20/03/2019,General,Appendicectomy,F,GA ETT,,,Urgent
23/03/2019,ENT,Thyroidectomy,M,GA LMA,,,Elective
```

Ideally, logbooks should provide a header to describe the type of data found in each column. This is specified in the first row and looks something like this:

```
Date,Speciality,Operation,Sex,Anaesthesia,Regional,Procedure,Priority
20/03/2019,General,Laparotomy,M,GA ETT,Spinal,Arterial line,Urgent
```

Unfortunately, most older logbooks don't provide this. Figuring out which column contains what data can be tricky. Developers have to manually analyse each logbook and create a custom-made header which is stored in the data migration tool. The user will have to provide the name of the logbook which created the backup file, otherwise, the file convertor won't know which custom-made header to use. We also have to bear in mind that the logbook creating the .csv file is different to the logbook parsing the .csv file. If no header is provided, updating the data structure of the former might break the latter.

Over the past few years, I noticed a few problems with importing .csv files.
* Some users chose the incorrect logbook name when uploading their backup. The file convertor used the wrong custom-made header and invalid data was imported. This seemed to be a common problem when users had multiple files from different logbooks.
* Some logbook files were corrupt because a comma was deleted or added by mistake. This can happen when manually editing files and makes the .csv file completely unusable.
* Users edited their logbook in Excel and re-arranged the order of columns, which also makes it impossible to parse the file.

**"eXtensible Markup Language"** XML files are also text-based but much more reliable as they represent structured information. This particular file format can be selected when exporting cases from the old RCoA FileMaker logbook. A disadvantage is the file size of .xml files which rapidly increases when storing thousands of cases. On mobile devices, this can have an impact on data costs and load time if the file is being converted on the internet.

The problem with file size can be overcome by using file compression. **Excel documents**, for example, are just a collection of compressed XML files. You can see what's inside an Excel file by renaming it to .zip and extracting it with a file archiver. Using the .xml file format allows Excel to use formulas, graphs and worksheets. The RCoA Lifelong Learning Platform LLP uses this file format when exporting logbook cases. Whilst it is a good file format to use, it's major limitation is the difficulty of reading the file on the server or client. It usually requires two libaries, one to unzip the file and one to parse the xml content. Integrating those into a data migration tool is possible, but not always straightforward.

Another problem with importing logbook cases is the variation in data structure of different logbooks.
* Dates, for example, can be stored as dd/mm/yyyy, yyyy-mm-dd or timestamps. Timestamps can also be stored in seconds or milliseconds.
* Logbooks that use the CSV and XML file formats store procedures in columns, whilst the LLP logbook stores each procedure in a new row.
* Different logbooks use different names for specialities (ie "Neuro" vs "Neurosurgery"), priority (ie "Routine" vs "Elective"), procedures ("Supraclavicular" vs "Supraclavicular Block" vs "SCB"), operations ("Appendix" vs "Appendicectomy"), and so on.

# How can we solve this mess?
Having a standardised logbook format makes it easier for other logbooks to import data. Losing logbook cases could be a thing of the past and switching between different logbooks would be a much smoother process. It's important to have a choice of different logbooks as different users have different use-cases:
* Users who switch mobile platforms may want to use a logbook designed for that platform. The AnaestheticsApp Logbook, for example, is a Progressive Web App that works best on Android, Chrome OS and Windows, but lacks features on iOS because Safari doesn't support them. iOS users may want to choose a native logbook app like Medberry.
* Trainees who become consultants may want to switch to a logbook that hides options that are no longer required, such as supervision and supervisor
* Anaesthetists may spent a year abroad and want to use a logbook that is designed for that particular country.
* Regional anaesthetists may want to use a logbook that allows more detailed logging of blocks

# What is the best file format to use?
Logbook files
* should be text-based to ensure readability and to avoid the requirements of third-party libraries (excludes Excel documents)
* should be able to handle complex data (excludes CSV files)
* should have a compact file size to allow fast data transmission (excludes XML files)

As all commonly used file formats have their limitations, I'm proposing to use a new file format which has been around for a long time. The JavaScript Object Notation (JSON) file format is the "industry standard" in web applications and has several notable features:
* It doesn't require any third-party libraries or dependencies. Many modern programming languages, including JavaScript, PHP, Python, C#, C++ and Java, already support generating and parsing JSON-format data
* Data can be easily validated before being imported
* JSON is scalable and future-proof
* File size is small
* No performance loss when working with large data
* Easy to implement for logbook developers

Just like opening a Word or Excel file, double clicking on a logbook file should open the user's preferred logbook app and display the file content (logbook cases). JSON files have the .json file extension but this is already being used by many applications. Assigning a logbook app to the .json extension would likely interfere with many programs. Instead, I suggest to use the .logbook file extension.

# Creating a standardised data structure
All logbooks should contain a minimal data set which contains the same information required for the Annual Review of Competences ARCP in the UK. This ensures that critical information required for the ARCP is never lost when switching logbooks. Logbook apps should be allowed to add optional data fields to set themselves apart from other apps and to allow the input of extra information for certain users (ie for regional or obstetric anaesthetists). However, users should be made aware that optional data fields may not be supported by other logbooks. Thanks to the JSON file format, other logbooks can easily add support for optional data fields created by another logbook (if they wish to do so).

When using the JSON file format, logbook cases are stored in arrays and each array element contains an object, which holds the information about the logbook case. The basic structure of the object is shown below. All object properties are mandatory and this should be validated before importing data.

```js
/**
 * @type {object}
 * @property {object} encounter
 * @property {string} encounter.date - ISO 8601 format "YYYY-MM-DD"
 * @property {string} encounter.session - only the following options are allowed: "Morning", "Afternoon", "Evening", "Night"
 *
 * @property {object} anaesthetic | criticalcare | clinic | pain - type of logbook case, only one is allowed
 *
 * @property {object} patient
 * @property {number} patient.age - no decimals allowed
 * @property {string} patient.age_units - only the following options are allowed: "Days", "Months", "Years"
 * @property {number} patient.asa - range 1-6, 6 being "Donor"
 *
 * @property {object} training
 * @property {string} training.supervision - only the following options are allowed: "Immediate", "Local", "Direct", "Solo"
 * @property {string} training.supervisor - ie "Consultant"; custom options are allowed
 * @property {string} training.teaching - ie "Medical Student"; custom options are allowed
 *
 * @property {array} regional - array of objects (see below)
 * @property {array} procedures - array of objects (see below)
 * @property {array} incidents - array of strings
 * @property {array} notes - array of objects (see below)
 *
 * @property {object} metadata
 * @property {string} metadata.id - case id that must be unique, the AnaestheticsApp Logbook uses uuid v4
 * @property {string} metadata.name - website of the logbook, without "https://", ie "anaesthetics.app" or "lifelong.rcoa.ac.uk"
 * @property {number} metadata.version - version of logbook data schema, increment if a property has been added or removed
 *
 * @property {number} dt_start - timestamp (milliseconds), date and time case was started ie "2020-09-15 08:00:00" if "Morning" session was selected
 * @property {number} dt_inserted - timestamp (milliseconds), date and time case was inserted
 */
```

Corresponding ```session``` times for ```dt_start```
```js
/**
 * Morning - 08:00:00
 * Afternoon - 13:00:00
 * Evening - 18:00:00
 * Night - 22:00:00
 */
```

The ```regional``` and ```procedures``` properties contain an array of objects:
```js
/**
 * @type {object}
 * @property {string} name - name of regional or procedure
 * @property {array} technique - "Landmark", "Ultrasound", "Nerve Stimulator", "Catheter"
 * @property {array} supervision - "Immediate", "Local", "Distant", "Solo"
 */
```

The ```notes``` property contains an array of objects:
```js
/**
 * @type {object}
 * @property {string} note
 * @property {string} heading - heading to describe the type of note. Optional but recommended if more than one note is present (ie. regional, incident)
 */
```

To define the type of logbook case, one of the following properties must be present
* ```anaesthetic``` or
* ```criticalcare```
* Logbooks may add further optional categories, such as ```clinic```, ```pain```, ```procedure```, ```session```. Other logbooks may not support optional categories.

This allows logbooks to filter cases based on the type of logbook case. In Javascript, this would look something like this:

```js
const logbook = [
 {..., anaesthetic: {...} }, // case 1
 {..., criticalcare: {...} }, // case 2
 {..., anaesthetic: {...} }, // case 3
]
const theatreLogbook = logbook.filter(obj => obj.anaesthetic);
const criticalCareLogbook = logbook.filter(obj => obj.criticalcare);
```

The structure of the ```anaesthetic``` and ```criticalcare``` objects are shown below:
```js
/**
 * @type {object}
 * @name surgery
 * @property {string} speciality - required
 * @property {string} operation - required; allow custom operations
 * @property {string} priority - required; **mandatory** options "Elective", "Urgent", "Expedited", "Immediate"
 * @property {string} destination - required; should include "Day Case" if present
 * @property {string} anaesthesia - required; for example "GA ETT RSI TIVA"
 * @property {string} secondary_speciality - optional
 * @property {string} secondary_operation - optional
 * @property {string} tertiary_speciality - optional
 * @property {string} tertiary_operation - optional
 */

/**
 * @type {object}
 * @name critical care
 * @property {string} diagnosis - required; allow custom diagnosis
 * @property {string} event - required; for example "Admission"
 * @property {string} referral - optional; for example "Hypotension"
 * @property {string} speciality - optional;
 */
```

Logbooks may add optional properties. All optional properties should be documented by the logbook so that other developers can support and import those properties in their logbook if they wish to do so. The AnaestheticsApp Logbook uses the following properties:

```js
/**
 * @property {number} duration - in minutes
 * @property {array} tags - array of strings with options "reflect", "draft", "followup"
 * @property {array} labels - array of strings with custom options
 * @property {string} country - using ISO 3166-1 alpha-2 country codes, such as "gb", "de", "at", "ug"
 * @property {string} region - ie "North West"
 * @property {string} hospital - ie "Manchester Royal Infirmary"
 *
 * @property {object} criticalcare.support - optional; organ support (see below)
 */

/**
 * @type {object}
 * @name support
 * @property {array} Respiratory - array of strings with options "HFNO", "NIV", "IPPV", "ECMO"
 * @property {array} Cardiovascular - array of strings with options "Intra-aortic balloon pump", "Pacing", "Ventricular assist device"
 * @property {array} Neuro - array of strings with options "EEG", "ICP monitoring"
 * @property {array} Renal - empty array, currently no further options available
 * @property {array} Liver - empty array, currently no further options available
 */
```

An example of how a logbook in JSON format would look like is shown below:

```json
[
  {
    "encounter": { "date": "2018-08-01", "session": "Morning" },
    "anaesthetic": {
      "speciality": "Plastics",
      "operation": "Other",
      "priority": "Elective",
      "destination": "Day Case",
      "anaesthesia": "GA LMA"
    },
    "patient": { "age": 12, "age_units": "Years", "asa": 1 },
    "training": {
      "supervision": "Immediate",
      "supervisor": "Consultant",
      "teaching": null
    },
    "regional": [],
    "procedures": [],
    "incidents": [],
    "notes": [{ "heading": null, "note": "Notes" }],
    "metadata": {
      "id": "anaesthetic-1",
      "name": "lifelong.rcoa.ac.uk",
      "version": "b1"
    },
    "dt_insert": 1601298522643,
    "dt_start": 1533153600000
  },
  {
    "encounter": { "date": "2018-08-01", "session": "Afternoon" },
    "anaesthetic": {
      "speciality": "ENT",
      "operation": "Adeno- tonsillectomy",
      "priority": "Immediate",
      "destination": "Day Case",
      "anaesthesia": "GA ETT"
    },
    "patient": { "age": 54, "age_units": "Years", "asa": 2 },
    "training": { "supervision": "Solo", "supervisor": null, "teaching": null },
    "regional": [
      { "name": "Fascia Iliaca", "technique": ["Landmark"] },
      { "name": "Spinal", "technique": ["Landmark"] }
    ],
    "procedures": [
      { "name": "Arterial cannulation" },
      { "name": "central venous access–internal jugular" }
    ],
    "incidents": [],
    "notes": [],
    "metadata": {
      "id": "anaesthetic-2",
      "name": "lifelong.rcoa.ac.uk",
      "version": "b1"
    },
    "dt_insert": 1601298522643,
    "dt_start": 1533153600000
  },
]
```

# Remaining questions
* There will be variation for values such as ```operation``` and ```regional``` and ```procedures```. Ideally there should be a common data set of operations and procedures but I think it's important that users should have the option to provide custom text for certain data fields. A common data set can be created on GitHub as a JSON file format. Data sets can be reviewed every 6-12 months. Logbooks can choose not to include certain data sets.
* Should all notes be stored in the ```notes``` property as described above or should the ```notes``` property contain a string of general notes and notes for incidents and procedures are listed in the corresponding objects in ```regional```, ```procedures``` and ```incidents```.

# Conclusion
These are my initial thoughts on how to improve and standardise logbooks. The exact data structure of objects is not final and subject to change. I have created a [GitHub repository](https://github.com/anaestheticsapp/logbook-standard) which you can use to post questions, comments and suggestions. I'm sure that not all logbook parties will be interested in this but at least it's a start and future developers have something to go by when developing a new logbook.

Currently, it's not possible to import your logbook to the RCoA Lifelong Learning Platform. The college stated that "We cannot incorporate your old logbook with the new Logbook on LLP. This is because the LLP logbook has fields that are not the same as other logbooks". By creating a standardised file format and structure, the RCoA will *hopefully* consider to add an import functionally for third-party logbooks. Parsing a JSON file and transforming the data to a format the Lifelong Learning Platform uses should pose no problems as this is a format that every developer is familiar with.
