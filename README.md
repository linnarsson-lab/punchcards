# Punchcards

``punchcards`` is a repository containing the files used to specify the `cytograph` analysis described in the paper:

Molecular Architecture of the Developing Mouse Brain 
Gioele La Manno*+, Kimberly Siletti*, Alessandro Furlan, Daniel Gyllborg, Elin Vinsland, Christoffer Mattsson Langseth, Irina Khven, Anna Johnsson, Mats Nilsson, Peter Lönnerberg, Sten Linnarsson+

## What is a punchcard?

A punchcard is a recipe, a text file containing a brief set of instructions that defines a semi-automated single-cell RNA-seq analysis. 

Punchcards take advantage of the automatic labeling capabilities of the [auto-annotation](https://github.com/linnarsson-lab/auto-annotations) framework and of the streamlined [cytograph](https://github.com/linnarsson-lab/cytograph) pipeline to implement a declarative, transparent, rule-based splitting procedure to perform analysis of specific subsets of the data. 

A punchcard defines an analysis pipeline by specifying the parent analysis to use as input and the auto-annotation labels that identify the subset of clusters to analyze.

## What is the philosophy?

Clustering a diverse single-cell dataset might result in only a rough representation of its heterogeneity, with interesting differences remaining concealed. Such a resolution limit is an intrinsic limitation of an analysis that aims to balance global and local similarities. Therefore, it is not uncommon that the reanalysis of a few clusters reveals more considerable heterogeneity than what could be appreciated at a global level. In particular, repeating feature selection and dimensionality reduction on a more homogeneous set of cells often highlights further structure.

To extract the most information from a dataset, adopting an iterative divide-and-conquer approach is sensible. This strategy involves starting from a global clustering, then splitting clusters into a few partitions and iteratively repeating the procedure on each partition, until a stopping condition is met. However, how to partition the clusters at each iteration remains an open issue. Performing the partitioning in a supervised way has the advantage of allowing for the use of biological knowledge to expose particularly interesting aspects of the heterogeneity even when low signal-to-noise ratio makes them potentially difficult to highlight. A challenge is that those decisions quickly become verbose to describe, difficult to inspect, and harder to evaluate as the size of the dataset grows (e.g., instructions like analyze clusters 1-4 with 5-8). Finally, the approach requires continuous manual intervention at each rerun of the analysis.


## How does a Punchcard look?

The format and usage of the punchcard are described more fully in the cytograph repository. In brief, each punchcard is a human-readable yaml-formatted file containing the name of each subset and two fields: *include* and, optionally, *onlyif*. The former accepts a list of auto-annotation tags, and the latter a conditional statement that further constrains the cells to include (e.g. Clusters == 3).


## Curiosity
[Punch cards]((https://en.wikipedia.org/wiki/Punched_card#History)) were one of the first means to store digital information, in particular automata instructions/inputs. Noticeably the earliest automatic [loom](https://github.com/linnarsson-lab/loompy) was controlled by punched holes in a paper tape.
