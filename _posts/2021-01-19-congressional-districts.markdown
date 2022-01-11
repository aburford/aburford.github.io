---
layout: post
title:  U.S. Congressional Redistricting Explorer
date:   2021-01-19 18:05:55 +0300
image:  districting.jpg
tags:   [Python, Java, Javascript]
github: https://github.com/mfoydl/US_Congressional_Districting
---

U.S. Congressional Redistricting Explorer is a website created in the capstone
software engineering course at Stony Brook. I worked in a group of 4 to create
a web interface allowing the user to filter and analyze a large set of randomly
generated U.S. congressional districting plans. My main role was to synthesize
and manipulate publicly available geospatial data and modify graph algorithms
to randomly generate feasible districting plans. I also helped with full-stack
web development: designing and debugging the Java backend, linking together the
JavaScript frontend, and fixing some UI issues.

### Background

Every state in the U.S. is geographically divided up into a set of precincts.
This set of precincts is then partitioned into contiguous sets of precincts
called districts. Each way of partitioning the precincts is referred to as a
districting. This process of assigning precincts to districts is called
redistricting. During an election, the candidate with the majority of votes in
a single district is said to win that district, and the candidate who has won
the most districts is the overall winner. However, this means that different
districtings can lead to different election results even if everyone voted the
same. It is typically the responsibility of the incumbent political party to perform
redistricting, so naturally politicians draw the borders to maximize their
chances of remaining in office. The practice of drawing unfair districts that
favor one political party is called
[Gerrymandering](https://en.wikipedia.org/wiki/Gerrymandering). Gerrymandering
is difficult to fight due to lack of a rigorous definition for what counts as
gerrymandering. The premise of this project is that randomly sampling from the
space of all possible districtings and measuring characteristics of each
districting can lead to statistical distributions for the "average"
districting. Districtings that stray far from this average are more likely
to be gerrymandered.

### Generating Random Districtings

There do exist a few formal requirements for what constitutes a valid
districting. For example, all of the precincts within a single district must be
contiguous. Additionally, there is a population equality requirement which
specifies a maximum percentage difference in population between any two
districts in a districting. Generating districtings under these constraints
requires gathering some publicly available geospatial data.  For this project,
we were assigned three U.S. states (Alabama, Arizona, and Michigan). The public
geospatial precinct data for these states did not contain population data so I
had to find population data from the 2010 census. This gave me the population
within each consensus block, but these census block regions have no geographic
relation to precinct regions. I used the computational geometry library
[shapely](https://shapely.readthedocs.io/en/stable/project.html) to assign the
population data from each census block to overlapping precincts in order to
come up with population estimates for each precinct.

I then analyzed the precinct geospatial data to compute an adjacency graph for
all of the precincts. The problem of redistricting can then be reduced to
partitioning this graph into a forest of spanning trees such that the
population of each spanning tree satisfies the population equality requirement.
The professor provided us with Python code that generates such spanning tree
forests, but he wasn't aware that this algorithm was asymptotically
inefficient, requiring $$O(n^2)$$ time to check the population equality
constraint. I came up with my own modified implementation that cuts the
population equality check down to $$O(n)$$ time, making the $$O(n \log n)$$
spanning tree generation step the bottleneck.

I then computed a variety of characteristics for each districting such as
compactness score, split county score, and deviation from enacted districting.
Compactness is a measure of how well rounded each district in a districting is.
Districts that are long and skinny, branching out to include specific
precincts, tend to be a sign of gerrymandering, so it is desirable for districts
to be more compact. One way to compute this is to count the precincts on the
border of a district and divide by the number of precincts within the district.
This simply required iterating through the precincts in the adjacency graph I already computed to
check for neighboring precincts not in the current district. The split county
score counts how many counties are split into multiple districts. It is
generally desirable to have every county contained in exactly one district. I
checked for split counties using the same shapely library used earlier for
population estimates and publicly available geospatial data for counties.

I determined the deviation from the enacted districting using a technique
called the Gill construct, which we were provided a research paper about. This
involves finding the optimal matching of districts in a given districting to
the currently enacted districts in that state such that geographic overlap
between district pairs is maximized. Less total overlapping area between paired districts corresponds
with greater deviation from the enacted districting. In order to compute this,
a bipartite graph is constructed with one partition containing the districts of
the generated districting and the other partition containing the enacted
districts. An edge is drawn between a district in the generated districting and
an enacted district if they overlap at all. The weight of this edge is set to
the area of overlap, reducing the problem to the assignment problem. I
implemented this using shapely and [SciPy](https://scipy.org).
Finally, I generated geospatial data for all of the districtings by stitching
together all of the precincts in each district to create a single geospatial
polygon for each district. District polygons were aggregated to create GeoJSON
files for each districting.

### The Website

I worked alone for the random districting generation process, but collaborated
with my 3 team members for the frontend website and backend server. We used
several Javascript libraries such as Bootstrap, Leaflet, and jQuery. The
backend server was written in Java with the help of
[JAX-RS](https://www.oracle.com/technical-resources/articles/java/jax-rs.html).
I designed the class diagrams and activity diagrams for the backend server and
helped my partner implement everything.
