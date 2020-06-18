+++
title = "Binderising Exercises from the Doing Data Science Book"
date= "2020-06-17T13:53:02+01:00"
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
Further Python packages can be included in the same manner. After including this file the Binder compiled with no problems, although it did take quite a long time! The link to Binder for the entire repo can be found here:  [![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/ncalvertuk/DoingDataScienceNbs_Julia/master)

Links to each individual notebook are also given at the beginning of each section below.

### Chapter 2 - Exploratory Data Analysis

Binder Link: [![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/ncalvertuk/DoingDataScienceNbs_Julia/master?filepath=notebooks%2FChapter2_RollingSales.ipynb)
__Note:__ Prior to running the first cell you have to ensure that the Kernel is set to Julia rather than Python.

The first exercise in the book is to perform some basic Exploratory Data Analysis(EDA) on New York Rolling Sales data containing housing sales data over a 12 month period. The data can be downloaded [here](https://github.com/oreillymedia/doing_data_science) and specifically I will be briefly looking at the `rollingsales_manhattan.xls` data.. I imported the data using [QueryVerse.jl](https://github.com/queryverse/Queryverse.jl) which is a meta-package containing a number of packages useful for data analysis. I have some experience of using Pandas in Python and there is a reasonable amount of overlap with the functionality used by QueryVerse so that made things a little bit easier when getting started.

The first thing I noticed when exploring the data was that ```DataFrames``` uses ```first(n)``` rather than ```head(n)``` to print the first n rows of the dataframe (you can use ```head``` however this is now deprecated).

One useful thing I learnt was how to use the Data Pipeline method in ```Query``` using the ```|>``` operator, allowing for a number of operators to be performed sequentially on the dataframe. This includes filtering, grouping, mutation, mapping and more - the docs can be found [here](https://www.queryverse.org/Query.jl/stable/standalonequerycommands/). An example of this is plotting the proportion of properties that were sold (i.e. a sale price of greater than $0) in each neighbourhood. I achieved this using the following
```
D |> @groupby(_.NEIGHBORHOOD) |> @map({Key=key(_), Count=length(_),PropSold = sum(_.SALEPRICE .>0 )./length(_)}) |> @orderby_descending(_.PropSold) |> @vlplot(:bar,x={:Key,sort="-y"},y={:PropSold})
```
by grouping the data by neighbourhood, creating a new column which contains the proportion of properties sold in that neighbourhood, ordering them, and plotting them as a bar chart. The ```@orderby_descending(_.PropSold)``` command is not required for the plotting as the ```x={:Key,sort="-y"}``` ensures the data is ordered in the plot, however I kept it in case I wanted to print it out to a line when testing.

The rest of the notebook explores the sale prices of the properties for different property types across time and neighbourhood - In particular it looks at Family Homes and Apartments. When looking at the data there was some clear differences between neighbourhoods, for example the median sale price in Harlem Central was $383 per sqf for a family home, compared with $2177 per sqf in Greenwich Village West. There did not appear to be any change of price over time (only 12 months of data was provided) however there was a large spike in sales in December which can be seen in the plot below. I was unsure why this might be the case, I was wondering if some of the dates had been automatically set to the end of the year but this did not appear to be the case. Perhaps there is some tax reason for this? Check out the notebook for my analysis at the link at the start of this section.

![SalesPerMonth](/images/SalesPerMonth.svg)

### Chapter 3 - Basic Linear Regression
Binder Link: [![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/ncalvertuk/DoingDataScienceNbs_Julia/master?filepath=notebooks%2FChapter3_Regression.ipynb)
__Note:__ Prior to running the first cell you have to ensure that the Kernel is set to Julia rather than Python.

The second exercise I attempted was the basic linear regression exercise in Chapter 3 of the book. The outline was to simulate some data, add noise and then try to recover the coefficients using linear regression.

The exercise starts by generating a single independent variable and a single dependent variable and performing a linear fit to show that the fit coefficients are returned almost exactly when Gaussian noise is added to the variables, before simulating different distribution and increasing the number of variables. I also split the datasets into training and test datasets of varying sizes to explore how the Mean Squared Error(MSE) varies with the size of the sets. I performed the fitting using the ```GLM``` [package](https://github.com/JuliaStats/GLM.jl) which was straightforward, a linear fit was performed using:
```
Data = DataFrame(X = x,Y = y);
model = lm(@formula(Y ~ X),Data)
```

One of the main learning points for me was to explore different plotting techniques and how to generate subplots. One of the major challenges was to generate a 4x4 grid of plots, where the plots on the diagonal contained histograms of the single variables and the plots off the diagonal were scatter plots showing the relationship between each pair of variables. When I was writing the notebook I was unable to do this using the standard functionality as it would only let me input up to 10 plots in a single subplot, so I delved further into the docs and found out how to generate my own [userplot](https://docs.juliaplots.org/latest/recipes/). To do this I first define a new ```userplot```  and define a recipe for my new type of plot. The recipe contains information about the layout of the plots and what type of plots they will be. I was able to do this for my 4x4 case, however ideally I'd like to generalise this to any number of variables. This is on my todo list! For now, I've included my code below and the image below that as well.
```
@userplot CompPlots_4x4

@recipe function f(h::CompPlots_4x4)
  if length(h.args) != 5 || !(typeof(h.args[1]) <: AbstractVector) ||
        !(typeof(h.args[2]) <: AbstractVector)||
        !(typeof(h.args[3]) <: AbstractVector)||
        !(typeof(h.args[4]) <: AbstractVector)
        error("CompPlots_4x4 should be given four vectors.  Got: $(typeof(h.args))")
    end
      x1,x2,x3,x4,labs = h.args
      legend := false
    link := :none
    framestyle := [:axes :axes :axes :axes :axes :axes :axes :axes :axes :axes :axes :axes :axes :axes :axes :axes]
    grid := true
      layout := @layout [hist scatter scatter scatter
    scatter hist scatter scatter
    scatter scatter hist scatter
    scatter scatter scatter hist]
      # first histo
    @series begin
        seriestype := :histogram
        subplot := 1
        title := string(labs[1], " Hist")
        x1
    end
      # second histo
    @series begin
        seriestype := :histogram
        subplot := 6
        title := string(labs[2], " Hist")
        x2
    end
      # third histo
    @series begin
        seriestype := :histogram
        subplot := 11
        title := string(labs[3], " Hist")
        x3
    end
      # fourth histo
    @series begin
        seriestype := :histogram
        subplot := 16
        title := string(labs[4], " Hist")
        x4
    end
       # x1-x2 scatter
    @series begin
        seriestype := :scatter
        subplot := 2
        title := string(labs[1]," - ",labs[2])
        x1,x2
    end
      # x1-x3 scatter
    @series begin
        seriestype := :scatter
        subplot := 3
        title := string(labs[1]," - ",labs[3])
        x1,x3
    end
      # x1-x4 scatter
    @series begin
        seriestype := :scatter
        subplot := 4
        title := string(labs[1]," - ",labs[4])
        x1,x4
    end
      
      # x2-x1 scatter
    @series begin
        seriestype := :scatter
        subplot := 5
        title := string(labs[2]," - ",labs[1])
        x2,x1
    end
      # x2-x3 scatter
    @series begin
        seriestype := :scatter
        subplot := 7
        title := string(labs[2]," - ",labs[3])
        x2,x3
    end
      # x2-x4 scatter
    @series begin
        seriestype := :scatter
        subplot := 8
        title := string(labs[2]," - ",labs[4])
        x2,x4
    end
      # x3-x1 scatter
    @series begin
        seriestype := :scatter
        subplot := 9
        title := string(labs[3]," - ",labs[1])
        x3,x1
    end
      # x3-x2 scatter
    @series begin
        seriestype := :scatter
        subplot := 10
        title := string(labs[3]," - ",labs[2])
        x3,x2
    end
      # x3-x4 scatter
    @series begin
        seriestype := :scatter
        subplot := 12
        title := string(labs[3]," - ",labs[4])
        x3,x4
    end
      # x4-x1 scatter
    @series begin
        seriestype := :scatter
        subplot := 13   
        title := string(labs[4]," - ",labs[1])
        x4,x1
    end
      # x4-x2 scatter
    @series begin
        seriestype := :scatter
        subplot := 14
        title := string(labs[4]," - ",labs[2])
        x4,x2
    end
      # x4-x3 scatter
    @series begin
        seriestype := :scatter
        subplot := 15
        title := string(labs[4]," - ",labs[3])
        x4,x3
    end
end
```
![4x4Plot](/images/4x4Plot.png)
