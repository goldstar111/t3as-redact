# t3as-redact

## Introduction

Redaction of private or otherwise sensitive information from confidential documents is often required before documents can be released
in Freedom of Information responses or as Open Data.
Adobe Portable Document Format (PDF) is often used as an electronically transferable record that is regarded as being 'non-editable' and suitable for archiving.  

This project:
- performs automatic Named Entity Recognition and co-reference detection on text extracted from a PDF document;
- allows the user to correct any errors in the automatic processing of entities of interest;
- allows the user to select entities and larger blocks of text for redaction, providing a reason for redaction;
- produces a redacted PDF with the selected text (and meta-data) removed and the redacted region highlighted with the reason for redaction.

The implementation consists of:
- RESTful web services; and
- an HTML5 user interface.

### Named Entity Recognition

The redaction application uses automatic Named Entity Recognition to highlight names of people, organisations, locations, dates/times/durations and numbers; which are likely targets for redaction.
The available named entity recognisers, ranked from 'smartest' and slowest first, are:
- CoreNLP with Corefs;
- CoreNLP;
- OpenNLP; and
- NICTA NER.

The one that works best for you depends on the content type of the document and your specific needs and could be any of these.
The Named Entity Recognition could be tailored for specific applications.

## Install Tools

To run the code install:

- a JRE e.g. from openjdk-7 (prefered for Scala 2.11) or openjdk-8 (will be required for Scala 2.12);
- a servlet v3.0 (or later) container e.g. [tomcat](http://tomcat.apache.org/) 7; and
- the build tool [sbt](http://www.scala-sbt.org/).

To develop [Scala](http://scala-lang.org/) web services install:

- the above items (or the full JDK instead of just the JRE, but the JRE should be sufficient);
- [Eclipse IDE](https://www.eclipse.org/downloads/); and
- the Scala plugin for Eclipse [scala-ide](http://scala-ide.org/download/current.html).

## Build
 
Automatic builds are provided at: <https://social-watch.dev.etd.nicta.com.au/>.

The command:

	sbt clean test package dumpLicenseReport

from the project's top level directory:

- cleans out previous build products,
- runs all tests,
- creates a war file (project code with all 3rd party dependencies) and
- generates a license report.

## Run

To run the webapp from sbt (the web app will be accessible at: <http://localhost:8080/>):

    sbt
    > jetty:start
    
and to then stop it:

	> jetty:stop
	
... or install `target/scala-2.11/t3as-redact_{scala-version}-{project-version}.war` as redact.war in the container of your choice.

## User Interface

A publicly accessible instance is available at: <http://redact.t3as.org/>.

### Browser compatability
The user interface uses HTML5 features and won't work on all browsers. It has been successfully tested on latest versions of:
- chrome on linux
- chrome on win 8.1
- firefox on win 8.1
- safari on OS-X

Unfortunately it is not correctly handling PDF on IE11/win 8.1, but it could be modified to run on this and other recent versions of IE.

### User Guide
There is a trade-off between visual clues to functionality and user interface clutter: at the moment the demonstration errs on the side of low clutter with hidden functionality (ok for experienced users, but new users will need instructions).

1. Optionally change Named Entity Recognition settings under `File > Settings`. The settings are used when the PDF is first opened, so to change the settings you have to start again from the beginning. Settings reset to default values when the page is (re)loaded.
1. Drag a PDF file onto the front page to open it.
1. Click `Redactions` to see the extracted text and named entities in the text pane (right side):
    - edit a named entity mention by selecting text that overlaps a highlighted mention - the selected text will be the new text of the named entity mention;
    - create a new named entity mention by selecting text that does not overlap any highlighted mention.
1. The named entity pane (left side) shows the text of named entity representative mentions and the number of co-references (other text referring to the same real-world entity). Try clicking and drag-and-drop in this pane and hopefully you can figure out the functionality offered more easily that way than by reading the following:
    - click on a representative mention to show any co-references and to highlight in the text pane the representative mention and it's co-references (with a weaker highlight);
    - click on a co-reference to switch the stronger highlight to that co-reference mention in the text pane;
    - drag a named entity (the representative mention and its co-references) to a new entity type to change its type (e.g. change all instances of "Victoria" from a Location to a Person);
    - drag a named entity a) to another named entity b) to convert the representative mention and co-references of a) to be new co-references of b) (e.g. because they are actually referring to the same real-world entity);
    - drag a co-reference to an entity type to convert it to a new named entity of that type (e.g. change just one instance of "Victoria" from a Location to a Person);
    - drag a co-reference a) to another named entity b) to convert a) to be a new co-reference of b) (e.g. because it is actually referring to the same real-world entity).
1. Once the named entities to redact are correct (no need to fix the others), click the checkbox of those to be redacted and enter a reason for redaction. The text pane shows text to be redacted with strike-through.
1. Click `Export` (at the top of the page) to display the redacted PDF. Click `Download` to download the redacted PDF.

To work on a new PDF file, either:
- reload the page then drag a new PDF file (settings will revert to defaults on reload); or
- `File > Close` then drag a new PDF file; or
- `File > Open`.

## Web Services

The [Swagger](http://swagger.io/) UI available at http://redact.t3as.org/swagger-ui:
- documents the web services; and
- provides a user interface allowing each web service to be called without programming.

Details from the following section are also pertinent to the web services.


## Implementation

The data structure for the result of named entity recognition is:
    
    case class Mention(start: Int, end: Int, text: String)
    case class NamedEntity(representative: Mention, ner: String, coRefs: List[Mention])
    case class Result(namedEntities: List[NamedEntity])
so each named entity has a `representative` mention, a type `ner` which can be "PERSON", "LOCATION" etc. and a list of other mentions `coRefs` (which is empty unless the selected NLP implimentation is `Stanford CoreNLP NER + Coref `).

If `Heuristic post-processing` is selected then code in the UI modifies this data, deleting some NamedEntity items and replacing some with coRefs.

The data structure for the redaction request is:

    case class RedactItem(page: Int, start: Int, end: Int, reason: String)
    case class Redact(redact: List[RedactItem])
Note that an item to be redacted is specified by page based character offsets into the extracted text. Page number is 1 based (PDF convention), char offsets are 0 based.


See also: [t3as-pdf](https://github.com/NICTA/t3as-pdf) which provides the PDF functionality used by this project.

## To do

- support separate markup and review processes prior to actual redaction (maybe related to next item)
- interoperate with Adobe tools by using standard metadata for redactions 


## Legal

This software is released under the terms of the [AGPL](http://www.gnu.org/licenses/agpl-3.0.en.html).
Source code for all transitive dependencies is available at [t3as-legal](https://github.com/NICTA/t3as-legal).
