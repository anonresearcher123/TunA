# TunA: Tunable Query Optimizer for Web APIs and User Preferences

## What is TunA?
TunA is a query optimizer that is able to combine RESTful Web APIs and RDF KBs in form of triple stores, tuning its (query) plans towards user preferences. Erroneous information from Web APIs is detected using a hierarchical agglomerative clustering. Our evaluation shows that TunA outperforms current state-of-the-art systems and is less vulnerable for erroneous information, even in settings where only unreliable sources are available

## How to run TunA?
In order to run TunA several configurations must be done first. First, clone the project and open it in an IDEA of your choice. Afterwards, three steps need to be done: (1) all local RDF databases that you want to query must be registered, (2) the function store (containing all alignments between the local used RDF databases and a set of APIs) must be configured and (3) you have to specify your queries and execute them.

All configurations of TunA are done in the config file `config.json`.

### Configure local databases
To add a database or change the location of a database, you must add or modify a JSON object of the following form in the databse array in the global configuration file `config.json`:
![database_config](https://user-images.githubusercontent.com/120786910/210595102-4378216a-69a7-4f48-b86b-e91dc1daa98c.JPG)

### Setting up a function store
![functionstore_config](https://user-images.githubusercontent.com/120786910/210595508-1316e0ff-ca17-4140-a951-af99de29169e.JPG)

### Running queries

## Where to find the data sets used for the evaluation?
For the evaluation we used as data sources the ones provided by [ETARA](https://github.com/anonresearcher123/ETARA) (an RDF version of dblp and data exports of the RESTful Web APIs from CrossRef, SciGraph and Semantic Scholar). Additionally, we created three tainted data sets, that contained erroneous data (i.e. incorrect titles, etc.) and removed information, e.g., titles and author names, from the dblp dataset. 

[Download used data sets](https://shorturl.at/fgqV4 "Link to used datasets")
