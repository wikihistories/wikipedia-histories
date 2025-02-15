# Wikipedia Histories
[![Downloads](https://pepy.tech/badge/wikipedia-histories)](https://pepy.tech/project/wikipedia-histories)
[![Downloads](https://pepy.tech/badge/wikipedia-histories/week)](https://pepy.tech/project/wikipedia-histories/week)

A tool to pull the complete revision history of a Wikipedia page.

## Installation

To install Wikipedia Histories, simply run:

```bash
$ pip install wikipedia-histories
```

Wikipedia Histories is compatible with Python 3.6+.


## Usage
The module has basic functionality which allows it to be used to collect the revision history and metadata from a Wikipedia page in a convenient list of objects, which can be converted into a DataFrame. This also includes the article quality from every revision.

```python
>>> import wikipedia_histories

# Generate a list of revisions for a specified page
>>> golden_swallow = wikipedia_histories.get_history('Golden swallow')

# Show the revision IDs for every edit
>>> golden_swallow
# [130805848, 162259515, 167233740, 195388442, ...

# Show the user who made a specific edit
>>> golden_swallow[16].user
# u'Snowmanradio'

# Show the text of at the time of a specific edit
>>> golden_swallow[16].content
# u'The Golden Swallow (Tachycineta euchrysea) is a swallow.  The Golden Swallow formerly'...
>>> golden_swallow[200].content
# u'The golden swallow (Tachycineta euchrysea) is a passerine in the swallow family'...

# Get the article rating at the time of the edit
>>> ratings = [revision.rating for revision in golden_swallow]
>>> ratings
# ['NA', 'NA', 'NA', 'NA', 'stub', 'stub', ...

# Get the time of each edit as a datetime object
>>> times = [revision.time for revision in golden_swallow]
>>> times
# [datetime.datetime(2007, 5, 14, 16, 15, 31), datetime.datetime(2007, 10, 4, 15, 36, 29), ...

# Generate a dataframe with text and metadata from a the list of revisions
>>> df = wikipedia_histories.to_df(golden_swallow)
```

Additional metadata for the article, including 
An example of this workflow is available in `tests/demo.py`.

## Domain level analysis
This module also contains functionality for advanced analysis of large sets of Wikipedia articles by generation social networks based on the editors who edited an article. This functionality can be utilized by installing:

```
pip install wikipedia_histories[networks]
```

The toolkit is available at `wikipedia_histories.networks.analyze_networks` and `wikipedia_histories.networks.network_builder`.

First, a domain is defined as a `dictionary` or `json`, where keys are domain names and values are lists of categories which represent that domain. For example, a set of domains representing "culture" and "politics":

```json
{
  "culture": [
      "Category:Television_in_the_United_States",
      "Category:American_films",
      "Category:American_novels"
   ],
   "politics": [
      "Category:Conservatism",
      "Category:Liberalism"
   ]
}
```

An example of this format is available in `examples/domains.json`.

The articles represented by those domains, up to a certain depth of nested categories, can be collected and saved as a `csv`, with the category and domain attributes attached using `wikipedia_histories.networks.get_category_articles.find_articles()`. Once this set of articles is collected, the articles themselves can be downloaded using `wikipedia_histories.get_history()` either with revision text or without. This set of articles can be used for analysis on Wikipedia revision behavior across categories or domains. 

Once a set of articles is downloaded using this methodology, it's possible to collect aggregate metadata for those articles, including the number of unique editors, average added words per edit and average deleted words per edit, the article age, and the total number of edits, and save that information into a DataFrame using `wikipedia_histories.get_metadata()`.

An example of this workflow is available in `examples/collect_articles.py`.


## Social network analysis
It is also possible to build and analyze the networks of users who edited those articles, and study how domains relate to one another. For this analysis, first a set of articles representing categorical domains must be downloaded using and saved to folders representing domains and the metadata sheet must be saved. 

Once this is set up, a set of networks representing connections within a domain or between domains can be generated. A `domain` is passed as input to signify which domain should be used to build the networks, if no `domain` is passed as input the networks generated will represent connections between categories from different domains. 


In each network created, nodes represent articles and weighted edges represent the number of common editors between two articles. The function `wikipedia_histories.networks.network_builder.generate_networks()` allows generation of a certain number of networks with a specific number of nodes and a specific count--because they are generated by sampling from the downloaded articles, generating many networks represents bootstrapping of the dataset. 

The function call:

```python
networks = wikipedia_histories.networks.network_builder.generate_networks(
    count=1000,
    size=300,
    domain=domain,
    metadata_path=metadata_path,
    articles_path=articles_path,
)
```

would generate 1000 networks, each with 300 nodes, or 150 nodes from each selected category. Because the category input is `None`, the two selected categories would be from different domains. The `metadata_path` parameter is a path to the metadata sheet generated by the `find_articles()` function and the `articles_path` parameter is a path to the articles downloaded based on the `find_articles()` metadata.

The function returns a list of `NetworkX` objects. Networks can be written to the disk as `.graphml` files as part of the function by toggling the `write` parameter to `True` and passing an `output_folder` (note that this aspect is necessary for analysis).

Once generated, the networks can be analyzed using the `get_network_metadata()` function, which returns a DataFrame containing purity scores based on Louvain communities detected and assortativity scores for each network based on the categories represented by the networks.

An example of this workflow is available in `examples/collect_networks.py`.

Wikipedia Histories is compatible with Python 3.6+.

## Notes

This package was used for a paper published by the McGill .txtlab: https://txtlab.org/2020/09/do-wikipedia-editors-specialize/.
