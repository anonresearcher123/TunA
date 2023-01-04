# TunA: Tunable Query Optimizer for Web APIs and User Preferences

## What is TunA?
TunA is a query optimizer that is able to combine RESTful Web APIs and RDF KBs in form of triple stores, tuning its (query) plans towards user preferences. Erroneous information from Web APIs is detected using a hierarchical agglomerative clustering. Our evaluation shows that TunA outperforms current state-of-the-art systems and is less vulnerable for erroneous information, even in settings where only unreliable sources are available

## How to run TunA?
In order to run TunA several configurations must be done first. First, clone the project and open it in an IDEA of your choice. Afterwards, three steps need to be done: (1) all local RDF databases that you want to query must be registered, (2) the function store (containing all alignments between the local used RDF databases and a set of APIs) must be configured and (3) you have to specify your queries and execute them.

### Configure local databases
### Setting up a function store
### Running queries

## Where to find the data sets used for the evaluation?
For the evaluation we used as data sources the ones provided by [ETARA](https://github.com/anonresearcher123/ETARA) (an RDF version of dblp and data exports of the RESTful Web APIs from CrossRef, SciGraph and Semantic Scholar). Additionally, we created three tainted data sets, that contained erroneous data (i.e. incorrect titles, etc.) and removed information, e.g., titles and author names, from the dblp dataset. 

[Download used data sets](https://shorturl.at/fgqV4 "Link to used datasets")
