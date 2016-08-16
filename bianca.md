---
title: "BIANCA: Preventing Bug Insertion at Commit-Time Using Dependency Analysis and Clone Detection"
bibliography: config/library.bib
abstract: abstract goes here
author:
- name: Mathieu Nayrolles,  Abdelwahab Hamou-Lhadj
  affiliation: SBA Lab, ECE Dept, Concordia University
  location: Montréal, QC, Canada
  email: \{mathieu.nayrolles, wahab.hamou-lhadj\}\@concordia.ca
- name: Emad Shihab
  affiliation: DAS Lab, CSE Dept, Concordia University
  location: Montréal, QC, Canada
  email: eshihab@cse.concordia.ca
csl: config/ieee.csl
classoption: conference
keyword: 
- Software Analytics
- Software Metrics
- Risky Software Commits
- Bug Prediction
- Clone Detection
- Software Maintenance
---

# Introduction

Software maintenance activities such as debugging and feature enhancement are known to be challenging and costly [@Pressman2005].
Studies have shown that the cost of software maintenance can reach up to 70\% of the overall cost of the software development life cycle [@HealthSocial2002].
Much of this is attributable to several factors including the increase in software complexity, the lack of traceability between the various artifacts of the software development process, the lack of proper documentation,  and the unavailability of the original developers of the systems.

Research in software maintenance has evolved over the years to include areas like mining bug repositories, bug analysis, prevention, and reproduction. The ultimate goal is to develop techniques and tools to help software developers detect, correct, and prevent bugs in an effective and efficient manner. 
Despite the recent advances in the field, the literature shows that many existing software maintenance tools have yet to be adopted by industry [@Lewis2013; @Foss2015; @Layman2007; @Ayewah2007; @Ayewah2008; @Johnson2013; @Norman2013; @Hovemeyer2004; @Lopez2011]. 
We believe that this is caused by the following factors. (a) Integration with the developer's workflow: Most existing maintenance tools ([@Kim2006a; @Ayewah2008b;  @Findbugs2015; @Palma; @Nayrolles2013d; @Nayrolles2013a; @Nayrolles2015a] are some noticeable examples) are not integrated well with the workflow of software developers (i.e., coding, testing, debugging, committing). (b) Corrective actions: The outcome of these tools does not always lead to corrective actions that the developers can implement. 
Most of these tools return several results that are often difficult to interpret by developers. Take, for example, FindBugs [@Hovemeyer2004], a popular bug detection tool. 
This tool detects hundreds of bug signatures and reports them using an abbreviated code such as `CO_COMPARETO_INCORRECT_FLOATING`. 
Using this code, developers can browse the FindBug's dictionary and find the corresponding definition _"This method compares double or float values using a pattern like this: `val1 > val2 ? 1 : val1 < val2 ? -1 : 0`"_. 
While the detection of this bug pattern is accurate, the tool does not propose any corrective actions to the developers that can help them fix the problem and developers decide simply to ignore them [@Arai2014; @Kim2007b; @Kim2007c; @Ayewah2010; @Shen2011]. 
(c) Leverage of historical data: These tools do not leverage cross-project knowledge.  
For defect prevention, for example, the state of the art approaches consists of adapting statistical models built for one project to another project [@Lo2013; @Nam2013].  
In addition, Lewis _et al._ [@Lewis2013] and Johnson _et al._ [@Johnson2013] argued that, approaches based solely on statistical models are perceived by developers as black box solutions. 
Developers are less likely to trust the output of these tools.

In this paper, we propose an approach named BIANCA (Bug Insertion ANticipation by Clone Analysis at commit time) that assesses these challenges by using dependency analysis and clone detection to prevent bug insertion at commit-time. 
More specifically, BIANCA lies on the idea that complex software systems are not monolithic; they have dependencies.
Software systems sharing the same dependencies are likely to perform related tasks and, therefore, share misunderstandings leading to defect introduction.
For example, Apache BatchEE [@TheApacheSoftwareFoundation2015] and GraphWalker [@Graphwalker2016] both depend on JUNG (Java Universal Network/Graph Framework) [@JoshuaOMadadhain]. 
BatchEE provides an implementation of the jsr-352 (Batch Applications for the Java Platform) specification [@ChrisVignola2014] while GraphWalker is an open source model-based testing tool for test automation.
These two systems are designed for different purposes. BatchEE is used  to do batch processing in Java, whereas GraphWalker is used to design unit tests using a graph representation of code. Nevertheless, Apache BatchEE and GraphWalker both rely on JUNG. The developers of these projects made similar mistakes while building upon JUNG. The issue reports Apache BatchEE #69 and  GraphWalker #44 indicate that the developers of these projects made similar mistakes when using the graph visualization of JUNG.

BIANCA integrates itself seamlessly with developers' workflow by acting at commit-time, propose concrete actions to implement to avoid bug insertion and leverage cross-project historical data without statistical models.

The remaining of this paper is organized as follows. 
In section \ref{sec:relwork}, we present works related to ours.
Sections \ref{sec:bianca} and  \ref{sec:exp} present the BIANCA approach and its validation, respectively.
Then, Sections \ref{sec:threats} and \ref{sec:conclusion} assess the threats to validity and present a conclusion accompanied with future work, respectively.

# Related Work {#sec:relwork}

Predicting crashes, faults, and bugs is very popular research area. The main goal of existing studies is to save on manpower when dealing with bugs and crashes. There are two distinct trends in crash, fault and bug prediction: History analysis and current version analysis.

In the history analysis, researchers extract and interpret information from the system. 
The idea being that the files or locations that are the most frequently changed are more likely to contain a bug. 
Additionally, some of these approaches also assume that locations linked to a previous bug are likely to be linked to a bug in the future. 
On the other hand, approaches using only the current version to predict bugs assume that the current version, i.e., its design, call graph, quality metrics and more, will trigger the appearance of the bug in the future. Consequently, they do no require the history and only need the current source-code.

In the remaining of this section, we describe approaches belonging
to the two families.
Then, we present relevant clones detection approaches as BIANCA uses such an approach to achieve bug prevention at commit-time.

## Change logs approaches {#sec:change-logs-approaches}

Change logs based approaches rely on mining the historical data of the application and more particularly, the source code *diffs*. 
A source code *diffs* contains two versions of the same code in one file. 
Indeed, it contains the lines of code that have been deleted and the one that has been added.

Naggapan *et al.* studied the churns metric and how it can be connected to the apparition of new defects in complex software systems. 
They established that relative churns are, in fact, a better metric than classical churn [@Nagappan] while studying Windows Server 2003.

Hassan interested himself with the entropy of change, i.e. how complex the change is [@Hassan2009]. 
Then, the complexity of the change, or entropy, can be used to predict bugs. 
The more complex a change is, the more likely it is to bring the defect with it.
Hassan used its entropy metric, with success, on six different systems.
Before this work, Hassan, in collaboration with Holt proposed an approach that highlights the top ten most susceptible locations to have a bug using heuristics based on *diffs* file metrics [@Hassan2005].
Moreover, their heuristics also leverage the data of the bug tracking system.
Indeed, they use the past defect location to predict new ones. The conclusion of these two approaches has been that recently modified and fixed locations where the most defect-prone compared to frequently modified ones.

Similarly to Hassan and Hold, Ostrand *et al.* predict future crash location by combining the data from changed and past defect locations [@Ostrand2005]. 
The main difference between Hassan and Hold and Ostrand *et al.* is that Ostrand *et al.* validate their approach on industrial systems as they are members of the AT&T lab while Hassan and Hold validated their approach on open-source systems. 
This proved that these metrics are relevant for open-source and industrial systems.

Kim *et al.* applied the same recipe and mined recent changes and defects with their approach named bug cache [@Kim2007a]. 
However, they are more accurate than the previous approaches at detecting defect location by taking into account that is more likely for a developer to make a change that introduces a defect when being under pressure. 
Such changes can be pushed to the revision-control system when deadlines and releases date are approaching.

## Single-version approaches

Approaches belonging to the single-version family will only consider the current version of the software at hand.
Simply put, they do not leverage the history of changes or bug reports.
Despite this fact, that one can see as a disadvantage compared to approaches that do leverage history; these approaches yield interesting results using code-based metrics.

Chidamber and Kemerer published the well-known CK metrics suite [@Chidamber1994] for object oriented designs and inspired Moha *et al.* to publish similar metrics for service-oriented programs [@Moha]. 
Another famous metric suite for assessing the quality of a given software design is Briand's coupling metrics [@Briand1999a].

The CK and Briand’s metrics suites have been used, for example, by Basili *et al.* [@Basili1996], El Emam *et al.* [@ElEmam2001], Subramanyam *et al.* [@Subramanyam2003] and Gyimothy *et al.* [@Gyimothy2005] for object-oriented designs. 
Service oriented designs have been far less studied than object oriented design as they are relatively new, but, Nayrolles *et al.* [@Nayrolles; @Nayrolles2013d], Demange *et al.* [@demange2013] and Palma *et al.* [@Palma2013] used Moha et *et al.* metric suites to detect software defects.
All these approaches, proved software metrics to be useful at detecting software fault for object oriented and service oriented designs, respectively.

Finally, Nagappan *et al.* [@Nagappan2005; @Nagappan2006] and Zimmerman [@Zimmermann2007; @Zimmermann2008] further refined metrics-based detection by using statical analysis and call-graph analysis.

While academia has published hundreds of bug prediction papers over the last decade, the developed tools and approaches fail to change developer behavior while deployed in industrial environments [@Lewis2013]. This is mainly due to the lack of actionable message, i.e. messages that provide concrete steps to resolve the problem at hand.

## Clone Detection {#sec:rel-clones}

BIANCA relies on code clone detection to perform bug prevention at commit-time. 
Consequently, we reviewed the literature of the field.
This section describes major works in clone detection.

Code clones appear when developers reuse code with little to no modification to the original code. Studies have shown that clones can account for about 7% to 50% of the code in a given software system [@Baker; @StephaneDucasse]. 
Developers often reuse code (and create clones) in their software on purpose [@Kim2005]. Nevertheless, clones are considered a bad practice in software development since they can introduce new bugs in the code [@Kapser2006; @Juergens2009; @Li2006]. 
If a bug is discovered in one segment of the code that has been copied and pasted several times, then the developers will have to remember the places where this segment has been reused to fix the bug in each place.

In the last two decades, there have been many studies and tools that aim at detecting clones.
They can be grouped into three categories.
The first category includes techniques that treat the source code as text and use transformation and normalization methods to compare various code fragments [@Johnson1994; @Johnson1993; @Cordy2011; @Roy2008].
The second category includes methods that use lexical analysis, where the source code is sliced into sequences of tokens, similar to the way a compiler operates [@Baker; @Bakera; @Baker2002; @Kamiya2002; @Li2006].
The tokens are used to compare code fragments.
Finally, syntactic analysis has also been performed where the source code is converted into trees, more particularly abstract syntax tree (AST), and then the clone detection is performed using tree matching algorithms [@Baxter1998; @Komondoor2000; @Tairas2006; @Falke2008].

Text-based techniques use the code — often raw (e.g. with comments) — and compare sequences of code (blocks) to each other to identify potential clones. Johnson was perhaps the first one to use fingerprints to detect clones [@Johnson1993; @Johnson1994]. 
Blocks of code are hashed, producing fingerprints that can be compared. 
If two blocks share the same fingerprint, they are considered as clones. Manber et al. [@Manber1994] and Ducasse et al. [@Ducasse1999] refined the fingerprint technique by using leading keywords and dot-plots.

Tree-matching and metric-based are two sub-categories of syntactic analysis for clone detection. Syntactic analysis consists of building abstract syntax trees (AST) and analyze them with a set of dedicated metrics or searching for identical sub-trees. 
Many approaches using AST have been published using sub-tree comparison including the work of Baxter et al. [@Baxter1998], Wahlert et al. [@Wahler], or more recently, the work of Jian et al. with Deckard [@Jiang2007]. 
An AST-based approach compares metrics computed on the AST, rather than the code itself, to identify clones [@Patenaude1999; @Balazinska].

Another approach to detect clones is to use static analysis and to leverage the semantics of the program to improve the detection. 
These techniques rely on program dependency graphs where nodes are statements and edges are dependencies. Then, the problem of finding clones is reduced to the problem of finding identical sub-groups in the program dependency graph. Examples of recent techniques that fall into this category are the ones presented by Krinke et al.[@Krinke2001] and Gabel et al. [@Gabel2008].

Many clone detection tools have been created using a lexical approach for clone detection. 
Here, the code is transformed into a series of tokens. 
If sub-series repeat themselves, it means that a potential clone is in the code. 
Some popular tools that use this technique include, but not limited to, Dup [@Baker], CCFinder [@Kamiya2002], and CP-Miner [@Li2006].

Furthermore, a large number of taxonomies have been published in an attempt to classify clones and ease the research on clone detection [@Mayrand1996; @Balazinska1999; @Koschke2006; @Bellon2007; @Kontogiannis; @Kapser].
Despite the particularities of each proposed taxonomy, researchers agree on the following classification. 
Type 1 clones are copy-pasted blocks of code that only differ from each other in terms of non-code artifacts such as indentation, whitespaces, comments and so on. 
Type 2 clones are blocks of code that are syntactically identical except literals, identifiers, and types that can be modified. 
Also, Type 2 clones share the particularities of Type 1 about indentation, whitespaces, and comments. Type 3 clones are similar to Type 2 clones in terms of modification of literals, identifiers, types, indentation, whitespaces, and comments but also contain added or deleted code statements. 
Finally, Type 4 are code blocks that perform the same tasks but using a completely different implementation.

BIANCA is different than the presented approach preventing the insertion of defects into the source code as it leverages cross-project historical data and is able to detect risky commit by using clone detection and dependencies analyses.

# The BIANCA Approach {#sec:bianca}

Figure \ref{fig:bianca} shows an overview of BIANCA. BIANCA (Bug Insertion ANticipation by Clone Analysis at commit time) has two parts: Offline and Online. During the offline part of BIANCA, we perform the following operations: (a) automatically extract system dependencies from software repositories to cluster them, and, (b) analyze the commits history to classify commits, link bugs introducing commits to issues and extract bug changesets.
In the online part, we compare new modifications of the code to modifications that are known to have introduced a defect in the past. This comparison is performed using clone detection.
In addition, we are able to show to the developers why their modifications are _risky_ and potentials fixes for the problem at hand.

\input{tex/approach}


The rest of this section is organized as follows: section \ref{sec:offline} describes the offline part while section \ref{sec:online} describes the online part.

## Offline {#sec:offline}

Offline refers to the fact that these processes are not interactive but follow a batch processing paradigm.
BIANCA first extracts dependencies of each repository (Section \ref{sec:dep}) and clusters repositories according to their dependencies (Section \ref{sec:clust}).
The idea of clustering project based on their dependencies is that project sharing dependencies are likely to propose related feature, and, unfortunately, related misunderstandings about their dependencies.
Then, repositories are only compared with repositories in their cluster.
In our experimentations (Section \ref{sec:exp}), we demonstrate that such a clustering performs significantly better than no clustering.

In parallel of the clustering of repositories based on their dependencies, we classify defect introducing commits (Section \ref{sec:bug-introduction}) and extract the blocks of code contained in defect introducing commits (Section \ref{sec:block-extract}).


### Building a project dependency graph {#sec:dep}

In this step, the dependencies of repositories are analysed and saved into a single no-SQL graph database. 
Graph databases use graphs structures as a way to store and query information. 
In our case, each project is a node and is connected to its dependencies. 
Dependencies can be automatically retrieved if projects use a dependency manager such as Maven. 

Figure \ref{fig:network-sample} shows a simplified view of a dependency graph for a project named \texttt{com.badlogicgames.gdx}.
As shown, \texttt{badlogicgames.gdx} depends on projects owned by the same organization (i.e., badlogicgames) and other organizations such as Google, Apple, and Github.

![Simplified Dependency Graph for \texttt{com.badlogicgames.gdx} (Zoomed from south of Figure \ref{fig:dep-graph})\label{fig:network-sample}](media/network-sample.png)
  
### Clustering projects {#sec:clust} 

The Girvan–Newman algorithm [@Girvan2002; @Newman2004] detects communities by progressively removing edges from the original network. 
The connected components of the remaining network are the communities. 
Instead of trying to construct a measure that tells which edges are the most central to communities, the Girvan–Newman algorithm focuses on edges that are most likely "between" communities.
This algorithm is highly effective at discovering community structure in both computer-generated and real-world network data.

The Girvan–Newman algorithm fits our problem as we are interested in discovering the _communities_ of repositories that depend on a similar set of dependencies.


### Identifying bug introduction {#sec:bug-introduction}

A risky commit is a commit that may introduce a bug in the future [@Kamei2013; @SunghunKim2008].
To detect risky commits, we follow four steps.
The first one consists of retrieving the commits that we know have introduced defects in the past by mining software and issue repositories. Then, we extract the blocks of code that were modified during these risky commits.
The extracted blocks form our _dataset_ of commits against which new commits will be compared.
To compare blocks from previous commits that have introduced a defect to blocks from incoming commits, we use a text-based matching technique.

To construct our _dataset_ of commits introducing defects, we use a modified version of the back-end of CommitGuru [@Rosen2015a]. Commit guru's back-end has three major components: ingestion, analysis, and prediction.
We reuse the ingestion and analysis components for BIANCA.
The ingestion component is responsible for ingesting (i.e., downloading) a given repository.
Once the repository is entirely downloaded on a local server, each commit of history is analysed and change-level metrics proposed by Kamei *et al.* are computed [@Kamei2013].
Also, commits are classified using the list of keywords proposed by Hindle *et al.* [@Hindle2008] and reported in Table \ref{tab:labels}.
After this has been completed, CommitGuru performs the SCM blame/annotate function on all modified lines of code for their corresponding files on the fixing commit's parent.
This returns the commits that previously modified them. 
These commits have introduced the bugs corrected by the fixing commit and mark them as such. 

Note that we could use a simpler and more established tool such as Relink [@Wu2011] to link the commits to their issues and re-implement the classification by Hindle *et al.* [@Hindle2008] on top of it. 
However, CommitGuru has the advantage of being open source. We were able to modify it to fit our needs and ensure a satisfactory performance.

\input{tex/table-words}

### Extracting blocks {#sec:block-extract}

A block is a set of consecutive lines of code that will be compared to all other blocks to identify clones. 
To achieve this critical part of BIANCA, we rely on TXL [@Cordy2006a], which is a first-order functional programming over linear term rewriting, developed by Cordy et al. [@Cordy2006a]. 
For TXL to work, one has to write a grammar describing the syntax of the source language and the transformations needed. 
TXL has three main phases: *parse*, *transform*, *unparse*. 
In the parse phase, the grammar controls not only the input but also the output forms.
The following code sample---extracted from the official documentation---shows a grammar matching an *if-then-else* statement in C with some special keywords: [IN] (indent), [EX] (exdent) and [NL] (newline) that will be used in the output form.


```bash
define if_statement
  if ( [expr] ) [IN][NL]
[statement] [EX]
[opt else_statement]
end define

define else_statement
  else [IN][NL]
[statement] [EX]
end define
```

Then, the *transform* phase applies transformation rules that can, for example, normalize or abstract the source code. 
Finally, the third phase of TXL, called *unparse*, unparses the transformed parsed input to output it. 
Also, TXL supports what its creators call _Agile Parsing_ [@Dean], which allow developers to redefine the rules of the grammar and, therefore, apply different rules than the original ones.

BIANCA takes advantage of that by redefining the blocks that should be extracted for the purpose of clone comparison, leaving out the blocks that are out of scope. 
More precisely, before each commit, we only extract the blocks belonging to the modified parts of the source code. 
Hence, we only process, in an incremental manner, the latest modification of the source code instead of the source code as a whole.

We have selected TXL for several reasons. First, TXL is easy to install and to integrate with the normal workflow of a developer. 
Second, it was relatively easy to create a grammar that accepts commits as input. This is because TXL is shipped with C, Java, Csharp, Python and WSDL grammars that define all the particularities of these languages, with the ability to customize these grammars to accept changesets (chunks of the modified source code that include the added, modified, and deleted lines) instead of the whole code.

\input{tex/extract}

Algorithm \ref{alg:extract} presents an overview of the "extract" and "save" blocks operations of BIANCA. This algorithm receives as arguments, the changesets and the blocks that have been previously extracted. 
Then, Lines 1 to 5 show the $for$ loop that iterates over the changesets. 
For each changeset (Line 2), we extract the blocks by calling the $~extract\_blocks(Changeset~cs)$ function. 
In this function, we expand our changeset to the left and to the right in order to have a complete block.

As depicted below, changesets contain only the modified chunk of code and not necessarily complete blocks. 

```diff
@@ -315,36 +315,6 @@
int initprocesstree_sysdep
(ProcessTree_T **reference) {
mach_port_deallocate(mytask,
  task);
}
}
- if (task_for_pid(mytask, pt[i].pid,
-  &task) == KERN_SUCCESS) {
-   mach_msg_type_number_t   count;
-   task_basic_info_data_t   taskinfo;
```

Therefore, we need to expand the changeset to the left (or right) to have syntactically correct blocks. 
We do so by checking the block's beginning and ending with a parentheses algorithms [@bultena1998eades]. 
Then, we send these expanded changesets to TXL for block extraction and formalization.

In summary, this step receives the files and lines, modified by the latest changes made by the developer and produces an up to date block representation of the system at hand.


## Online {#sec:online}

In this section, we describe the online part of our approach.
We call it _online_ because it is an interactive process with the software developer committing new modifications.
In this part, each time a commit is made, a pre-commit hook kicks in (Section \ref{sec:Pre-Commit-Hook}) and we extract the modified block using the technique previously presented (Section \ref{sec:block-extract}).
The newly modified blocks are compared to block modification known to have introduced a defect (Section \ref{sec:extracted}) and we recommend to apply the fixes that were used to correct the identified defect (Section \ref{sec:propose}). 


### Pre-Commit Hook {#sec:Pre-Commit-Hook}

Hooks are custom scripts set to fire off when certain important actions of the versionning process occur.
There are two groups of hooks: client-side and server-side.
Client-side hooks are triggered by operations such as committing and merging, whereas server-side hooks run on network operations such as receiving pushed commits.
These hooks can be used for all sorts of reasons such as compliance with coding rules or automatic run of unit test suites.

The pre-commit hook is run first, before the developer types in a commit message.
It is used to inspect the modifications that are about to be committed.
Depending on the exit status of the hook, the commit will be aborted and not pushed to the central repository.
Also, developers can choose to ignore the pre-hook.
In Git, for example, they will need to use the command `git commit –no-verify` instead of `git commit`.
This can be useful in case of an urgent need for fixing a bug where the code has to reach the central repository as quickly as possible. Developers can do things like check for code style, check for trailing white spaces (the default hook does exactly this), or check for appropriate documentation on new methods.

BIANCA is a set of bash and python scripts, and the entry point of these scripts lies in a pre-commit hook. 
Note that even though we use Git as the main version control system to present BIANCA, we believe that the techniques presented in this paper are readily applicable to other version control systems.

### Comparing the extracted blocks {#sec:extracted}

To compare the extracted blocks and detect potential clones, we can only resort to text-based matching techniques. 
This is because lexical and syntactic analysis approaches (alternatives to text-based comparisons) would require a complete program to work, i.e., a program that compiles. 
In the relatively wide-range of tools and techniques that exist to detect clones by considering code as text [@Johnson1993;  @Johnson1994; @Marcus; @Manber1994; @StephaneDucasse; @Wettel2005], we selected NICAD as the main text-based method for comparing clones [@Cordy2011] for several reasons. 
First, NICAD is built on top of TXL, which we also used in the previous step. 
Second, NICAD can detect Types 1, 2 and 3 software clones.

NICAD works in three phases: *Extraction*, *Comparison* and *Reporting*. 
During the *Extraction* phase all potential clones are identified, pretty-printed, and extracted. 
We do not use the *Extraction* phase of NICAD as it has been built to work on programs that are syntactically correct, which is not the case for changesets. 
We replaced NICAD’s *Extraction* phase with our scripts, described in the previous section.

In the *Comparison* phase, extracted blocks are transformed, clustered and compared to find potential clones. 
Using TXL sub-programs, blocks go through a process called pretty-printing where they are stripped of formatting and comments. 
When code fragments are cloned, some comments, indentation or spacing are changed according to the new context where the new code is used. 
This pretty-printing process ensures that all code will have the same spacing and formatting, which renders the comparison of code fragments easier. 
Furthermore, in the pretty-printing process, statements can be broken down into several lines. Table \ref{tab:pretty-printing} [@Iss2009] shows how this can improve the accuracy of clone detection with three `for` statements, ` for (i=0; i<10; i++)`, `for (i=1; i<10; i++)` and ` for (j=2; j<100; j++)`. 
The pretty-printing allows NICAD to detect Segments 1 and 2 as a clone pair because only the initialization of $i$ changed. This specific example would not have been marked as a clone by other tools we tested such as Duploc [@Ducasse1999]. 
In addition to the pretty-printing, code can be normalized and filtered to detect different classes of clones and match user preferences.

\input{tex/Pretty-Printing}

Finally, the extracted, pretty-printed, normalized and filtered blocks are marked as potential clones using a Longest Common Subsequence (LCS) algorithm [@Hunt1977]. 
Then, a percentage of unique statements can be computed and, given threshold (see Section \ref{sec:exp}), the blocks are marked as clones.


### Proposing Modifications {#sec:propose}

In the previous step, we compared the new modification contained in the commit at hand with modifications known to have introduced a defect in the past.
The defect can be in any of the repositories belonging to the same cluster as the project at hand, including the project itself.

As we linked bug-introducing changes to their commits (Section \ref{sec:bug-introduction}), we can propose to the developers not only the code that _looks like_ their modification and we know introduced a defect in the past but also what fixes have been deployed to eradicate it.

BIANCA does not rely on statistical models to predict risky commits but on code.
We believe that this can make BIANCA a practical approach for the developers as they will know why a given modification has been reported as risky in terms of code and not fit (or lack thereof) to a statistical model.
Furthermore, we propose concrete actions, in the form of code, which can be taken to reduce the risk of introducing defects to the system.
Finally, the online part happens before the commit reach the central repository (Section \ref{sec:Pre-Commit-Hook}), thus, preventing unfortunate pull of defect by other members of the organization.
    
# Evaluation  {#sec:exp}

In this section, we present our experimentations. 
We begin by explaining how we selected the repositories we analysed (Section \ref{sec:rep}) and the results of the clustering algorithm (Section \ref{sec:dependencies}).
Then, we show the effectiveness of BIANCA in detecting risky commits at commit-time using precision, recall and F-measure.

## Repository Selection {#sec:rep}

The selection of the open source repositories under analyze was made automatically according to the following criteria:

a) Java project using Maven for its dependencies.
b) Followed by at least 2000 unique developers.
c) Use a public issue repository.

These criteria ensure that (a) the dependencies of each project are retrievable in an automatic fashion and (b) that the repository has a significant interest expressed by the community.
When developers follow a project, they receive notification for each new commit and issue.
Finally, (c) we must be able to consult past issues and their resolution to be able to evaluate the efficiency of our approach.
A high number of followers demonstrates an interest of the community. 
For the reader to appreciate this hard limit of 2,000 followers, on Github, 298 Java projects have at least 2,000 followers.
The highest ranked repository, regarding followers, is react-native by Facebook with a total of 36,071 unique followers.
React native is a framework for building native apps with the React library (a JavaScript library for building user interface, also by Facebook).


Out of the 298 Java projects with at least 2,000 followers acquired using the Github API, 21 were referencing projects that have either gone private or been deleted.
On the 277 remaining projects, 42 repositories matched our criterion.
This 42 repositories composed our dataset.

Our dataset is composed of 28 different ecosystems including majors open-source contributors such as Alibaba, Apache Software Foundation, Eclipse, Facebook, Google and Square.

## Dependency analysis {#sec:dependencies}

Figure \ref{fig:dep-graph} presents the project dependency graph.
The dependency graph is composed of 592 nodes divided into five clusters shown in yellow, red, green, purple and blue.
The size of the nodes in Figure \ref{fig:dep-graph} are proportional to the number of connections from and to the node.

![Dependency Graph\label{fig:dep-graph}](media/network.png)

As depicted by our dependency map, the dependency landscape of the most popular Github repositories is very much interconnected and interdependent.
Indeed, we have an average of 77 dependencies per projects and, as seen earlier, 592 unique dependencies.
Meaning that, in average, our 42 repositories share 62 of their 77 dependencies with at least one other repository. 
Table \ref{tab:communities} presents the clusters, computed using the Girvan–Newman clustering algorithm, in terms of centroids, betweenness. The blue cluster (or community) is dominated by Storm [REF] from The Apache Software Foundation.
Storm is a distributed real-time computation system. Druid by Alibaba, the e-commerce company that provides consumer-to-consumer, business-to-consumer and business-to-business sales services via web portals, dominates the yellow cluster.
In recent years, Alibaba has become an active member of the open-source community and make some of its projects publicly available.
The red cluster has Hadoop by the Apache Software Foundation as its centroid. 
Hadoop is an open-source software framework for distributed storage and distributed processing of very large data sets on computer clusters built from commodity hardware.
The green layer is dominated by the Persistence project of OpenHab. 
OpenHab proposes home automation solutions and the Persistence project is their data access layer.
Finally, the purple cluster is dominated by Libdx by Badlogicgames.
Libdx is a cross-platform framework for game development.

We believe that these clusters are accurate as they efficiently divide repositories in terms of high-level functionalities.
We have the blue cluster which is almost entirely composed of projects from the Apache Software Foundation.
Projects from the Apache Software Foundation tend to build on top of one another.
We also have the red cluster for Hadoop which is itself an ecosystem inside the Apache Software Foundation.
Finally, we got a cluster for e-commerce applications (yellow), real-time network application for home automation (green) and game development (purple).

While it is difficult to assess the effectiveness of this clustering in terms of precision and recall because of the lack of the  ground truth, we show, in the next section, that this partitioning of projects provides excellent results.

\input{tex/clusters}

## Results of Bianca {#sec:result}

\input{tex/result}

In this section, we present the results of our experimentations.
Table \ref{tab:results} shows our results in terms of organization, project name, a short description of the project intent, number of class, number of commits, number of commits that introduced a defect, number of commits introducing a defect detected by BIANCA, precision (%), recall (%), F$_1$-measure and the average difference, in days, between detected commit and the _original_ commit inserting the defect for the first time. 

For a commit to be marked as risky (i.e., potentially introducing a defect) by BIANCA, the following conditions shall be met: 

a) The commit under analysis ($commit_a$) must be a Type 3 clone of an older commit ($commit_b$) with a similarity of at least 35%. 
b) $commit_a$ and $commit_b$ must belong to projects of the same cluster. 
c) $commit_a$ and $commit_a$ hashes are not equal.

Type 3 clones are blocks of code that are syntactically identical except literals, identifiers,  types that can be modified, indentation, whitespaces, and comments. 
Also, Type 3 can contain added or deleted code statements.
In our opinion type 3 clones (also known as near-miss) are more adequate for BIANCA than Type 1 and 2 as they allow blocks of code to have added or deleted code statements.
Type 4 clones (i.e., code blocks that perform the same tasks, but using a completely different implementation) would be an even better fit for BIANCA but (a) their detection is still a challenge [@Duala-Ekoko2007a] and (b) NICAD, our clone detection engine does not support Type 4 clone. 
The choice of the similarity threshold (35%) does not follow any specific indication, general law from clone detection or deeper insight into our dataset. 
We were only guided by the need to filter out all spurious matches while still keeping enough clones to represent the dataset.
Thus, we have made several incremental attempts, starting from 10%. 
For each attempt, we incremented the value by 5% and observed the obtained precision and recall.
The current value seems to offer the best trade-off.

Overall, we achieve a precision of 90.75% (13899/15316) commits identified as risky by BIANCA are true positives.
They did trigger the opening of an issue and had to be fixed later on.
BIANCA reaches 37.15% for the recall measure (15316/41225) and 52.72% for the F$_1$ measure.
Consequently, we can validate our assumption that project sharing dependencies are open to the same flaws. 
Moreover, detecting these common defects allow us to reach high precision. 
Obviously, our recall is somewhat low (37.15%) as we can only catch risky commit with common root causes.

It is important to note that we do not claim that 37.15% of open-source systems issues are directly linked to their dependencies. 
To support such a claim, the 15,316 detected commits would have to be analyzed manually.
However, we have proven that complex software systems sharing dependencies also share common issues, directly related to their dependencies or not.

Finally, we can evaluate the efficiency of our clustering by analyzing BIANCA's result with and without it.
First of all, the computation time per commit without clustering is 182% higher. 
Indeed, it goes from 32.2 seconds, in average, to 58.6 seconds (with an i5@1.8Mhz, 19.6 GiB of RAM and SSD disks on Debian 8).
Also, without the clustering, the recall metric reaches 42.4%, which is higher than our current 37.15%. 
However, when compared by projects with a Mann-Whitney test, the difference is not statistically significant (i.e. p-value > 0.05).
On the precision side, the metric drops significantly to 71.6% (i.e. p-value < 0.05).


# Threats to validity {#sec:threats}

The selection of target systems is one of the common threats to validity for approaches aiming to improve the analysis of software systems. 
It is possible that the selected programs share common properties that we are not aware of and therefore, invalidate our results. 
However, the systems analyzed by BIANCA were not selected per se. 
Indeed, we use all the systems available and matching our experimental criterion.
Moreover, the systems vary in terms of purpose, size, and history.
In addition, we see a threat to validity that stems from the fact that we only used open source systems. 
The results may not be generalizable to industrial systems. We intend to undertake these studies in future work.

The programs we used in this study are all based on the Java programming language. 
This can limit the generalization of the results. 
However, similar to Java, one can write a TXL grammar for a new language then BIANCA can work since BIANCA relies on TXL. 
Finally, we use NICAD as the code comparison engine. 
The accuracy of NICAD affects the accuracy of BIANCA. 
This said, since NICAD has been tested on large systems, we are confident that it is a suitable engine for comparing code using TXL. 
Also, there is nothing that prevents us from using other text-based code comparisons engines, if need be. 
In conclusion, internal and external validity have both been minimized by choosing a set of 42 different systems, using input data that can be found in any programming languages and version systems (commit and changesets).

# Conclusion {#sec:conclusion}

In this paper, we presented BIANCA (Bug Insertion ANticipation by Clone Analysis at commit time) an approach that can detect risky commit (i.e. commit likely to lead to a bug report) with a 90.75% precision and a 37.15% recall.
BIANCA uses code comparison of similar projects rather than statistical models built on change-level metrics.
Moreover, BIANCA is able to perform its detection of risky commit before the commit actually reaches the central repository using pre-commit hooks.
Finally, BIANCA is able to show the engineers the root cause behind the _riskiness_ of their commit with code.
Indeed, BIANCA presents engineers the defect introducing change against which the current commit has been matched and the related fixes.
We believe that all these characteristics can make engineers more interested and likely to adopt bug prediction tools as we (a) integrate ourselves with the developer's workflow, (b) propose concrete corrective action and (c) leverage historical code from similar projects.

To build on this work, we need to conduct an human study with developers and engineers in order to gather their feedback on the approach.
These feedbacks will help us to fine-tune the approach.
Also, we want to improve BIANCA to support Type 4 clones.

# Reproduction Package

We provide a reproduction package for this paper. 
Our reproduction package is available at [http://bit.ly/bianca-icse](http://bit.ly/bianca-icse) and provides the data at different stages of processing (initial, after clustering, final result) in addition to scripts and code used.




# References


<!-- Footnotes text -->

[^graphwalker-link-44]: https://github.com/jrtom/jung/issues/44
[^BatchEE-link-69]: https://issues.apache.org/jira/browse/BATCHEE-69
[^react-native]: https://github.com/facebook/react-native
[^maven]: https://maven.apache.org/
[^txl]:http://txl.ca


<!-- End Footnotes text -->

\footnotesize
