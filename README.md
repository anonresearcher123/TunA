# TunA: Tunable Query Optimizer for Web APIs and User Preferences

## What is TunA?
TunA is a query optimizer that is able to combine RESTful Web APIs and RDF KBs in form of triple stores, tuning its (query) plans towards user preferences. Erroneous information from Web APIs is detected using a hierarchical agglomerative clustering. Our evaluation shows that TunA outperforms current state-of-the-art systems and is less vulnerable for erroneous information, even in settings where only unreliable sources are available

## How to run TunA?
In order to run TunA several configurations must be done first. First, clone the project and open it in an IDEA of your choice. Afterwards, three steps need to be done: (1) all local RDF databases that you want to query must be registered, (2) the function store (containing all alignments between the local used RDF databases and a set of APIs) must be configured and (3) you have to specify your queries and execute them.

All configurations of TunA are done in the config file `config.json`.

### Configure local databases
To add a database or change the location of a database, you must add or modify a JSON object of the following form in the databse array in the global configuration file `config.json`:

![database_config](https://user-images.githubusercontent.com/120786910/210595102-4378216a-69a7-4f48-b86b-e91dc1daa98c.JPG)

The key `label` indicates the name of the database with which it can later be selected in order to make queries to it. The path to the triple store (in this case a triple database created by Apache Jena) or index is stored under `index`. It is also possible to specify a URL leading to the SPARQL endpoint of an RDF knwoledge base. Lastly, `source` stores the path to the source file (i.e., an .nt, .ttl, or other file).

### Setting up a function store
The configuration file `config.json` can be used to define the storage location for one or multiple sets of function definitions (also denoted as function store). 

```"functionstore": "res/functionstores/"```

In the presented example above, the path to the function store is set to `res/functionstore/`. In the specified folder, several sub-folders can be created (see the example below) with different function definitions in order to model different function stores.

![function_defs_config](https://user-images.githubusercontent.com/120786910/210748228-89a0e146-d9d7-469b-9257-3f2a9a56ac8a.JPG)

In this example, three different function stores have been defined, namely `evalF1`, `evalF2` and `evalF3`. The function stores mentioned above contain a number of different function definitions, which map an API response to a knowledge base. In this case, a function definition is a JSON file, structured like the following example:

```
{
  "meta": {
    "database": "sample_dblp",
    "api": "a13",
    "inputType": "https://dblp.org/rdf/schema-2020-07-01#Publication",
    "inputRelation": "http://www.wikidata.org/prop/direct/P356",
    "responseTime": 504,
    "responseProbability": 0.32
  },
  "alignments": [
    {
      "reliability": 0.75,
      "relation_path": [
        {
          "path": [
            "https://dblp.org/rdf/schema-2020-07-01#title"
          ]
        }
      ],
      "api_path": [
        "feed.entry.title"
      ]
    }
  ]
}

```

<!--- ![functionstore_config](https://user-images.githubusercontent.com/120786910/210595508-1316e0ff-ca17-4140-a951-af99de29169e.JPG) --->

### Running queries

## Where to find the data sets used for the evaluation?
For the evaluation we used as data sources the ones provided by [ETARA](https://github.com/anonresearcher123/ETARA) (an RDF version of dblp and data exports of the RESTful Web APIs from CrossRef, SciGraph and Semantic Scholar). Additionally, we created three tainted data sets, that contained erroneous data (i.e. incorrect titles, etc.) and removed information, e.g., titles and author names, from the dblp dataset. 

[Download used data sets](https://shorturl.at/fgqV4 "Link to used datasets")
