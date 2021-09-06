# Creating Ontologies for the SMP

An ontology is a file describing the classes, properties and relationships that make up the data models visualised by the Shared Meaning Platform.

A class is a type of object -- a `Person`, an `Organisation`, an `IT System`, etc.

A property is an attribute an object has -- the class `Person` might have the properties `name`, `country` and `birthdate`.

Properties do not have to be string values, but can point to other objects in the data model via their IRI. For example, a specific instance of a `Person` with the `name` property of `Alice` might also have a `worksFor` property that points to an IRI for an object of class `Organisation` whose `name` is `Visual Meaning`. This is how links and relationships are made in the data model.

It is important to note that an ontology does *not* contain any actual data. An ontology would not contain any information about the person called `Alice` or the organisation called `Visual Meaning`, just the `Person` and `Organisation` classes that they belong to.

VM uses ontology files in the Turtle (.ttl) format.

## Converting ontology files that have been exported from Grafo

Grafo is a nice visual interface for creating ontologies without having to worry about manually hacking around with .ttl file formatting. You create your classes and relationships in the GUI, and then when you're finished you can export that ontology as a .ttl file. In theory Grafo should do all of the work for you. Unfortunately the reality is disappointing and there might still be a bit of manual hacking required before you'll have something that's ready for SMP ingest.

Note that you may not have to do all of the below steps depending on how well you've configured the ontology in Grafo prior to export. If it already looks fine, skip to the next step.

In the worst case, Grafo will export something that looks like this:

```turtle
@prefix : <http://www.gra.fo/schema/untitled-ekg#> .
@prefix dc: <http://purl.org/dc/elements/1.1/> .
@prefix gf: <http://www.gra.fo/schema/untitled-ekg#> .
...
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .
@base <http://www.gra.fo/schema/untitled-ekg#> .

<http://www.gra.fo/schema/untitled-ekg> rdf:type owl:Ontology .

#################################################################
#    Object Properties
#################################################################

###  http://www.gra.fo/schema/untitled-ekg/attendedBy
<http://www.gra.fo/schema/untitled-ekg/attendedBy> rdf:type owl:ObjectProperty ;
                                                   rdfs:domain <http://www.gra.fo/schema/untitled-ekg/GovernanceBoard> ;
                                                   rdfs:range <http://www.gra.fo/schema/untitled-ekg/Organisation> ;
                                                   rdfs:isDefinedBy <http://www.gra.fo/schema/untitled-ekg> ;
                                                   rdfs:label "attended by" .

...


#################################################################
#    Classes
#################################################################

###  http://www.gra.fo/schema/untitled-ekg/Category
<http://www.gra.fo/schema/untitled-ekg/Category> rdf:type owl:Class ;
                                                 rdfs:isDefinedBy <http://www.gra.fo/schema/untitled-ekg> ;
                                                 rdfs:label "Category" .

...


#################################################################
#    Annotations
#################################################################

<http://www.gra.fo/schema/untitled-ekg> rdfs:comment "" ;
                                        rdfs:label "Untitled Document" .
```

Note that it has been exported with the base URL of `http://www.gra.fo/schema/untitled-ekg`. As VM's base URL for IRIs is `http://visual-meaning.com/rdf` we'll need to change this; fortunately we can just do a find-replace operation in any text editor. **Replace all instances of the generic base URL with the VM base URL**.

For most of the document this should be fine, however there's a couple more tweaks that need to be made.

- The line `@prefix gf: <http://visual-meaning.com/rdf#> .` should read `@prefix vm: <http://visual-meaning.com/rdf> .` (note `gf` replaced by `vm` and the `#` character removed from the end).

- The line `<http://visual-meaning.com/rdf> rdf:type owl:Ontology .` should read `<http://visual-meaning.com/rdf/your-ontology-name-here> rdf:type owl:Ontology .`. Here we are defining the IRI for our ontology - come up with some unique, human-parsable text slug to go in place of `your-ontology-name-here`, for example `pppt-ecosystem`.

- All instances of `rdfs:isDefinedBy <http://visual-meaning.com/rdf> ;` should be replaced by `rdfs:isDefinedBy <http://visual-meaning.com/rdf/your-ontology-name-here> ;`

- At the end of the document in the `Annotations` block are two fields, `rdfs:comment` and `rdfs:label`. `comment` should be filled with a brief description of what the ontology is describing, while `label` is the name of the ontology.

## Inserting the correct indefinite article

When the SMP displays the data model in the left panel and you click on an object, you'll see an expanding dropdown with "is a Class" along with the object properties. By default the SMP will always use the indefinite article "a" when referring to a class, but if the class in question happens to start with a vowel, such as `Organisation`, we get the bad grammar of "is a Organisation".

To fix this, we can include which article to use in the ontology via the `skos.altLabel` property:

```turtle
vm:ItSystem a owl:Class ;
    rdfs:label "IT system" ;
    rdfs:isDefinedBy vm:pppt-ecosystem ;
    skos:altLabel "an IT system" ;
    .
```

And then it will display correctly in the SMP client. As an ontology may have many classes and which indefinite article to use isn't immediately obvious (is it a hotel or an hotel?) we have a script over in `map-tools` called `indefinite.py` that automatically adds the correct indefinite article and also converts the ontology into a nicer format for versioning.

    python indefinite.py ontology.ttl > new_ontology.ttl

**We should run this script prior to versioning any ontology.**

## Including the ontology in the data model

This side of things is handled in the `sheet-to-triples` repository. `sheet-to-triples` combines the labels export from Illustrator, data extracted from Excel workbooks via transforms and any other relevant data into a single set of map terms containing all data in the data model. A typical `sheet-to-triples` run command looks like this:

    python3 -m sheet_to_triples --model ~/backups/bare.json poleco_labels

To include the ontology in the data model, simply include the .ttl file as an `--add-graph` parameter:

    python3 -m sheet_to_triples --model ~/backups/bare.json poleco_labels --add-graph poleco_ontology.ttl

## What ontology attributes the SMP cares about

TBD

## How to write ontology files manually

TBD
