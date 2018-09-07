# ontology-mapping-chatbot

Ontology Mapping ChatBot is a  semi-interactive ontology mapping algorithm. It provides an easy-to-use interface (Yes or No questions) in order to semantize (i.e. to map ontologies on) OpenDataSoft datasets.

# Glossary

## DBPedia
DBpedia is wikipedia in RDF format with ontologies to describe resources.
Chatbot uses DBpedia to perform named entity recognition. In other word, to find class of entities (e.g., Italia is a country/PopulatedPLace.. or B.Obama is a President/Person..)

## Wikidata
Wikidata is an other dataset that is used for named entity recognition by the chatbot.

## LOV (Linked Open Vocabularies)
LOV is an ontology search engine. [This API](http://lov.okfn.org/dataset/lov/api) is used by the chatbot to find candidate ontologies for opendatasoft datasets.

## Ontology (The curse word)

An `ontology` is a vocabulary defining the concepts and relationships used to describe an area of concern.
It's composed of:
* `classes` (e.g. Car, Building, Person, Disease, Source Code) to represent a concept.
* `properties` (e.g. horsepower, financed By, date of birth, has Symptoms, Author) to represent relation between concepts.
* `rules` (e.g. A person have a unique date of birth).
Ontologies can be created for every area of concern and by everyone using RDF (Resource Description Framework), RDFS (RDF Schema) and OWL (Web Ontology Language).

## HDT
HDT (Header, Dictionary, Triples) is a compact data structure and binary serialization format for RDF

## RML

RML is a generic mapping language to describe multi-format to RDF transformations.

# Installation
Assuming you already have `python 2.7` and `pip 9`,

create a new virtualenv.

install dependencies with pip (~2 minutes to install hdt)

```bash
pip install pybind11==2.2.2
pip install -r requirements.txt
```

Download `hdt` versions of DBPedia and Wikidata datasets (splited in parts of 1Go) and override `/data_dumps`
```
https://eu.ftp.opendatasoft.com/bmoreau/dataset_dumps.zip
https://eu.ftp.opendatasoft.com/bmoreau/dataset_dumps.z01
https://eu.ftp.opendatasoft.com/bmoreau/dataset_dumps.z02
https://eu.ftp.opendatasoft.com/bmoreau/dataset_dumps.z03
https://eu.ftp.opendatasoft.com/bmoreau/dataset_dumps.z04
https://eu.ftp.opendatasoft.com/bmoreau/dataset_dumps.z05
https://eu.ftp.opendatasoft.com/bmoreau/dataset_dumps.z06
https://eu.ftp.opendatasoft.com/bmoreau/dataset_dumps.z07
https://eu.ftp.opendatasoft.com/bmoreau/dataset_dumps.z08
https://eu.ftp.opendatasoft.com/bmoreau/dataset_dumps.z09
https://eu.ftp.opendatasoft.com/bmoreau/dataset_dumps.z10
https://eu.ftp.opendatasoft.com/bmoreau/dataset_dumps.z11
https://eu.ftp.opendatasoft.com/bmoreau/dataset_dumps.z12
https://eu.ftp.opendatasoft.com/bmoreau/dataset_dumps.z13
https://eu.ftp.opendatasoft.com/bmoreau/dataset_dumps.z14
https://eu.ftp.opendatasoft.com/bmoreau/dataset_dumps.z15
https://eu.ftp.opendatasoft.com/bmoreau/dataset_dumps.z16
https://eu.ftp.opendatasoft.com/bmoreau/dataset_dumps.z17
```

# Run the demo
Navigate to ontology-mapping-chatbot folder and execute:

```bash
python manage.py runserver
```

App should be running on [http://127.0.0.1:8000/](http://127.0.0.1:8000/)

Go on [http://127.0.0.1:8000/chatbot/{dataset-id}](http://127.0.0.1:8000/chatbot/dataset-id) and replace `dataset-id` with the Data dataset id you want to semantize.

example: [http://127.0.0.1:8000/chatbot/roman-emperors@public/](http://127.0.0.1:8000/chatbot/roman-emperors@public/)

Semantization result will be stored in the `results` folder in a file named results/{dataset-id}.rml

# API

Chatbot is powered by an API exposed by this service:

## Correspondances API

Correspondences are the semantic correspondences between a dataset's field and an ontology. Fields can be linked to classes (Car, Person, ...) or poroperties of class (Engine horsepower, full name, ...).

`GET` `/api/{dataset-id}/correspondances/classes` to retrieve field values/class correspondences (powered by Dandelion API).

`GET` `/api/{dataset-id}/correspondances/properties` to retrieve field name/properties correspondences (powered by LOV API).

`GET` `/api/{dataset-id}/correspondances/` to retrieve both correspondences.

`POST` `/api/{dataset-id}/correspondances/mapping` to translate a set of correspondences into valid RML mapping file.

`POST` `/api/{dataset-id}/correspondances/confirmed` push confirmed correspondences after semantization (Machine Learning, logs, etc.).

`POST` `/api/{dataset-id}/correspondances/awaiting` push confirmed correspondences after semantization (Machine Learning, logs, etc.).

`POST` `/api/{dataset-id}/correspondances/denied` push denied correspondences after semantization (Machine Learning, logs, etc.).

## Conversation API

Conversations API is used to translate possible correspondences into Human Readable questions.

`POST` `/api/conversation/question/class` to retrieve a question about a class/field correspondence.

`POST` `/api/conversation/question/property` to retrieve a question about a property/field correspondence.

`POST` `/api/conversation/question/property-class` to retrieve a question about a class/property correspondence.

`GET` `/api/conversation/error/lov-unavailable` to retrieve phrase when LOV is unavailable.

`GET` `/api/conversation/error/no-classes` to retrieve phrase when no class is found.

`GET` `/api/conversation/greeting` to retrieve welcome phrase.

`GET` `/api/conversation/instructions` to retrieve instructions to use the chatbot.

`GET` `/api/conversation/answer/positive` to retrieve response to positive user input.

`GET` `/api/conversation/answer/neutral` to retrieve response to neutral user input.

`GET` `/api/conversation/answer/negative` to retrieve response to negative user input.

`GET` `/api/conversation/salutation` to retrieve phrase to say goodbye to the user.

## Named Entity Recognition API

`GET` `/api/ner?q=[query]&lang=[language]` returns the class of the term in the query.

# Clubhouse

[Link](https://app.clubhouse.io/opendatasoft/epic/11656) to Clubhouse story

# How it works?
## 1 Class matching
It use a local dumps of DBPedia and Wikidata to find classes corresponding to dataset's fields using named entity recognition.
N-first values of each fields are analysed and corresponding classes names are returned.

Each class name is sent to Class LOV API. Class LOV API returns class URI and class description. Those informations are stored in chatbot's candidate correspondances dict.

Using class description and field name, class/field associations are proposed to the user. A positive answer dispatch correspondance in chatbot's confirmed correspondance dict. A negative answer dispatch it in denied correspondance dict. Empty answer dispatch it in awaiting correspondance dict.

the following figure illustrate the class matching process.

![Class process](img/class_process.png "Class process")

## 2 Property matching
The goal of this step is to associate properties to already confirmed classes (e.g. full name of a Person?, full name of a car?, etc.)

Property LOV API is used to retrieve retrieve corresponding URI and description of properties.

Using property description, user is asked to confirm property/field association (e.g. date of birth property/ date_birth field). Then it is asked to link confirmed property to one of the confirmed class (e.g. date of birth linked to Person class).

the following figure illustrate the property matching process.

![Property process](img/property_process.png "Property process")
