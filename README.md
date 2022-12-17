# TunA: Tunable Query Optimizer for Web APIs and User Preferences

## What is TunA?
TunA is a query optimizer that is able to combine RESTful Web APIs and RDF KBs in form of triple stores, tuning its (query) plans towards user preferences. Erroneous information from Web APIs is detected using a hierarchical agglomerative clustering. Our evaluation shows that TunA outperforms current state-of-the-art systems and is less vulnerable for erroneous information, even in settings where only unreliable sources are available

## How to run TunA?

## Where to find the data sets used for the evaluation?
For the evaluation we used as data sources the ones provided by [ETARA](#) (an RDF version of dblp and data exports of the RESTful Web APIs from CrossRef,
SciGraph and Semantic Scholar). Additionally, we created three tainted data sets, that contained erroneous data (i.e. incorrect titles, etc.) and removed infor-
mation, e.g., titles and author names, from the dblp dataset. 

[Download used data sets](#)
