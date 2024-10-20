<h1> RDF Project </h1>

The Resource Description Framework (RDF) is a framework for representing information on the web.

This library implements the W3C Recommendation [RDF 1.1 XML Syntax](https://www.w3.org/TR/2014/REC-rdf-syntax-grammar-20140225/). RDF/XML files can be read as Graph, which is a simple collection of RDF Statements. Likewise, an RDF Graph can be written in RDF/XML. The library also implements reading and writing the simple [N-Triple](https://www.w3.org/TR/2014/REC-n-triples-20140225/) format, since this is used in the official test cases.

The code is in the bundle `{RDF Project}` in the public store. The licence is MIT.

## Motivation

PDFs can have metadata describing the document. In PDF 2.0 this is mandatory. The metadata is in the [XMP](https://www.adobe.com/products/xmp.html) (`Extensible Metadata Platform`) format defined by Adobe, which is now an ISO standard. XMP is also interesting, because many other digital formats (pictures, movies, etc.) embed their metadata as XMP.

The impuls to add the metadata feature to the PDF library came from a request from [Joachim Tuchel](http://www.objektfabrik.de/), who asked me if I could deal with [ZUGFeRD](https://www.ferd-net.de/zugferd/definition/index.html), a new format for electronic bills. A ZUGFeRD document is a PDF of an invoice with an attached XML document containing the same information in a structured form. For this, I would need to implement PDF attachments (should be simple) and XMP because ZUGFeRD requires certain entries in the metadata.

XMP is an RDF language, a subset of RDF/XML with restrictions (seemingly the way to define RDF languages). Many difficult features of RDF/XML are not needed for XMP, so it might have been easier to implement just that. But I got interested in RDF and wanted to do the real thing.

RDF represents facts in a very basic way: by asserting statements about something. A statement has a subject, a predicate and an object. A subject is usually an IRI (internationalized URL) pointing to some document on the web. A subject can also be an anonymous placeholder, a blank. Predicates always have to be an IRI. They specify the relation between the subject and the object and are usually defined by a vocabulary. Objects can be anything: IRI, Blank or Literal.

This simple representation is interesting, because it is more flexible and more expressive than relational databases or objects in object-oriented systems. RDF data does not need uniform structure as in the other approaches. Also, because predicates are IRIs, anyone can add their own semantics to existing objects without disturbing other users.

These features of RDF are inspiring and their use on the web is fascinating. [Wikidata](https://www.wikidata.org/wiki/Wikidata:Database_download) and [OpenStreetMap](https://wiki.openstreetmap.org/wiki/OSM_Semantic_Network), for example, are based on RDF. Now I dream of using RDF to describe the budget data for [Unsere Gelder](https://unsere-gelder.de/) or to use it for creating a PDF examples database.

Although I want to use RDF for XMP and the PDF library, RDF itself is valuable on its own. And since the RDF implementation does not have any dependencies on PDF, I publish it as stand-alone library which depends only on XML.

## Usage

The namespace is RDF. The core classes are

* **Graph**: a collection of **`Statement`s**
* **Statement**: consists of `subject`, `predicate` and `object`; each of which is a **`Term`**
* **Term**: with **`IRI`**, **`Blank`** and **`Literal`** as concrete subclasses

There are Parsers for XML and N-Triples. To get a **`Graph`** use:

```
(RDFXMLParser on: <ReadStream>) graph    "or"
(RDFXMLParser onString: <String>) graph  "or"
(RDFXMLParser onDom: <XMLDocument>) graph
```

If you have a String, you can simply say:

```
Graph fromXMLString: <String>
```

For N-Triples you do:

```
(NTriplesParser on: <ReadStream>) graph
```

With a String:

```
Graph fromNTriplesString: <String>
```

A **`Graph`** can be written out by creating an XML DOM which can be printed to a string:

```
<aGraph> asXMLDocument xmlPrintString
```

Writing a **`Graph`** as an N-Triples string:

```
<aGraph> nTripleString
```

## Experience

First, I read the nicely written [introductory chapter](https://www.w3.org/TR/2014/REC-rdf-syntax-grammar-20140225/#section-Syntax) of the RDF specification. For the examples, I programmed a prototyp ad-hoc to read and write them. That was fun, but many details were not mentioned, so that I used wild guesses for parts of the code.

Then I discovered all the [tests for RDF](https://www.w3.org/TR/rdf11-testcases/). Among them was the official test suite which covers every detail. How cool is that! After importing all tests as SUnit tests into the image, I refactored and refined my implementation with the use of the tests. In the beginning, many - about half - failed or lead to errors. This was fun too, but again, there has not been enough information for a “right” implementation. The tests covered all features but only isolated and didn't give any explanation about what the test is testing; just the expected outcome. Sometimes it was not clear how the features would interact with others. Anyhow, I fiddled with my implementation until all tests were green (about 350).

When an example or feature was not clear, I consulted the [formal grammar](https://www.w3.org/TR/2014/REC-rdf-syntax-grammar-20140225/#section-Infoset-Grammar). The grammar consists of syntax productions which clearly describe the form and interplay of all elements. The chapter is not for reading. But it is invaluable for deciding on the right way to program things. So I decided to implement the productions as shown in the spec. This was a lot of fun. I had the tests - all green - and all functionality had been implemented already. Therefore it was just refactoring of the code into a form close to the productions in the spec. This lead to less and cleaner code. And now I am pretty sure, that the implementation is complete. I think that it will handle any legal RDF/XML.

## Future directions

RDF with RDF/XML is one of the basic parts for the “semantic web”. There are many vocabularies designed to describe various domains. Therefore, several interesting possibilities arise.

### XMP

For the PDFtalk library I need XMP to work with the metadata of documents. XMP is an RDF language which restricts the very general RDF. Certain features are forbidden in XMP - luckily the complex ones: `XMLLiterals` and `xml:base`. Other restrictions apply to properties and how they are used. This defines the semantics of the language.

I wonder how to do this: defining a language on RDF. Interesting task.

### Triple store

Eventually, I want to store a lot of statements. For now, a `Graph` is just an `OrderedCollection` of `Statement`s. This will not be sufficient for large amounts of data. I will need a [triplestore](https://en.wikipedia.org/wiki/Triplestore).

#### Other syntaxes: Turtle and SPARQL

There are other syntaxes for RDF. Besides [HTML](https://www.w3.org/TR/2015/NOTE-rdfa-primer-20150317/) and [JSON](https://www.w3.org/TR/json-ld/) forms, [Turtle](https://www.w3.org/TR/turtle/) is a human readable form, often used for ontologies. Turtle itself has a few variants of which [SPARQL](https://www.w3.org/TR/sparql11-query/) is the most important. It resembles SQL and allows variables in any part of the triple. SPARQL is used as query language for triplestores, which usually offer a SPARQL endpoint where queries can be made. With a nice SPARQL writer, one can talk to any other triplestore. That would be cool! A parser would be nice for your own triplestore.

#### Schemas and inferencing

Schemas (or vocabularies or ontologies) for RDF can be easily (?) defined by anyone. A schema itself is defined in RDF. Commonly used (and sometimes mixed) are [RDFS](https://www.w3.org/TR/rdf-schema/) (RDF Schema) and [OWL](https://www.w3.org/2001/sw/wiki/OWL) (Web Ontology Language). Besides concepts like `Class` and `Property`, predicates and value restrictions, an ontology also defines properties for relationships. For example, you can define that the relation `married` between two persons is symmetrical; or that the `subclass` relationship is transitive.

With this, only basic facts have to be entered into a triple store. Many more facts can get deducted / inferenced. It would be an interesting project to build such an inferencer. I wonder if Gemstone would make a good triplestore.

#### Modeling

At some point, I will want to model my own fields of knowledge: budget data from all communities I can get hold of and an index for PDFs by the technical features they are using.

The challenge with inferencing systems is that they can get out of hand easily. Even small knowledge bases can become intractible.
