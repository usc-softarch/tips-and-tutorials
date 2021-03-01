# ARCADE

Most of what you'll need for ARCADE can probably be found in the manual, located [here](https://tiny.cc/arcademanual).

## Summary of ACDC

ACDC is a pattern-based architectural recovery technique: it takes a list of system entities and applies a sequence of matching patterns to identify structural clusters between these entities. The original publication of ACDC lists a good number of patterns as an "algorithm", but the implementation only actually has three meaningful patterns: Headers, SubGraphs, and OrphanAdoption. The implementation is also easily extensible (although, to my knowledge, no one ever actually extended it), meaning anyone could independently program a new pattern and add it to the execution sequence. For you, this means that testing individual patterns might be desirable, but since ACDC has been around for a good 20 years and no one's ever tried messing with the patterns, it should be safe to create a single test encompassing all of them, and consider them as an "unit", rather than subcomponents.

First, ACDC runs the Headers, or BodyHeader, pattern. This pattern was designed with C++ in mind, and what it does is essentially attempt to join header (.h) and body (.cpp) files that pertain to the same entity. The assumption here is that a "Body-Header" pair is inherently structurally cohesive, and so considering them to be an unit is reasonable. I am not sure that the algorithm, as it is written currently, actually has any bearing on systems in any language other than C or C++, but it might be worth examining the code to make sure.

The second pattern run by ACDC is called SubGraph, and it attempts to identify subgraphs from the overall dependency graph of the system, in order to minimize coupling and maximize cohesion. The metric used here is to minimize "edges" between "clusters", and maximize them within each cluster. To do this, ACDC must have a list of the dependencies for each entity in the system, and it gets that from the fact extractors. The fact extractors are run before any of the patterns are executed, and fed into ACDC's recovery component when it begins, but the "facts" (dependency graph) only start being used during this phase. The SubGraph pattern is actually a pretty basic Subgraph Dominator algorithm, so I won't go into detail here; if you need help with that, I'm sure I can find some external material that'll do a much better job explaining than I would!

Finally, the third pattern is OrphanAdoption. Originally this pattern was meant to deal with incrementally adding new entities to an existing architecture, so that these new "orphans" could be added to the existing clusters. In the implementation, however, what happens is that sometimes some entities aren't covered by the previous patterns, and are left as "orphans". What it does, in a very simplified explanation, is check each potential "parent" (cluster) for its "fitness" to adopt that orphan entity.

Once all three patterns are executed, ACDC has generated a set of subgraphs, or clusters, which it outputs in the format you see in the "*_clustered.rsf" output files. The files literally represent a set of clusters and the entities contained in each cluster. The naming of these clusters is a little unfortunate: since ACDC is a structural recovery algorithm, it typically uses the package structure of the system to give clusters their names, which means they aren't always very meaningful. However, in my experience, if you spend some 10-15 minutes trying to decipher why certain clusters have certain names, it's usually quite easy to figure it out.

Finally, after all of this has been done, ARCADE feeds the results of ACDC, along with the results of the fact extraction, to a smell analyzer. Because ACDC only has structural information, the smell analyzer only executes the detection algorithms for structural-type smells (hence why AcdcWithSmellDetection calls RunStructuralDetectionAlgs, rather than RunAllDetectionAlgs, which is used by ARC). The types of smell detected here, if I'm not mistaken, are specifically BDC (Brick Dependency Cycle) and BUO (Brick Use Overload). The former represents any large dependency cycle between clusters, and the latter represents any cluster that is involved in too many "links", or edges in the graph. Whether a dependency cycle is "large" or a cluster has "too many links" is determined in one of two ways: 1) a statistical method is applied to determine a mean and standard deviation, wherein a smell is detected if a BDC is larger than the mean + stdev, or a BUO is involved in mean + stdev links, or; 2) an arbitrary value is given by ARCADE's user which they, as the system's engineer, consider to be sufficiently excessive to denominate it as a smell. Currently, DetectBuo is implemented using the first method, and DetectBdc is implemented using the second.

ARCADE IS able to detect some other kinds of structural smells from ACDC's output, but these are detected using a separate Analyzer component (you can see more about this in section 5 of the manual). These smells, while inherently structural, depend on evolutionary data which the fact extractor and ACDC itself don't output on their own (I mentioned this to you in our meeting, how some types of smells require data from versioning repositories). For now, you don't need to worry about these; we're concerned with making sure the ArchSmellAnalyzer component is properly tested, and later on we'll move to the other Analyzer components.

In summary: ACDC is a clustering algorithm that groups implementation entities together based on patterns, of which three are implemented in ARCADE. These patterns require information about the dependencies between implementation entities, which is acquired by the fact extraction component. Finally, the data output by both Fact Extraction and ACDC's Recovery is fed into a Smell Analyzer component, which uses that to detect two types of structural smells. The tests should cover each of these components, bearing in mind that the Fact Extractor is used universally by other recovery techniques, but the Smell Analyzer runs differently depending on the recovery technique that was used.