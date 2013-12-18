% Using Make-like and Makefiles in R
% Rob Schick 
% November 28, 2013


# Introduction
- Reproducible research
- Motivation: I wanted a better way to link multiple files/analysis chunks together
- make-like files in R
- Makefiles at the command line

# Research Scandal at Duke
![alt text](images/duke60.png)

# Research Scandal at Duke - further reading
![baggerly article](images/baggcombs.png)
* [http://bit.ly/1bnzxUT](http://bit.ly/1bnzxUT)

# Reproducible research in psychology
<img src="images/nature1.png" alt="Nature" style="width: 600px;"/>

# Reproducible research in psychology
<img src="images/nature2.png" alt="nature 2" align = 'middle' style="width: 600px;"/>

# Reproducible research in the news
* Roger Peng's excellent series on reproducible research within R:
  * [http://bit.ly/1bnz6tO](http://bit.ly/1bnz6tO)
  * [http://bit.ly/1cuKL6O](http://bit.ly/1cuKL6O)
  * [http://bit.ly/1hi2yUs](http://bit.ly/1hi2yUs)
  
# Roger Peng's Reproducible Research Schematic
<img src="images/dsmpic.png" alt="Peng" align = 'middle' style="width: 700px;"/>

# Sample Workflow form Christopher Gandrud
<img src="images/Workflow.png" alt="Peng" align = 'middle' style="width: 700px;"/>


# make-like files vs. makefiles
* make-like files are simply *.R files
* easy to code
* less easy to maintain
* difficult to chain together 
* if one file is computationally intensive, you would want to run them _only_ when they are updated

# make-like Files in R
Inspired by Gandrud (2013):

```r
setwd("ExampleProject")
source("gather1.R")
source("gather2.R")
source("gather3.R")
source("mergedata.R")
```


# make-like Files in R: changing by hand

```r
setwd("ExampleProject")
source("gather1.R")
# source('gather2.R')
source("gather3.R")
source("mergedata.R")
```

* There must be a better way...

# What's a make file?
* Defined:

> In software development, Make is a utility that automatically builds executable programs and libraries from source code by reading files called makefiles which specify how to derive the target program. Though integrated development environments and language-specific compiler features can also be used to manage a build process, Make remains widely used, especially in Unix.

* Created by Stuart Feldman at Bell Labs in 1976
* [http://en.wikipedia.org/wiki/Make_(software)](http://en.wikipedia.org/wiki/Make_(software))

# How does it work?
* By comparing the time stamps of input and output files
* If the source file is newer than the output file, make will rerun the input file
* If the output file is newer than the source file, make will skip the file

# In Make terminology
* output files are called "targets"
* source files are called "prerequisites"
* You need to specify a "recipe" to create the targets from the prerequisites:

# Visually

```r
TARGET... : PREREQUISITE ...
  RECIPE
```

* tabs are important and meaningful

# Example makefile

```r
RDIR = .
MERGE_OUT = MergeData.Rout

RSOURCE = $(wildcard $(RDIR)/*.R)

OUT_FILES = $(RSOURCE:.R=.Rout)

all: $(OUT_FILES)

$(RDIR)/%.Rout: $(RDIR)/%.R
	R CMD BATCH $<

clean:
	rm -fv $(OUT_FILES)

cleanMerge:
	rm -fv $(MERGE_OUT)
```



# Let's work through these in turn

```r
RDIR = .
MERGE_OUT = MergeData.Rout
```

* set the working directory with .
* set the variable for the output of the last file
  * we'll need this to ensure we always run this file

# List the .R Source Code Files

```r
RSOURCE = $(wildcard $(RDIR)/*.R)
```

* Keeping the working directory with RDIR
* finding all source files that end with .R
* wildcard creates a list of all these files
  * default syntax is `$(wildcard PATTERN)`
* in make, $(variable) substitutes the value of the variable

# Using .Rout files within make

```r
OUT_FILES = $(RSOURCE:.R=.Rout)
```

* make will use the .Rout files to determine when each .R file was run
* `$(RSOURCE:.R=.Rout)` uses the file name form `RSOURCE` and changes the extension to .Rout
* `gather1.R` becomes `gather1.Rout` automagically

# This tells make what we want to create

```r
all: $(OUT_FILES)
```

* This is what Make tries to create when you type `make` at the command prompt with no targets

# This actually runs the R source code

```r
$(RDIR)/%.Rout: $(RDIR)/%.R
  R CMD BATCH $<
```

* The first line specifies that .Rout files are the TARGETS of the .R files
* the second line runs these files

# A different TARGET: clean

```r
clean:
  rm -fv $(OUT_FILES)
```

* if you type `make clean` then make follows the recipe that says `rm -fv` for all the out files

# Pushing changes downstream

```r
cleanMerge:
  rm -fv $(MERGE_OUT)
```

* This makefile doesn't push changes downstream
  * if we change `Gather2.R` then run `make` only `Gather2.R` will be rerun
* this line ensures the merged dataset will be rerun by removing the `MergeData.Rout` file

# Running the file
* Can do this from the command line:
  * using xtools on a mac
  * using rtools on windows
* Can run it straight from RStudio
  * Create a makefile in the same directory
  * Use the Build tab that automatically pops up when you have the makefile
  
# Let's see it in action
![alt text](images/buildtab.png)
*  (example)

# Dependencies
* We can structure the makefile to account for dependencies
* (example)
* For example, only run `runRegression.R` if `getData.R` is updated:

```r
$(RDIR)/runRegression.Rout: $(RDIR)/runRegression.R $(RDIR)/getData.R  
  R CMD BATCH $<
```

* These can get a little tricky (example)

```r
$(RDIR)/outputTable.Rout: $(RDIR)/outputTable.R $(RDIR)/runRegression.Rout 
  R CMD BATCH $<
```


# $\LaTeX$, Make, and R
* It is possible to have one makefile for your analysis code and your writing code

```r
$(TEXFILE).pdf: $(TEXFILE).tex $(OUT_FILES) $(CROP_FILES)
  latexmk -pdf -quiet $(TEXFILE)
```

* See Rob Hyndman's post: [http://bit.ly/1bnUtLp](http://bit.ly/1bnUtLp)

