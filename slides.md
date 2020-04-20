---
marp: true
theme: gaia
_class: lead
paginate: true
backgroundColor: #fff
backgroundImage: url('https://marp.app/assets/hero-background.jpg')
---

# Why?

* Python >> shell scripts, Matlab
* Your workflow will be reusable by others and your future self

--- 
# Typical process without a workflow manager

--- 
# Snakemake at-a-glance

* inspired by Makefiles 
    * if you don't know how Make works, don't worry about it..
* 

--- 
# Snakemake vs Nipype

Both are workflow management environments.

---
## Nipype pros:
* was developed for neuroimaging, so many wrappers and helper scripts specifically for tools like FSL, SPM, etc.. 
* workflows can understand files 
---
## Nipype cons:
* To use a tool, you must *wrap* it
    * i.e. write some code to indicate what the input/output parameters, files are 
* Complicated means for parallel execution & file i/o
    * MapNodes, Identity iterables, DataSinks
* Does not support workflows with heterogeneous dependencies
    * Everything must be in a single environment
* Code/workflow is not very readable
    * Many layers of abstraction to get to the actual code of interest
* Has been challenging to adopt universally
    
--- 

--- 
# How to get started


## Quick-start on graham:
* Install latest neuroglia-helpers
  * make sure to use the khanlab cfg
* This includes shortcuts to a virtualenv that has snakemake installed, as well as wrappers for cluster execution
* Note: if you want to customize your python environment, you can also follow the instructions on the snakemake docs to install miniconda and snakemake


--- 
# Cool features

## Easily combine shell and python
Can use the `shell()` command inside any Python `run:` blocks, even inside Python loops

## Don't have to worry about creating folders or checking if files exist

## Visualization of the workflow is simple (similar to nipype)


--- 
# Rules

Each rule should be a step in your workflow that generates some file(s)

Can be some inline shell or python code (`shell:` or `run:` directives), or external Python (or R) scripts

