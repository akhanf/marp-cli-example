---
marp: true
theme: gaia
paginate: true
backgroundColor: #fff
backgroundImage: url('https://marp.app/assets/hero-background.jpg')
headingDivider: 1
---
Snakemake for Neuroimaging
===

# Snakemake Intro Slides

<https://slides.com/johanneskoester/ismb-snakemake-tutorial-2019#/>

# Typical workflow for neuroimaging researcher

1) Start with some data
2) Perform some processing on it using tool you downloaded, or built with a python/matlab script
3) This generates some new processed data
4) Repeat steps 2-3 until satisfied


# Common challenges with this kind of workflow?

* What if you want to change something done early on?
* What if you want to run it on a different dataset?
* What if you want to run it on 1000 subjects (instead of 20)? 
* What if you want someone else to run or modify your workflow?


# Snakemake FTW

* Text-based workflow, extends Python
* Uses rules to define how output files are created, and to automatically determine dependencies
* Easy-to-read workflows that can combine Python, shell, R, ...
* Contains rules using conda environments and/or Singularity containers
* Easily scales to any compute resource


# Outline for this introduction

* Describe the basic components of a Snakefile 
* Highlight some cool features you can add to your workflow
* Take a peek at a more complex workflow
* Describe cluster execution on graham

 

# Snakefile

* A Pythonic text file that defines **rules** for generating files
* Dependencies are automatically determined by starting with the **final** output file (*target*), and working backwards
* This will be a bit counter-intuitive at first.. 


# Rules

Each rule should be a step in your workflow that generates some file(s)

Can be some inline shell or python code (`shell:` or `run:` directives), or external Python (or R) scripts

# Wildcards
- Provides a way to generalize a rule
- Wildcards are automatically resolved by (regular expression) patterns in the **output** filenames

```
rule complex_conversion:
    input:
        "{dataset}/inputfile"
    output:
        "{dataset}/file.{group}.txt"
    shell:
        "somecommand --group {wildcards.group} < {input} > {output}"
```        

# Aggregation with expand()

- Input/output files can also be **lists**
- e.g. in Python, we can create a list as:
```
rule aggregate:
    input:
        expand("{dataset}/a.txt", dataset=DATASETS)
    output:
        "aggregated.txt"
    shell:
        ...
```

# Functions as input files
- For more flexibility, you can also have your input files defined by a function
- This function **must** take a `wildcards` argument, which contains the wildcards resolved by the output files
```
def myfunc(wildcards):
    return [... a list of input files depending on given wildcards ...]

rule:
    input: myfunc
    output: "someoutput.{somewildcard}.txt"
    shell: "..."
```
# Log files

- Log files can be defined with `log:` but you have to make sure your code actually writes to them!
```
rule abc:
    input: "input.txt"
    output: "output.txt"
    log: "logs/abc.log"
    shell: "somecommand --log {log} {input} {output}"
 ```


# Configuration files
- Written in YAML/JSON
- Defines a dictionary of config parameters
```
bids_dir: /path/to/bids_dir
seeds:
 - ZI
 - STN
```
- Variables can also be overriden at command-line
```
$ snakemake --config yourparam=1.5
```

# Cool features in Snakemake

# Easily combine shell and python
Can use the `shell()` command inside any Python `run:` blocks, even inside Python loops
```
for line in shell("somecommand {output.somename}", iterable=True):
    ... # do something in python
```    
# Runs rules only as needed
* Snakemake checks file existence and modication times to determine if a rule should be run (*)
* Automatically creates needed sub-folders
* If a job fails (e.g. exit non-zero), any output files are removed
</br>
* (*) Note: If a file exists, but it corrupted, snakemake will not know the difference. Would need to force re-run, or delete the file first.. 

# Visualization of the workflow is simple (similar to nipype)

- The graph can be output in a programmatic format amenable to visualization 
- The `graphviz` package has a tool, `dot`, to render these graphs
```
$ snakemake --rulegraph | dot -Tpng > rulegraph.png
```
- There are `--dag` and `--filegraph` options, but can be huge if many files

# Generated Rule graph 
![height:500px](https://github.com/akhanf/zona-diffparc/raw/master/doc/rulegraph.png)

# Automated generation of interactive HTML reports
* You can easily create a report showing the workflow you ran:
```
$ snakemake --report
```
[example report](report.html)


# Best practices for reproducibility
- Keep your workflow in a git repository
- Built-in `--lint` can let you know if you are following current best practices
- Can use cookiecutter to generate a skeleton repo from a template:
```
cookiecutter gh:snakemake-workflows/cookiecutter-snakemake-workflow
```
# Archive a workflow

- Stores everything you need to produce the outputs (including input files, code, config files etc) in a single `tar.gz`
```
$ snakemake --archive my-workflow.tar.gz
```
- Then, someone can **easily** reproduce same analysis on the same data by running:
```
tar -xf my-workflow.tar.gz
snakemake -n
```



# Running workflows on graham

* Some recommendations on how to easily get started on graham


# Getting started on graham:
- Pick your favorite text editor (vi/emacs/nano etc)
- Install latest neuroglia-helpers
    - make sure to use the khanlab cfg
```
ssh graham.computecanada.ca
git clone https://github.com/khanlab/neuroglia-helpers
~/neuroglia-helpers/setup.sh khanlab
```

# Snakemake & Neuroglia-helpers
* Includes shortcuts to a virtualenv that has snakemake installed, as well as wrappers for cluster execution
* Note: if you want to customize your python environment, you can also follow the instructions on the snakemake docs to install miniconda and snakemake

# Different options to run snakemake:

1) Use an interactive job to run `snakemake` (e.g. `regularInteractive`)
2) Submit a job that runs `snakemake` (e.g. `regularSubmit`)
3) Use the `snakemake_slurm` wrapper
    * submits a job for each rule instance
    * for workflows with many small jobs, should use `group: ` directive to group rules in the same job
4) Use the `snakemake_remotebatch` wrapper to split the DAG into batches and submit each as a separate job



# Setting resources for jobs

* By default, rules have resources of 1core, 1000 mb memory, and 1 hr walltime
* If a rule requires more resources, you should specify that with directives so snakemake knows how much it can parallelize
* Note: resources always have to be type `int` 


---
E.g.: this rule is expected to use 4 cores, 4gb memory, and takes 6 hours
```
rule myjob:
    ...
    threads: 4
    resources:
        mem = 4000,
        time = 360
    shell: ...
```

# Interactive jobs:

- Interactive jobs are typically 8-core, 32gb memory, for 3 hours
- You should make sure you set the resources available to snakemake when you run it:
```
snakemake -j 8 --resources mem=4000 
```

# Interactive Jupyter notebook:
- Neuroglia-helpers also has a shortcut to start a jupyter notebook session on graham that you can connect to from your own 
browser

From your working folder on graham, run:
```
jupyter_snakemake
```
Then on your local computer use `sshuttle` so you can connect to the remote host via your browser.

# Pitfalls
 
* Stuck when Building DAG
    * Many files and inputs can make building the DAG take a looong time.. 
    * Intrinsic weakness of Snakemake
    * You can help alleviate this somewhat by using the `batch` option
* Input functions can be tricky 

# Start snakemaking!
* Your future self will thank you!
* Join #snakemake on slack!
* Post your questions/issues there, along with any Python questions!
* Happy to help walk through creating your first snakefile


# Appendix A - Resources

Snakemake Documentation:
https://snakemake.readthedocs.io/en/stable/index.html

Neuroglia-helpers for Khanlab Snakemake environment: https://github.com/khanlab/neuroglia-helpers

# Appendix B - Snakemake vs Nipype

* Both are workflow management environments, built with Python

# Nipype pros:

* Was developed for neuroimaging, so many wrappers and helper scripts specifically for tools like FSL, SPM, etc.. 
* Workflows can understand files such as Nifti
* Pipeline is explicitly described, rather than inferred through rules

# Nipype cons:

* To use a tool, you must *wrap* it
    * i.e. write some code to indicate what the input/output parameters, files are 
* Complicated means for parallel execution & file i/o
    * MapNodes, Identity iterables, DataSinks
* Does not support workflows with heterogeneous dependencies
* Code/workflow is not very readable
    * Many layers of abstraction to get to the actual code of interest

    


