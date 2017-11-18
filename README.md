# Punchcards

``punchcards`` is a repository containing description files to feed to cytograph `punchcards` analysis pipeline.

``punchcards`` will contain a complete collection of all the sub-analysis performed on the developmental brain atlas dataset.

# Curiosity/History
[Punch cards]((https://en.wikipedia.org/wiki/Punched_card#History)) were one of the first means to store digital information, in particular automata instructions/inputs. Noticeably the earliest automatic [loom](https://github.com/linnarsson-lab/loompy) was controlled by punched holes in a paper tape.

## Analysis description files

A `punchcard` is ``.yaml`` file that contains all the information required to run a cytograph-luigi *Punchcard* analysis.
(Note: The *Punchcard* lugig analysis is being refactored and it is currently broken).
 
It is possible to run an analysis as following:

```bash
luigi --workers 5 --local-scheduler --module development-mouse Punchcard --card DifferentiationNeuralCrest
```

A `punchcard` looks like this:

```yaml
name: Punchcard name extended
abbreviation: PunchcardName

requires:
- type: Level1
  kwargs:
    target: All
    time: E7-E18

include:
  auto-annotations: all
  categories: all
  classes: all
  clusters: all

exclude:
  auto-annotations: none
  categories: none
  classes: none
  clusters: none

timepoints: all

run:
- type: PlotGraphProcess
  kwargs: {}

comments: |
  PLEASE insert a extra information.
  This can be multiline

```

# Note: the doc below is outdated

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
* `classes`: list,  use the classifier probability found in the loom file (prob>0.5)
* `clusters`: list,  use the cluster numbering to filter (NOTE!!! makes sense only when the source file is only one)
* `tissues`: list, use the tissue name (as it is found in the loom column attributes)

`exclude` the same as include but for excluding. default is `none`. The different types of conditions are combined by a set union operator (consider exclude == True, notexclude == False).

`timepoints` takes a list of entries, ranges are not supported yet

The *final* filter is `(include - exclude) ∩ timepoints`

## Todo Analysis

It describe in a list of *lugi.Task* to be performed. They are specified as in `parent_analysis`.

By default (hard-coded) *ClusterLayoutProcess*, *AutoAnnotateProcess* will be run for every process.

## Comment section

You can add here description of the process and relevant litterature

## Example of a real analysis

```yaml
name: Early Differentiation of Neural Crest into different lineages
abbreviation: DifferentiationNeuralCrest

parent_analyses:
- type: Level1
  kwargs:
    target: All
    time: E7-E18

include:
  auto-annotations:
    - CNC
    - FrontM
    - SCHWA
    - PSN
    - CrNerv

exclude:
  auto-annotations:
    - GUM
    - ["@CC", MHBm]

timepoints:
  - E8
  - E9
  - E10
  - E11
  - E12

todo_analyses:
- type: PlotGraphProcess
  kwargs: {}
- type: PlotGraphAgeProcess
  kwargs: {}

comments: |
  PLEASE insert a extra information.
  This can be multiline


```
