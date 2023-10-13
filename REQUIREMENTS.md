# Requirements

A webapp capable of handling PDF uploads of newspapers issues and doing whatever is required to:

* Extract text from the PDF using OCR technology
* Index OCR extracted text using a search backend
* Generate zoomable images for every page
* Provide a global search capable of querying the search backend and linking to the related issue
* Display a single issue in a web PDF viewer with integrated search and highlighting

In order to achieve this we are looking for the most lightweight and straightforward solution.

## Open ONI

Open ONI is the most complex but feature complete solution. It's an all in one piece of software that gives us little freedom, but satisfy all our requirements.

### Creation of a Batch

For every issue a `Batch` needs to be created. A `Batch` is packed with [Bagit](https://en.wikipedia.org/wiki/BagIt) and for every page issue needs:

* a PDF file
* a `.jp2` image
* a METS-ALTO XML file containing metadata about the PDF file (text and coordinates)
* other indexing XML files
* a MARC XML record

Even if a lot of documentation about the METS-ALTO standards is available ([METS](https://www.loc.gov/standards/mets/), [ALTO](https://www.loc.gov/standards/alto/)), it's currently unclear how to programmatically generate MATS-ALTO XML files starting from a PDF file.
Looks like a custom software needs to be developed for the purpose, or a subscription for on premise software is required ([Veridian](https://veridiansoftware.com/services/#dataconversion), [BSLW](https://bslw.com/metadata/#nonmarc)). The only possible open-source software probably capable of exporting METS-ALTO XML files I was able to find is [Goobi](https://github.com/intranda/goobi-workflow), which is still untested.

Same goes for fabricating a MARC record, since we (probably) don't want to obtain one from the Library of Congress.

### Loading and displaying of a Batch

Supposing we are able to create batches the rest of the implementation should be straightforward.
Open-ONI provides a Django command to load batches in the database and index it's OCR in Solr. It will also extract coordinates for every word in a page and output a JSON file with a mapping that is exposed throught an Apache server at a specific location (`lccn/<issuue-id>/<date>/<edition>/<sequence>/coordinates/` [Example](https://oregonnews.uoregon.edu/lccn/sn96088442/1903-04-11/ed-1/seq-3/coordinates/)).
Those coordinates are later used by the frontend on search in order to draw highlights using the OpenSeadragon API.

## Custom Django app using a PDF viewer

Due to the difficulties and complexity involved in generating batches different approaches were evaluated to make a PDF OCR text indexable, searchable and browsable with highlighting.

This involves the development of a webapp (or a fork on Open-ONI) which allows to:

* upload a PDF file with OCR included
* extract the OCR text. This is possible using tools like [Tesseract](https://github.com/tesseract-ocr/tesseract) or [Open-ONI OCR extractor](https://github.com/open-oni/open-oni/blob/dev/core/ocr_extractor.py).
* index the extracted text in a search backend (like [Solr](https://github.com/open-oni/open-oni/blob/dev/core/solr_index.py) or Elasticsearch)
* build interfaces to perform searches across all indexed document
* build an interface to display the matched document in a PDF viewer supporting search and hightlighting

### Evaluated PDF viewers

#### PDF.js

This looked like a good Javascript solution, capable of running search and highlighting on a PDF document. An example [can be found here](https://mozilla.github.io/pdf.js/web/viewer.html).

This will avoid the need to adopt the METS-ALTO standard, as well as complex solutions to extract coordinates and draw highlights.
I still have to figure out how to make a JS build for our example PDF using this tool.

### Browser PDF plugin

The built-in PDF browser plugin is already capable of searching through a PDF document. Unfortunately search and highlight functionalities are still (limited on Chromium)[https://bugs.chromium.org/p/chromium/issues/detail?id=792647].
An example of how this may work is included here.

### Adobe PDF embed API

We may consider using the [Adobe PDF embed API directly](https://developer.adobe.com/document-services/apis/pdf-embed/). Unfortunately this is a propietary solution, and will require and API key (even if it's free).
