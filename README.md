# dev-processes

dev-processes is repository containing cytograph analysis description files.

dev-processes contains aims at containing a complete collections on all the analysis performed on the developmental brain atlas dataset.


## Analysis description files

Analysis description file are .yaml files that contain all the information to run a cytograph-luigi Process analysis.
 
It is possible to run an Process analysis as following:

```bash
nohup luigi --workers 5 --local-scheduler --module cytograph StudyProcess --processname DifferentiationNeuralCrest > nohup.out &
```

below a model for analysis

```yaml
name: Process name extended
abbreviation: ProcessName

parent_analysis:
  type: Level1
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

todo_analysis:
- type: PlotGraphProcess
  kwargs: {}

comments: |
  PLEASE insert a extra information.
  This can be multiline
```

### Parent Analysis field

The parent analysis field aims at describing the parent analyses that generates the input files by the Process analysis. The StudyProcess will pool all from the .loom output files of the parent analysis, and will make use of the .aa.tab autoannotation files to filter cellsusing the exclude/include/timepoints conditions.

`type` is the parent analysis name as defined in cytograph. This is required to be a luigi wrapper task (i.e. inherits from luigi.WrapperTask).

`kwargs` are the keyword arguments used to call the parent-analysis. If it not required the parent can should be `kwargs: {}`

In the current version it is also assumed that the wrapper-task `.requires()` method returns an iterator of tuples containing Iterator[Tuple[luigi.Task, luigi.Task, ...]]. Where the order of the tasks is:
-  first a ClusterLayout-like taks (or any taks outputing a .loom with the `Age`, `Class_*` and `Clusters` column attribute
-  second  AutoAnnotate-like tasks (or any task outputing a .aa.tab file)
-  others... the order does not matter

## Exclude/Include and Timepoints filtering conditions

This entries describe the filtering that is performed on the cells.

Any of those options can be omitted, in that case the value will use the default value. Defaults values are the ones described by the `Model.yaml` file.

`include` is a dictionary of including conditions with default `all`. The different types of conditions are combined by a set union operator.
    - `auto-annotations` list, use the autoannotations contained in .aa.tab to filter clusters. 
    - `categories`: list, use categories to filter set of clusters that contain a certain category attribute
    - `classes`: list,  use the classifier probability found in the loom file (prob>0.5)
    - `clusters`: list,  use the cluster numbering to filter (NOTE!!! makes sense only when the source file is only one)
    
NOTE: for now only `auto-annotations` supports the and logical operator that is expressed providing a list. For example:

```yaml
auto-annotations:
- GUM
- ["@CC", MHBm]
```

Coresponds to the filter `GUM ∪ ( @CC ∩ MHBm )`

`exclude` the same as include but for excluding. default is `none`. The different types of negative conditions are combined by a set union operator.

`timepoints` takes a list of entries, range are not supported yet

The final filter is `(include - exclude) ∩ timepoints`

## TODO Analysis

Describe in a list the set of analyses to be performed.
By default (hard-coded) ClusterLayoutProcess, AutoAnnotateProcess will be run for every process

## Comment section

You can add here description of the process and relevant litterature

## Real-World Example

```yaml
name: Early Differentiation of Neural Crest into different lineages
abbreviation: DifferentiationNeuralCrest

parent_analysis:
  type: Level1
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

todo_analysis:
- type: PlotGraphProcess
  kwargs: {}
- type: PlotGraphAgeProcess
  kwargs: {}

comments: |
  PLEASE insert a extra information.
  This can be multiline

```