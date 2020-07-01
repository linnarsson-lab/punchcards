# Punchcards

``punchcards`` is a repository containing description files to feed to cytograph `punchcards` analysis pipeline.
It contains the collection of all the semi-automated analyses performed on the developmental brain atlas dataset.

## What is a punchcard?

A Punchcard is a recipe, a text file containing a brief set of instructions that fully define a semi-automated single-cell RNA-seq data. 

Punchcards take advantage of the automatic labeling capabilities of the [auto-annotation](https://github.com/linnarsson-lab/auto-annotations) framework and of the streamlined [cytograph](https://github.com/linnarsson-lab/cytograph) pipeline to implement a declarative, transparent, rule-based splitting procedure to perform analysis of specific subsets of the data. 

A punchcard defines an analysis pipeline by specifying the parent analysis to use as input and the auto-annotation labels that identify the subset of clusters to analyze.

## What is the philosophy?

Clustering a diverse single-cell dataset might result in only a rough representation of its heterogeneity, with interesting differences remaining concealed. Such a resolution limit is an intrinsic limitation of an analysis that aims to balance global and local similarities. Therefore, it is not uncommon that the reanalysis of a few clusters reveals more considerable heterogeneity than what could be appreciated at a global level. In particular, repeating feature selection and dimensionality reduction on a more homogeneous set of cells often highlights further structure.

To extract the most information from a dataset, adopting an iterative divide-and-conquer approach is sensible. This strategy involves starting from a global clustering, then splitting clusters into a few partitions and iteratively repeating the procedure on each partition, until a stopping condition is met. However, how to partition the clusters at each iteration remains an open issue. Performing the partitioning in a supervised way has the advantage of allowing for the use of biological knowledge to expose particularly interesting aspects of the heterogeneity even when low signal-to-noise ratio makes them potentially difficult to highlight. A challenge is that those decisions quickly become verbose to describe, difficult to inspect, and harder to evaluate as the size of the dataset grows (e.g., instructions like analyze clusters 1-4 with 5-8). Finally, the approach requires continuous manual intervention at each rerun of the analysis.


## How does a Punchcard look?

A punchcard consists of a human-readable yaml-formated file with the three fields: *require, include and exclude* defining the subset of cells to reanalyze, and the fields name, abbreviation and comments intended for human readability. The require field is used to specify the input dataset that can be a cytograph analysis or another punchcard, thus enabling hierarchical analyses. The include and the exclude fields define rules for subsetting clusters from the parent dataset. Subsetting rules are constrained to be either a list of auto-annotation labels and categories, or a boolean array. Because those rules are not necessarily specific to a particular analysis, punchcards define a series of biologically meaningful analyses that remain general to the specific input dataset and analysis parameters. The subset of cells will influence the feature selection and the granularity of the clustering, revealing additional information about cellular variability.


## Analysis description files

A `punchcard` is ``.yaml`` file that contains all the information required to run a cytograph-luigi *Punchcard* analysis.
(Note: The *Punchcard* lugig analysis is being refactored and it is currently broken).
 
It is possible to run an analysis as following:

```bash
luigi --workers 5 --local-scheduler --module development-mouse Punchcard --card DifferentiationNeuralCrest
```

A `punchcard` looks like this:

```yaml
name: Selecting motor/gly/ser/chol/th neurons from Hindbrain E9-11
abbreviation: HindbrainE911MiscNeurons

require:
- type: Punchcard
  kwargs:
  card: HindbrainE911NonRgl
 
include:
  auto-annotations:
  - "@GLY"
  - HinSert
  - VscMN
  - SpMN
  - CrMN
  - "@CHOL"
 
exclude:
  auto-annotations:
  - "@VGLUT2"
  - "@GABA"

comments: |
```


### Parent Analyses entry

The parent analysis entry describes a list of `luigi.WrapperTask` that generates the inputs files for the current StudyProcess analysis. *StudyProcess* will make use of the .loom output files and the .aa.tab autoannotation files to filter cells. Filtering is specified in the exclude/include/timepoints entries.

`type` is the parent analysis name as defined in cytograph. This is required to be a `luigi.WrapperTask` and its use needs to be enabled in cytograph.

`kwargs` are the keyword arguments used to call the parent-analysis. If it not required the parent should be `kwargs: {}`

In the current version it is also assumed that the `luigi.WrapperTask.requires()` method returns an iterator of tuples `type: Iterator[Tuple[luigi.Task, luigi.Task, ...]]`. Where the order of the tasks is:

* first a *ClusterLayout*-like task (or any task outputing a .loom file with the `Age`, `Class_*` and `Clusters` column attribute
* second  AutoAnnotate-like tasks (or any task outputing a .aa.tab file)
* others... the order does not matter

## Exclude/Include and Timepoints filtering conditions

This entries describe the filtering that is performed on the cells.

Any of the options below can be omitted, in that case the value will use the default value. Defaults values are described by the `Model.yaml` file.

* `include` is a dictionary that specify wich cell to include, the default is `all`. The different types of conditions are combined by a set interesection operator.
* `auto-annotations` list, use the autoannotations contained in .aa.tab to filter clusters.  
NOTE: for now only `auto-annotations` supports the `and` logical operator. This can be expressed providing a list instead of a string. For example the following example coresponds to the filter `GUM ∪ ( @CC ∩ MHBm )`

        auto-annotations:
        - GUM
        - ["@CC", MHBm]

* `categories`: list, use categories to filter set of clusters that contain a certain category attribute
* `tissues`: list, use the tissue name (as it is found in the loom column attributes)

`exclude` the same as include but for excluding. default is `none`. The different types of conditions are combined by a set union operator (consider exclude == True, notexclude == False).

`timepoints` takes a list of entries, ranges are not supported yet

The *final* filter is `(include - exclude) ∩ timepoints`

## Curiosity
[Punch cards]((https://en.wikipedia.org/wiki/Punched_card#History)) were one of the first means to store digital information, in particular automata instructions/inputs. Noticeably the earliest automatic [loom](https://github.com/linnarsson-lab/loompy) was controlled by punched holes in a paper tape.
