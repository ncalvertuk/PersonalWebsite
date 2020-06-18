+++
title = "Binderising Exercises from Doing Data Science Book"
date = "2020-06-18T13:33:0+01:00"
draft = false
+++
### Introduction
I've been going through the [Doing Data Science : Straight Talk from the Frontline book][booklink] by Cathy O'Neil & Rachel Schutt published by O'Reilly Media and working through the exercises in Julia. I'll keep this post updated as I work through it and post more examples. I have written the code in notebooks using [nteract](https://nteract.io/). As I was going through the exercises I found a tutorial on how to generate a Binder from notebooks, and I thought it might be a good idea to try this out as well. I've briefly outlined how I did this below, before going through each of the exercises I did very quickly.

[booklink]: https://www.oreilly.com/library/view/doing-data-science/9781449363871/

### Binder

[Binder](https://mybinder.readthedocs.io/en/latest/) "allows you to create custom computing environments that can be shared and used by many remote users". Essentially it allows you to build an online Docker image of a GitHub repository, which seems like it could be very handy for teaching or sharing code from a publication, etc. There is a tutorial on how to go from "Zero-to-Binder" in [Python](https://github.com/alan-turing-institute/the-turing-way/blob/master/workshops/boost-research-reproducibility-binder/workshop-presentations/zero-to-binder-python.md), [R](https://github.com/alan-turing-institute/the-turing-way/blob/master/workshops/boost-research-reproducibility-binder/workshop-presentations/zero-to-binder-r.md), and [Julia](https://github.com/alan-turing-institute/the-turing-way/blob/master/workshops/boost-research-reproducibility-binder/workshop-presentations/zero-to-binder-julia.md) provided by the [Alan Turing Institute](https://www.turing.ac.uk/).

I followed the Julia tutorial which is relatively straightforward however did provide a few hurdles to overcome along the way! Firstly I created a repo from my Doing Data Science notebooks, rather than create a new hello, world script. The first thing I did was open up my [JuliaPro IDE](https://juliacomputing.com/products/juliapro) and change directory to my repository. I entered the package manager by pressing `]` and typed in `activate .`. I executed the following
```
add DataFrames, Gadfly, Queryverse, Statistics, Dates,Statistics, Random, Plots, GLM
```
which generates the `Project.toml` and `Manifest.toml` files which contain the dependencies that I'll be using. To the `Project.toml` I also added
```
[compat]
julia = "1.4"
```
to ensure I'm using the same version of Julia in my Binder. I then attempted to build my first Binder, which failed! The errors pertained to some dependencies failing, in particular it could not find the correct versions of some packages on GitHub that matched the versions I have installed on my machine. In particular, this failed for the [```BinaryProvider```](https://github.com/JuliaPackaging/BinaryProvider.jl) and [```GR```](https://github.com/jheinen/GR.jl) packages. To get around this issue I had to add the latest versions of the packages to my project using the following.
```
add https://github.com/JuliaPackaging/BinaryProvider.jl
add https://github.com/jheinen/GR.jl
```
The next issue (fortunately also the final issue!) was related to the ```PyCall``` package that was being used by some of the dependencies. In particular, the Excel Reader in ```QueryVerse``` was calling the ```xlrd``` Python package, however this could not be found by the Binder. This issue can be solved by creating a ```requirements.txt```, which lists the Python dependencies required, and entering the following.
```
xlrd==1.1.0
```
Further Python packages can be included in the same manner. After including this file the Binder compiled with no problems, although it did take quite a long time! The link to Binder for the entire repo can be found here:  [![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/ncalvertuk/DoingDataScienceNbs_Julia/master) and the links to each individual notebook are given at the beginning of each section below.
