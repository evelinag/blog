@{
    Layout = "post";
    Title = "Comparing F# and C# with dependency networks";
    Date = "2014-06-9T11:45:25";
    Tags = "F#, C#, networks, dependency, motifs";
    Description = "It is difficult to objectively compare different programming languages because " +
		"terms like 'clarity' or 'maintainability' are vague and subjective. " + 
		"What if we used some tools from network science?";
	Image = "http://evelinag.com/blog/2014/06-09-comparing-dependency-networks/motifs-small.png";
	Url = "http://evelinag.com/blog/2014/06-09-comparing-dependency-networks/index.html";
}

<div class="row">
<div class="medium-8 columns">

Fans of different programming languages always argue about benefits of their language
of choice. It is difficult to use objective criteria in a debate like this. Terms like 
'clarity' or 'maintainability' are too vague and subjective. What if we used some tools from network 
science to compare projects written in different languages?

In this blog post I use network analysis to investigate how complex dependency graphs are and
if they differ between C# and F#.
It turns out that F# and C# 
dependency networks have quite different structures and use 
different local network patterns.
For example, I'll describe specific types of cyclic dependencies 
that frequently appear only in C# projects. 

</div>
<div class="medium-4 columns">

![Examples of motifs on 3 and 4 nodes](http://evelinag.com/blog/2014/06-09-comparing-dependency-networks/Motifs.png "Examples of patterns (motifs) on 3 and 4 nodes")

</div>
</div>

<!-- more -->

This blog post is an addition to an excellent article by Scott Wlaschin on 
modularity and cyclic dependencies in real-world F# and C# projects
[Cycles and modularity in the wild](http://fsharpforfunandprofit.com/posts/cycles-and-modularity-in-the-wild/).
I wanted to look at the same data that Scott extracted in his 
article but from network analysis perspective. 

Dependency networks
-------------------------------

For my analysis, I extracted dependency networks from 40 different projects, half of
them written in C# and half of them in F#. All the networks come from 
compiled assemblies that can be downloaded through NuGet. 
I used similar method as Scott for my analysis. If you want more details, head
over to [F# for fun and profit](http://fsharpforfunandprofit.com/posts/cycles-and-modularity-in-the-wild/)
for a more detailed description. I'll give just a brief overview here.

#### Structure of a dependency network

A dependency network is formed by nodes and oriented links between them.

Nodes in the dependency network are formed by

- Classes in C#
- Modules in F#

Compiler turns F# modules into static classes so the two definitions should
be roughly comparable, at least on the CIL level. 
Both types of nodes represent only top-level classes
and modules, nested types are incorporated into their parent class or module.
The networks analyzed in this blog post contain all the classes and modules from
each project, not just the public ones.

![Link from A to B](linkAB.png)

Links between the nodes represent dependencies. 
There is a link from A to B in the network if:

- Class B inherits from class A or implements interface A.
- Function in B calls a function or method from A.
- Field, property, method or function in B references A
as a parameter or as a return type.

Note that I switched direction of dependency arrows
in the network compared to the original article at 
[F# for fun and profit](http://fsharpforfunandprofit.com/posts/cycles-and-modularity-in-the-wild/).
Now links represent the direction in which information is passed between
nodes. This definition corresponds more to the logic of information flow in a program. 
For example if there is a bug in a function, it will propagate along the dependency
arrows into all nodes that call the function. 

#### Projects under the spotlight

I expanded the list of analysed projects compared to the original analysis 
at [F# for fun and profit](http://fsharpforfunandprofit.com/posts/cycles-and-modularity-in-the-wild/).
Again, the projects are not directly comparable in general. 
I hope that by using more projects, data get averaged and we can
get a bigger picture out of them. 
The results are still biased from the small sample size though. 

Here are the 40 projects (individual dlls) that got included 
into the analysis (in no particular order):

###### C# projects:
Antlr, AutoMapper, Castle, elmah, EntityFramework, FParsecCS, log4net, MathNet.Numerics, SignalR, Bcl.Runtime, Owin, Cecil, Moq, Nancy, Newtonsoft.Json, Nuget, NUnit, SpecFlow, xunit, YamlDotNet

###### F# projects:
canopy, Deedle, Fake, Foq, FParsecFS, FsCheck, FSharp.Compiler.Service, FSharp.Core, FSharp.Data, FSharp.Data.Twitter, FSharpx, FsPowerPack, FsSql, FsUnit, FsYaml, Storm, TickSpec, WebSharper, WebSharper.Core, WebSharper.Html

Network statistics
-------------------------------

The networks extracted from compiled project dlls have very different sizes.
The following chart shows the number of nodes (classes or modules) and number 
of dependencies in each project. The axes in the figure are logarithmic so that we
can put data with different scales into one picture.

![Number of nodes vs. number of dependencies](networkSize.png "Number of nodes vs. number of dependencies")

Projects written in F# seem to be generally smaller. On the other hand, C# projects tend 
to be larger both in the number of nodes and number of dependencies. 
It is interesting that the plot looks approximately like a straight line. This indicates
a power law relation between the number of nodes and links both in F# and C# projects.  

Next question we might ask is how complex are the networks? One measure of 
complexity in code depedendency networks might be how many dependencies are chained
together in the graph. Long chains of dependencies increase complexity of code.
For example, bugs that get propagated through a long dependency path might affect
a large part of the whole project. 
A standard measure for this is the network diameter. It is computed by looking at shortest
paths between all possible pairs of nodes in a network. 
Diameter is defined as the length of the longest
of these paths. For diameters in C# and F# projects we get these box plots:

![Network diameters](diameters.png)

Diameter of analyzed C# projects is on average more than double the diameter 
of F# projects. Diameters are actually roughly proportional to the number of nodes and 
links in each network. Because C# has larger networks, diameters expand as well. 

One aspect where F# and C# projects differ dramatically is the number of 
isolated nodes. These represent standalone modules or classes that do not have any
dependency within the project. Here is a box plot showing the proportion of 
 standalone nodes.

![Isolated nodes](isolatedNodes.png)

Isolated nodes appear much more frequently
in F# projects than in C# projects. This is probably an effect of different 
programming paradigms. Object-oriented language like C# might require
the programmer to introduce more dependencies into the code. As a result,
functional F# has cleaner modularity than C# on average. 

Below are images of networks from two different projects as an example. 
There is `Yaml.NET` on the left and `FSharp.Core` on the right. The two projects
are not comparable in terms of their scope. However, their networks have 
roughly the same number of nodes and similar diameter. 

`FSharp.Core` has more isolated nodes that do not have any dependencies
within the project which seems to be typical for F# projects.
The densely connected core of the project is much smaller than in C#. 
The two networks are meant just as an illustration of typical features
of C# and F# dependency networks. 

<div class="row">
<div class="large-6 columns">

![Yaml.NET network](csNetwork-YamlDotNet.png "Yaml.NET network")

</div>
<div class="large-6 columns">

![FSharp.Core network](fsNetwork-FSharp.Core.png "FSharp.Core network")

</div>
</div>

Here are the detailed numbers for the analyzed projects:

#### C# code statistics

<div class="smallTable">

| Project | Code size | Number of nodes | Number of links | Isolated nodes | Diameter |
|---------|:---------:|:---------------:|:---------------:|:--------------:|:--------:|
Antlr|34344|91|257|8.8 %|5
AutoMapper|34793|152|549|5.3 %|8
Castle|112538|430|1766|5.6 %|8
elmah|43728|116|300|7.8 %|5
EntityFramework|1144189|1679|11671|4.7 %|16
FParsecCS|32230|35|48|14.3 %|3
log4net|102651|227|746|0.9 %|10
MathNet.Numerics|492095|342|1285|5.6 %|8
SignalR|63690|221|735|6.8 %|11
Bcl.Runtime|73|8|2|62.5 %|1
Owin|13376|55|98|10.9 %|7
Cecil|100650|240|1145|5.0 %|8
Moq|158417|541|1536|11.1 %|14
Nancy|130818|369|1205|5.4 %|12
Newtonsoft.Json|157716|237|1005|4.6 %|13
Nuget|101586|229|943|2.2 %|10
NUnit|45873|183|505|14.2 %|7
SpecFlow|41187|242|578|2.5 %|7
xunit|14590|72|209|1.4 %|7
YamlDotNet|42372|161|550|2.5 %|7

</div>

#### F# code statistics

<div class="smallTable">

| Project | Code size | Number of nodes | Number of links | Isolated nodes | Diameter |
|---------|:---------:|:---------------:|:---------------:|:--------------:|:--------:|
canopy|23630|11|12|27.3 %|2
Deedle|122918|95|249|18.9 %|5
Fake|1395|3|1|33.3 %|1
Foq|38532|40|75|5.0 %|3
FParsecFS|45946|6|4|33.3 %|2
FsCheck|76418|54|103|16.7 %|5
FSharp.Compiler.Service|110523|42|23|50.0 %|2
FSharp.Core|206348|154|287|40.3 %|6
FSharp.Data|135001|94|173|8.5 %|6
FSharp.Data.Twitter|10372|20|29|25.0 %|3
FSharpx|290577|175|77|56.0 %|2
FsPowerPack|102878|93|68|46.2 %|4
FsSql|15311|13|14|0.0 %|4
FsUnit|1580|2|0|100.0 %|0
FsYaml|14573|8|10|12.5 %|3
Storm|55072|67|195|3.0 %|5
TickSpec|27970|34|48|5.9 %|3
WebSharper|43747|56|22|57.1 %|2
WebSharper.Core|83201|12|13|25.0 %|2
WebSharper.Html|14152|19|37|10.5 %|2

</div>


Network motifs
-------------------------------

We looked at some global properties of dependency networks, now we turn to 
explore more local features. Motifs are small reccurring patterns of 
links between nodes that appear in
real-life networks. For example, there has been a lot of research on motifs
in gene regulatory networks and their functional meaning. We can apply the same approach
to our dependency networks to see if there are any typical patterns. 

Motif finding in general networks is computationally hard because it involves
identifying graph isomorphisms. The larger the motif, the harder it is to find it
in a network. In this analysis, I looked only at motifs
on three and four nodes. I used the
`igraph` package in R with F# RProvider. 
The motif finding function from `igraph` counts the number of times each possible motif
on three or four nodes appears in a given network.

### Motifs on 3 nodes

![Motifs on 3 nodes](Motifs_3nodes.png)

There are 13 possible motifs on three nodes. I computed how many times each of these 
motifs appears in the project networks. Because
each network has different size, the counts were normalized with respect to the total number
of motifs in each network. The following bar plot compares average frequencies of all the motifs.

#### Average motif profiles on 3 nodes in C# and F# projects
![Motif profiles in C# and F# projects](fsVsCsMotifs_3nodes.png)

Motifs number 1, 2, 4 and 5 are the most common in both C# and F# projects. 
They seem to differ only in how often each motif appears. The results seem 
quite intuitive because these motifs look like standard patterns that
would be expected in a software project.
The bar plot shows only the average frequencies
and variance between individual projects is quite high. 
Summary of results for each project is available [here](https://github.com/evelinag/Projects/blob/master/CodeNetworks/code/data/motifs_3nodes.csv).

#### Motifs that are C#-specific

What is interesting is that there are several motifs that appear in 
many C# projects but they are not in any of the analyzed F# projects.
Here they are: 

![C# only motifs](csharpOnlyMotifs_3nodes.png)

Additionally motif number 12 appears just once in `FSharp.Core` and nowhere else
among the F# projects. 
What all these motifs have in common is that they all contain 
cyclic dependencies. Scott Wlaschin wrote
a nice blog post on why [cyclic dependencies are evil](http://fsharpforfunandprofit.com/posts/cyclic-dependencies/). 
Simply said, they add complexity, mess up structure of code
and complicate maintainability. 
So, this is how the evil cyclic dependencies look in real-world projects. 
Especially motif number 13 with full connectivity looks like 
something that should be avoided. How frequent are these cyclic motifs?

| Motif | Number of projects |
|-------|:------------------:|
8 | 13
9 | 9
10 | 14
13 | 4

The table shows how many projects contain each of the C#-specific motifs. 
Motifs number 8 and 10 are in majority of the analysed networks which means they are
quite widespread. Fortunately, the most entagled motif number 13 is the least common one and
occurs only in 4 projects. There are no motifs that would appear only in F# projects. 

### Motifs on 4 nodes

I will not give the full analysis of motifs on 4 nodes because there are 199
of them. However, there are a few interesting things to point out. Again, F# and C# share
the most common motifs which look like patterns that we would expect to see:

![Most common motifs on 4 nodes](Motifs_4nodes.png "Most common motifs on 4 nodes")

And again, we have some motifs that appear exclusively in C# projects, this time we have 129
motifs that are C#-only. There are no motifs that would be just in F# projects. 
These are the most common C#-specific ones:

![C#-specific motifs on 4 nodes](csharpOnlyMotifs_4nodes.png "Most common C#-specific motifs on 4 nodes")

These motifs are also quite widespread.  
The first one appears in 14 projects, the rest of them in 13 projects.
Finally, what about the most complex motif on 4 nodes?

![The most complex motif on 4 nodes](ugliest_4nodes.png "The most complex motif on 4 nodes")

It turns out that this motif appears in 3 of the C# projects (specifically in EntityFramework, 
Mono.Cecil and Newtonsoft.Json). This pattern looks like quite a poor design choice. 

Explore motifs in your projects
---------------------------------------

If you want to find what is the motif profile of your own project, this
[FsLab Journal](journal/analyseCodeNetworks.html) shows how to run the analysis. 
Source code from the Journal is available [here](https://github.com/evelinag/Projects/tree/master/CodeNetworks/networksJournal).
You can also download the full source code that replicates results from this blog post 
from my [GitHub page](https://github.com/evelinag/Projects/tree/master/CodeNetworks/code).

Summary
------------------------------

In this blog post, I looked at dependency networks in several 
C# and F# projects. The analysis shows some similarities and differences
between the two programming languages. In general, C# projects tend 
to be larger, with more classes and dependencies. They also have
longer chains of dependencies on average. Real world F# projects
are smaller with cleaner modularity.

I also described recurring patterns (motifs) that appear in dependency networks. 
The most common motifs are similar in C# and F# projects. However, most of C# 
projects contain motifs with complicated cyclic dependencies that do not appear
in F# at all. Cyclic dependencies in general complicate the code and obscure
dependency structure. 

This analysis is still very limited. For example we can debate if  
the dependency networks are well defined with respect to both languages
to be truly comparable. Nevertheless, it seems that this type of analysis 
can reveal some aspects of dependency networks.

In general, it seems that most C# projects would be harder to maintain because of all the
cyclic dependencies and more complex structure overall. The question is whether
it is a feature of the language itself that encourages programmers 
to create more complex systems. 

I also presented a poster on this topic at 
[Cambridge Networks Day 2014](http://www.cnn.group.cam.ac.uk/cambridge-networks-day).

<iframe src="http://wl.figshare.com/articles/1036337/embed?show_title=1" width="568" height="525" frameborder="0"></iframe>

<div class="edit">
Correction 13/6/2014: Relation between number of nodes and number of links is a power law function.
</div>