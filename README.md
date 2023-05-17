# TunA: Tunable Query Optimizer for Web APIs and User Preferences

## What is TunA?
To answer queries many query processors combine different SPARQL sources, e.g., various knowledge bases or APIs. RESTful Web APIs are rarely the focus of those systems as they come with many limitations, such as not being able to process SPARQL queries. Additionally, most existing approaches optimize their query plans only for performance, even though users often have additional preferences regarding coverage, reliability, or currency. In addition, data is often provided with different levels of quality so that not all sources should be trusted equally.

TunA is a query optimizer that is able to combine RESTful Web APIs and RDF KBs in form of triple stores, tuning its (query) plans towards user preferences. Erroneous information from Web APIs is detected using a hierarchical agglomerative clustering. Our evaluation shows that TunA outperforms current state-of-the-art systems and is less vulnerable for erroneous information, even in settings where only unreliable sources are available

## How to run TunA?
In order to run TunA several configurations must be done first. First, clone the project and open it in an IDEA of your choice. Afterwards, three steps need to be done: (1) all local RDF databases that you want to query must be registered, (2) the function store (containing all alignments between the local used RDF databases and a set of APIs) must be configured and (3) you have to specify your queries and execute them.

All configurations of TunA are done in the config file `config.json`.

### Configure local databases and APIs
To add a database or change the location of a database, you must add or modify a JSON object of the following form in the database array, named `databases`, in the global configuration file `config.json`:

```
{
  "label": "example_dataset",
  "index": "C:\\path\\to\\tdb",
  "source": "C:\\path\\to\\dataset.nt"
}
```

The key `label` indicates the name of the database with which it can later be selected in order to make queries to it. The path to the triple store (in this case a triple database created by Apache Jena) or index is stored under `index`. It is also possible to specify a URL leading to the SPARQL endpoint of an RDF knwoledge base. Lastly, `source` stores the path to the source file (i.e., an .nt, .ttl, or other file).

The procedure is similar for APIs, because these must also be defined in the configuration file `config.json`. For this purpose, an entry of the following form must be created in the section called `apis`:

```
{
  "name": "Name of the API",
  "label": "apiLabel",
  "timeout": 500,
  "format": "json",
  "parameters": [
    {
      "name": "q",
      "type": "https://dblp.org/rdf/schema-2020-07-01#Publication",
      "relation": "http://www.wikidata.org/prop/direct/P356"
    }
  ],
  "url": "http://www.example.com/work/title?doi={q}"
}
```

The first two fields describe the name or label of the API. Here, the label also serves as a key and can be uniquely identified. The field `timeout` describes the time that must elapse between two requests in order to be allowed to make another request to the API. The timeout was introduced to be able to model rate limits, as APIs are often limited in the number of requests per second. The fields `format` and `url` represent the response format of the API and the URL that must be used to make requests.

The field `parameters` is used to specify the parameters of a call URL, e.g, `doi={q}` is a call parameter that needs to be set to a value of a DOI in order to successfully request information. The field `name` specifies the name of the placeholder of the parameter, that will be replaced with the actual value, e.g., a DOI. The `type` and `relation` field specify the type of the entity that must provide the input value, denoted by the relation specified in the field `relation`.

### Setting up a function store
The configuration file `config.json` can be used to define the storage location for one or multiple sets of function definitions (also denoted as function store).

```
{
  "functionstore": "res/functionstores/",
  ...
}
```

In the presented example above, the path to the function store is set to `res/functionstore/`. In the specified folder, several sub-folders can be created (see the example below) with different function definitions in order to model different function stores.

![function_defs_config](https://user-images.githubusercontent.com/120786910/210748228-89a0e146-d9d7-469b-9257-3f2a9a56ac8a.JPG)

In this example, three different function stores have been defined, namely `evalF1`, `evalF2` and `evalF3`. The function stores mentioned above contain a number of different function definitions, which map an API response to a knowledge base. In this case, a function definition is a JSON file, structured like the following example:

```
{
  "meta": {
    "database": "databaseName",
    "api": "apiLabel",
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

The first section, named `meta`, contains various information about the API and the response, modelled sub-graphs of the local database. The first fields `database` and `api` describe which API is modelled and with which local database it is aligned. The fields `responseTime` and `responseProbability` describe the average response time of the API and how likely it is that information can be returned for a requested entity. The field `inputRelation` describes the relation that points to a value (in this case a DOI) that must be used to request an API. In this example, a DOI (marked with the predicate P356) must be sent to the API in order to receive a response with information (i.e. title).

The second section is named `alignments` and contains for each path in an API response (typically in JSON or XML) a mapping to the corresponding relation or predicate of the local database. If a path cannot be mapped because, for example, the information is not available locally, no mapping is specified. In the above example, the path `feed.entry.title`, which leads to the title of a publication, is mapped to the relation `dblp:title`. This way the query processor knows that the function definition can be used to find a missing title of a publication.

### Running queries
By executing the class `TunaConsole`, the query processor TunA is started. The user is then asked to specify the database to which he wants to send a query, to specify the function store to be used to fill in missing information in the query result and to specify his query. The user formulates the query in a file and saves it in the path `res/queries/`. The example below shows that the user submits a query `queryName.sparql` to the database `databaseName` with funciton store `functionStoreName`.

```
___________               _____   
\__    ___/_ __  ____    /  _  \  
  |    | |  |  \/    \  /  /_\  \ 
  |    | |  |  /   |  \/    |    \
  |____| |____/|___|  /\____|__  /
                    \/         \/ 

-----------------------------
Data Management
-----------------------------
Select Database: sample_dblp_gaps
Select Function Store: evalF1
Query Optimizer: tuna
Query File: query.sparql

-----------------------------
User Preferences
-----------------------------
Time Limit (ms): 500000000.0
Minimum Coverage: 1.0
Minimum Reliability: 0.9
```

Finally, before the query is executed, the user must specify their preferences (in terms of time limit, minimum required coverage, minimum required reliability). In the example above, the user has specified a maximum execution time of 5 seconds (measured in milliseconds). Furthermore, he requires a minimum coverage of 0.7 and a minimum reliability of 0.8.

## Where to find the data sets used for the evaluation?
For the evaluation we used the ones provided by [ETARA](https://github.com/anonresearcher123/ETARA) which consists of two domains, namely bibliographic data and filmographic data. The bibliographic set contains an RDF version of dblp and data exports of the RESTful Web APIs CrossRef, SciGraph and Semantic Scholar. The filmographic set contains the Linked Movie Database (LMDB) and exports of Open Movie Database (OMDB), The Movie Database (TMDB) and Internet Movie Database (IMDB). We used dblp and LMDB in our experiments as local KBs. To ensure both datasets contain missing information, we removed randomly chosen triples from both KBs, such as triples containing titles and author names. Additionally, we created six \emph{tainted} data sets that contain erroneous data, e.g., incorrect titles. Lastly, to evaluate TunA, we created a set of over 45 SPARQL queries. 

[Download used data sets](https://tinyurl.com/iswcdata "Link to used datasets")

## How to reproduce the ISWC experiments?
The following section explains in detail how each of our three experiments can be repeated. We created three experiments to evaluate the performance of TunA. The first experiment is used to compare TunA's computed execution plan with the optimum execution plan. The second experiment focuses on the tunability and scalability of TunA. The last experiment compares the performance of the state of the art system ANGIE with TunA.

First, the benchmark system [ETARA](https://github.com/anonresearcher123/ETARA) must be downloaded and started in an arbitrary IDE. Unfortunately, at the current time no executable JAR with a user friendly GUI exists. Furthermore the [configuration data](https://www.dropbox.com/scl/fo/6yycimmdb77x39viza91i/h?dl=0&rlkey=kwq70w5legsashswftvvm1sfl) of ETARA should be downloaded. They contain on the one hand all configurations of the Web APIs to be simulated and on the other hand the used datasets.

The datasets can be stored anywhere, but a TDB index must be created so that ETARA can successfully access the data and make requests to the databases. For this purpose the TDB Command Line Tools from Apache Jena can be used (see [Documentation](https://jena.apache.org/documentation/tdb2/tdb2_cmds.html)).

Depending on the point in time when the data was downloaded, it is possible that the configs folder is redundant, since ETARA already provides a folder with the same content. However, to be safe, we recommend replacing the configs folder in the Git repository with the data downloaded from the cloud.

Afterwards the file `databases.json` must be adapted in the configs folder. The database entries are composed as follows:

```
{
  "databases": {
    "example_db": {
      "path": "C:\\Databases\\example\\tdb",
      "source": "C:\\Databases\\example\\example.nt",
      "identifierMap": "configs\\identifierMaps\\example.json"
    }
  }
}
```

The JSON object `databases` consists of entries, also JSON objects, which store all the information that needs to be specified for a database. In this example, only one database is stored with the label `example_db`. Here `path` describes the path to the TDB index. The entry `source` points to the raw text file of the database. The access to the raw data is needed because ETARA initially generates a so-called identifier map. What exactly the identifier map is and how it works is irrelevant for the reproducibility of the results and is therefore not discussed in detail here. However, if you want to know more about the functionality and configuration options of the ETARA system, you can find documentation [here](https://github.com/anonresearcher123/ETARA).

To start ETARA you only need to adjust the `path` and `source` information of the databases. To do this, simply specify the location of the previously downloaded .nt data and the generated TDB index. After that ETARA is ready to run and can be started. The startup sequence should produce an output of the following form:

```
Initialising server..
Added /webservices/a0/work
Added /webservices/a1/work
Added /webservices/a2/work
Added /webservices/a3/work
...
Added /etara/webserviceManager
Server started: http://localhost:8080
```

Before the experiments can be repeated, a few settings must first be configured in TunA. First, download the TunA repository and open it with an IDE of your choice. Next, you need to register the local knowledge bases you are using. To do this, look at the file `config.json` in the root directory of the TunA repository. Since the file contains a lot of information it can seem a bit overwhelming, but for the reproduction of the evaluation only little customization is necessary. For you, only the `databases` section is important, as it refers to the local [Knowledge Bases](https://www.dropbox.com/scl/fo/nlger7mf71hlu0jb6kms8/h?dl=0&rlkey=yx6n48qz4n6lqobk8l1uz5mcc) used and their location. An entry in Databases has a similar structure as in the ETARA system:

```
{
  "databases": [
    {
      "label": "sample_dblp_gaps",
      "index": "C:\\Databases\\sample_dbs\\dblp_with_gaps\\tdb",
      "source": "C:\\Databases\\sample_dbs\\dblp_with_gaps\\dblp_with_gaps.nt"
    }
  ]
}
```

The field 'label' indicates the name of the knowledge base. As before, you must also create a TDB index for the knowledge bases used by TunA. Next you just have to adjust the paths to the TDB index and the source files in the `config.json` file.

Furthermore the used [funciton sets](https://www.dropbox.com/scl/fo/tx9cxjooowilh43zzcva9/h?dl=0&rlkey=guf47709enzr252xymvb34404) are needed for the experiments. Depending on when the data and the repository were downloaded, it is possible that the TunA already provides the function sets (see the following [subfolder](https://github.com/anonresearcher123/TunA/tree/master/res/functionstores)). However, to be safe, we recommend replacing the function sets or creating new folders for re-running the experiments. In the following sections, we will go into more detail about what to look for when replacing the data.

### How to reproduce the first experiment (comparison with optimal plans)?
The first experiment is used to compare TunA's computed execution plan with the optimum execution plan. Since only the number of required Web API calls and the estimated execution time are important, the configuration and use of ETARA is not necessary.

First you should make sure that the function set used for the first experiment exists in TunA. Check under `TunA/res/functionstores/` if a folder named `optimal` exists. This folder should contain all used function definitions for the first experiment. If you are not sure if the files are correct, you can also replace files in this folder with the following [files](https://www.dropbox.com/scl/fo/7qkt5hmfnmc6h1t8xpdrx/h?dl=0&rlkey=5wcqw3gtaimxh0tkip2unuogt).

Next, you will need the [queries](https://www.dropbox.com/scl/fo/e2xuz7do0nsr4j3fr8vzj/h?dl=0&rlkey=22ovum0nauqgnkb57og9x4j53) we used for this experiment. These can be saved to the `TunA/res/testcases/` folder. Next, you can start the class `TunaConsole.java` in your IDE, as this is used to start the TunA Query Optimizer.

```
___________               _____   
\__    ___/_ __  ____    /  _  \  
  |    | |  |  \/    \  /  /_\  \ 
  |    | |  |  /   |  \/    |    \
  |____| |____/|___|  /\____|__  /
                    \/         \/ 

-----------------------------
Data Management
-----------------------------
Select Database: sample_dblp_gaps
Select Function Store: optimal
Query File: bib0-3.sparql
Query Optimizer: tuna

-----------------------------
User Preferences
-----------------------------
Time Limit (ms): 500000.0
Minimum Coverage: 1.0
Minimum Reliability: 0.9
-----------------------------
```

First, TunA prompts the user for information about data management, i.e. which database to query, which function set to use, which query to pose, and which query optimizer to use. Note that you need to use `sample_dblp_gaps` for the bibliographic queries and `smaple_lmdb_gaps` for the filmographic queries. In order to chose the optimizer you can use `tuna` or `optimal`.  

Next, the user can specify the TunA specific user preferences like, maximum execution time, minimum coverage and reliability. Afterwards, a suitable (or optimal) query plan is identified. Note that we set the maximum execution time for each query that high, since we only wanted to evaluate if TunA is able to identify a suitable query plan for the required minimum coverage and reliability that is clos the the optimal query plan in terms of execution time and number of calls. The output of the identified query plan looks as follows:

```
-----------------------------
Plan
-----------------------------
Calls: 6
Est. Time: 3.435
Crawl Time: 1.93
Est. Coverage: 1.0
Est. Reliability: 0.96
-----------------------------
```

### How to reproduce the second experiment (evaluation of scalability and tunability)?
The second experiment focuses on the tunability and scalability of TunA. For the second and third experiment, in opposite to the first experiment, the ETARA system is needed. Therefore, the first step is to start the ETARA system as described at the beginning of this section.

Next, the TunA system must be prepared for the second experiment. To do this, you should first make sure that all three functions sets used are present in TunA. To do this, check under `TunA/res/functionstores/` if the three folders `evalF1`, `evalF2` and `evalF3` exist. These three folders should contain all function definitions for the second experiment. If you are not sure if these are the correct files, you can also replace the files of the folders with the following [files](https://www.dropbox.com/scl/fo/7oona0clz7f359sy9ee9v/h?dl=0&rlkey=3tks3e952yso0wn32vwfw9pl3).

Next, you will need the [bibliographic](https://www.dropbox.com/scl/fo/5lacogpg8t2x517mdhoxs/h?dl=0&rlkey=sp42ngh9nu8nl72ccsewxjwpb) and [filmographic](https://www.dropbox.com/scl/fo/n7z8ljhlxhguc6jhqwttc/h?dl=0&rlkey=6585a0xp6dimk57q62zepxmnx) queries we used for this experiment. For this experiment we only used  

These can, as before, be saved to the `TunA/res/testcases/` folder. Next, you can start the class `TunaConsole.java` in your IDE.

```
___________               _____   
\__    ___/_ __  ____    /  _  \  
  |    | |  |  \/    \  /  /_\  \ 
  |    | |  |  /   |  \/    |    \
  |____| |____/|___|  /\____|__  /
                    \/         \/ 

-----------------------------
Data Management
-----------------------------
Select Database: sample_dblp_gaps
Select Function Store: evalF1
Query File: query_file.sparql
Query Optimizer: tuna

-----------------------------
User Preferences
-----------------------------
Time Limit (ms): 500000.0
Minimum Coverage: 1.0
Minimum Reliability: 0.9
-----------------------------
```

As in the first experiment, TunA prompts the user for information about data management, i.e. which database to query, which function set to use, which query to pose, and which query optimizer to use. Note that you need to use `sample_dblp_gaps` for the bibliographic queries and `smaple_lmdb_gaps` for the filmographic queries. In order to chose the optimizer you can use `tuna` or `angie` for this experiment.   

Next, the user can specify the TunA specific user preferences like, maximum execution time, minimum coverage and reliability. Note that we set the maximum execution time for each query that high, since we only wanted to evaluate if TunA is able to achieve the requered minimum coverage and reliability values. The output of TunA for this experiment is slightly more elongated than in the first experiment. The output consists of a table that contains all the information provided in the query. The query result can now be checked for reliability and coverage.

### How to reproduce the third experiment (comparison with ANGIE)?
The last experiment compares the performance of the state of the art system ANGIE with TunA. ANGIE is a framework for generating queries to encapsulated RESTful Web APIs and efficient algorithms for query execution and result integration. TunA was implemented in Java using the Apache Jena framework. To make a comparison with ANGIE as fair as possible, ANGIE was likewise re-implemented in Java.

First you should make sure that the functions set of the last experiment exist in TunA. Check under `TunA/res/functionstores/` if a folder named `angie_benchmark` exists. This should contain all the function definitions used for the third experiment. If you are not sure if these are the correct files, you can also replace files in this folder with the following [files](https://www.dropbox.com/scl/fo/ic81rpht4ihmrd5wyeazh/h?dl=0&rlkey=wwhv1z4lmk9cxn1xncly731ge).

Next, you will need the [bibliographic](https://www.dropbox.com/scl/fo/g216bk9jv4yvx5ixl2j5g/h?dl=0&rlkey=jznnjvvdi23raaa66de532mfl) and [filmographic](https://www.dropbox.com/scl/fo/cbg1n2yvvbayyoygo7dja/h?dl=0&rlkey=9uz6rm3eqdsf5tg2194r2rtn4) queries we used for this experiment. These can, as before, be saved to the `TunA/res/testcases/` folder. Next, you can start the class `TunaConsole.java` in your IDE.

```
___________               _____   
\__    ___/_ __  ____    /  _  \  
  |    | |  |  \/    \  /  /_\  \ 
  |    | |  |  /   |  \/    |    \
  |____| |____/|___|  /\____|__  /
                    \/         \/ 

-----------------------------
Data Management
-----------------------------
Select Database: sample_dblp_gaps
Select Function Store: angie_benchmark
Query File: query_file.sparql
Query Optimizer: tuna

-----------------------------
User Preferences
-----------------------------
Time Limit (ms): 500000.0
Minimum Coverage: 1.0
Minimum Reliability: 0.9
-----------------------------
```

As in the first and second experiment, TunA prompts the user for information about data management, i.e. which database to query, which function set to use, which query to pose, and which query optimizer to use. Note that you need to use `sample_dblp_gaps` for the bibliographic queries and `smaple_lmdb_gaps` for the filmographic queries. In order to chose the optimizer you can use `tuna` or `angie` for this experiment.  

Next, the user can specify the TunA specific user preferences like, maximum execution time, minimum coverage and reliability. Note that we set the maximum execution time for each query that high, since we only wanted to evaluate if TunA and ANGIE are able to achieve the requered minimum coverage and reliability values. After the query result is printed to the console it can be checked for reliability and coverage.
