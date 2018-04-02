# Reviewers' Comments

## Reviewer: 1

_Summary: the paper presents an approach, called BIANCA, to predict potentially buggy code changes at commit time by detecting code clones between the changes being committed and the known historical buggy changes across software projects that share library dependencies. It can also suggest fixes for the buggy commits based on the fixes for the historical bugs. BIANCA works by (1) building dependency graphs among many open source projects and segregating the projects into multiple clusters based on a clustering algorithm, (2) building a database of historical buggy commits and fix commits, (3) applying clone detection to detect potentially buggy commits. The evaluation of 42 large projects from GitHub shows that the approach could achieve reasonable precisions but low recalls._

- Nothing to add.

_Evaluation: overall, the technique has a major problem that may invalidate various arguments and evaluation results in the paper, and some technical details are unclear although the idea of using project dependencies to scope the search spaces for potentially buggy commits across projects is somewhat new._

- The novelty of the approach lies is not only to reduce the search space of potentially buggy commits but also proposing fixes to these commits.

_It should refer to more state-of-the-art defect prediction techniques, instead of a simple random classifier, and evaluate BIANCA against them._

- As noted in section 5.1 _To the best of our knowledge, this is the first approach that relies on  code similarity instead of code or process metrics for the detection of risky commits. Comparing it to other approaches will not be accurate. In addition, existing metric-based techniques (e.g., [@Nam2013]) detect risky commits within single projects only. BIANCA, on the other hand, operates across projects. We compared BIANCA  with a random classifier to have a baseline and show that we perform better than a simple baseline._
- @Emad, would it be possible for you to run commit guru on the same projects ? BIANCA will likely lose by a bit but we can argue that we offer the fixes. We can also plan a merge of both approaches for future work.

_Its discussion should consider more important threats to validity. More detailed comments are as follows._

- Nothing to add.

_Regarding the idea of the paper, it is essentially a defect prediction technique based on code clones: if a newly committed code change is "similar" to a known buggy commit, then it is likely to be buggy too. This idea has existed in many papers in the literature, although the definitions of "similarity" have many variants. This paper borrows the similarity metric used in the NiCad clone detector, but how the metric threshold is used is the major problem of the paper (see further comments below related to technical expositions)._

- Will address on the detailed comments

_The paper is different from other papers by constructing a search space of similar buggy commits by clustering all historical buggy commits across projects based on their shared dependencies, but the benefits of this clustering strategy are not sufficiently justified (see further comments below)._

- Will address on the detailed comments

_Regarding its technical exposition, the major problematic setting is the similarity threshold alpha used in the NiCad clone detector. As illustrated in Figure 7, the precision of identified potential buggy commits is higher when alpha is larger, which is understandable. However, the paper uses the related work [54,61] to support its choice of an alpha around 35%, as it claims the related work uses an alpha around 30%, which is a completely reversed claim. Firstly, the 30% in the related work refers to "dissimilar" threshold, instead of "similar threshold"; the related work rarely uses a similarity threshold lower than 70%. Secondly, even if a 35% alpha is used, which means 65% of lines of code in clones can be different from each other; then, should two pieces of code still be considered as clones if they have so much difference? If what reported are no longer "valid" clones, what's the point of using clone detection for defect prediction in this paper?_

- In [61], _C. K. Roy and J. R. Cordy, “An Empirical Study of Function Clones in Open Source Software,” in Proceedings of the working conference on reverse engineering, 2008, pp. 81–90._ Section 3.2, the authors wrote the following: _"In this paper we present our results for the representative set of UPI thresholds 0.0, 0.10, 0.20 and 0.30, ..."._ This is also explained in [54] _J. R. Cordy and C. K. Roy, “The NiCad Clone Detector,” in Proceedings of the international conference on program comprehension, 2011, pp. 219–220. In the previous version of the paper, we incorrectly reported that a similar alpha of 30% was used in [54, 61]. As the reviewer rightly points, in these papers it is a dissimilarity threshold while ours is a similarity threshold. In practice, we used NICAD 3.5 and put the `threshold` (usually found at line 11 of the configuration file) values to 0 (no difference) to 100 (all lines are different), with 1% increments and found 65% to be the best values in terms of precision and recall for defect detection. Nicad was built for and evaluated on its capacity to detect near-miss clones (i.e. clones that are almost identical). For this purpose, a very-high similarity threshold (or low dissimilarity threshold) is desirable. For our purpose however, one similar line in a block could be significant. The trivial example being the `null` check as exposed in the examples given from page 12 to 17. The same reasoning apply for fixes. As pointed out by other reviewers, the degree of helpfulness of the proposed fixes could be addressed by an user study. As a future work, we intend to focus on one system or two for which we could secure a collaboration with developers of said systems. We believe it is important for the actual developers of a selected system to be involved in the evaluation of fixes.

_In Figure 7, the lines for precision and recall are almost flat and close to 100% and 0% respectively when alpha is larger than 60%. What does that mean? Does it imply there are really few buggy commits that are "similar" to each other, and clone detection may not be an effective approach for defect prediction?_

- You interpretation would be correct if we used our full dataset to compute alpha. However, as explained in the paper in section 4.4 "_We randomly selected a sample of 1700 commits from our dataset and checked the results by varying α from 1 to 100%_".

_How is the dependency between two projects determined? It only says it's based on Maven. Does it mean the dependency relations used in the paper are at the project level, without considering function call relations? Then, why the dependency graph contains 592 nodes when there are only 42 projects? If each node is not a project, what is it?_

- Maven is a dependency managed system used for Java projects. Using the maven description file (pom.xml), we can extract all the dependencies of a given project. The dependencies of all the projects are then used to compute the clusters. We do not investigate the function calls to create the clusters. The graph contains 592 nodes that are all JAVA projects. They are the dependencies of our 42 selected projects.

_When the projects are segregated, the Girvan-Newman algorithm for community detection is used. But why is this algorithm chosen, as there could be many different ways to constrain the search space for similar code changes? For a simple alternative, I would consider a dynamic neighborhood selection strategy that, when given a new commit in a project, all projects that share dependencies with this project are used for setting the search space, i.e., 2-degree neigbors of the project in the dependency graph. This way, the search space for each project can be tailored to itself and different from others. I would appreciate more comparisons and insights into why this Girvan-Newman algorithm is chosen to set up the static search spaces for each project in this paper._

- To be addressed.

_When the paper says "we expand our changeset to the left and to the right in order to have a complete block." How is the "expansion" done exactly?_

- The number of non-modified lines (i.e. without - or +) in a diff is known as the context. Using the `git` command line, one can adjust the context of a diff in order to obtain more lines. For example, `git diff -U$(wc -l MYFILE)` gives all the lines of the `MYFILE` including the modified ones. When the context is not sufficient in the original diff, we expand it by 100 lines. Then, we explore the code until we can discover a complete block.

_To make the changeset a syntactically valid piece of code, how are the "-" and "+" lines are transformed? Are "-" and "+" both included or separated?_

- For each diff, we consider two versions. The before (after) version that is composed of the unmodified lines plus the lines preceded by a "-" ("+"). In the preprocessing step, the before and after version are store independently without their markers (i.e. "+" and "-") which makes them syntactically valid. Then, we compare the after version of incoming commit with the before version of known fixes. If the after version of a new commit matches the before version of a diff applied to fix a bug it means that it is likely to introduce a defect. Finally, we propose the after version of the fix to  developers in order to help them fix their defect.

_When commits are "replayed" in the evaluation, how are they done? Because this paper involves cross-project defect prediction at change commit time, the replay should be done in a way that all commits from all related projects were considered together in the order of their commit timestamps. It would not match realistic settings if the replay was done for all commits in each project separately_

- Suppose that we have three commits from two projects as presented by Figure 6. At time t 1 , commit c 1 in project p 1 introduces a defect. The defect is experienced by a user that reports it via an issue i 1 at t 2 . A developer fixes the defect introduced by c 1 in commit c 2 and closes i 1 at t 3 . From t 3 we know that c 1 introduced a defect. If at t 4 , c 3 is pushed to p 2 and classified by the metric-based classifier as risky, we extract c 3 blocks and compare them with the ones of c 1 . If c 3 and c 1 are a match after preprocessing, pretty-printing and formatting, then c 3 is classified as risky by BIANCA and c 2 is proposed to the developer as a potential solution for the defect introduced in c 3.

_The paper says "To address this [false positives], we did not include the last six months of history." This processing may indeed cause false "false positives", hurting the precisions, but it would also hide false negatives, benefiting the recalls. How could this problem of unseen future bugs be better addressed?_

- This is a known problem. @Emad, could you answer this?


_Regarding related work, the paper itself says it "shares a similar goal to the work on the prediction of risky changes" even though the approach is different from others. However, no sufficient amount of related work on change-level (or just-in-time) defect prediction is discussed, especially those papers sharing some elements of its idea. For example,_
- _Xinli Yang, David Lo, Xin Xia, Jianling Sun: TLEL: A two-layer ensemble learning approach for just-in-time defect prediction. Information & Software Technology 87: 206-220 (2017)_
- _Tan, M., Tan, L., Dara, S. and Mayeux, C., 2015, May. Online defect prediction for imbalanced data. In Proceedings of the 37th International Conference on Software Engineering-Volume 2 (pp. 99-108). IEEE Press._
- _Hall, T., Beecham, S., Bowes, D., Gray, D. and Counsell, S., 2012. A systematic literature review on fault prediction performance in software engineering. IEEE Transactions on Software Engineering, 38(6), pp.1276-1304._
- _Turhan, B., Menzies, T., Bener, A.B. and Di Stefano, J., 2009. On the relative value of cross-company and within-company data for defect prediction. Empirical Software Engineering, 14(5), pp.540-578._
- _Pattern Insight Code Assurance: Detect and Eliminate Known Software Bugs: https://vimeo.com/22633955 2011_
- _Islam, R. and Sakib, K., 2014, December. A Package Based Clustering for enhancing software defect prediction accuracy. In Computer and Information Technology (ICCIT), 2014 17th International Conference on (pp. 81-86). IEEE._

- To do

_Regarding its evaluation, it's insufficient to just compare with a random classifier, in view of many existing studies on change-level, cross-project defect prediction. The paper may refer to the above papers to see how comparisons with many other techniques for the same purpose can be performed._

- @Emad, could we do commit guru ? Would it be sufficient ?

_In the discussion of several cases detected by BIANCA, I don't understand the claim in the paper that knowing one commit can help developers to prevent the error in another commit. For example (Figures 8 and 9), although both pieces of code involve null-pointer dereferencing, the two pieces of code have different structures (i.e., two sequential "if" statements with two "return" statements vs. two nested "if" with one "return" only), and null-pointer dereferencing is a common defect pattern. How could a developer know that the "aliasMap" in Figure 9 really needs a check for null-pointer just by looking at a null-pointer check of an irrelevant variable in FIgure 8? How much cognitive burden does that put on the developer, especially when the code is from different projects? Or, does it suggest to simply add null-pointer checks for all pointers? This same kind of complaints is applicable to all the other cases in the paper, and I think it's related to the aforementioned major problem and a fundamental question for the paper: to what extend clone detection is suitable for detecting potentially buggy commits? What's the suitable similarity threshold setting?_
- _A reference on how clones can be really used for detecting security bugs at Microsoft may be of value for the paper to answer such a question: Dang, Y., Zhang, D., Ge, S., Huang, R., Chu, C. and Xie, T., 2017, May. Transferring code-clone detection and analysis to practice. In Proceedings of the 39th International Conference on Software Engineering: Software Engineering in Practice Track (pp. 53-62). IEEE Press._

- To do.

_In the discussion of threats to validity, it again makes a wrong claim about the "similarity threshold" based on a misunderstanding of the related work [54,61]. Further, it should discuss the fundamental question mentioned above and consider why its recalls are so low._

- See above.


## Reviewer: 2

_Public Comments (these will be made available to the author) Summary: The paper proposes an approach called BIANCA, that uses clone detection and dependency analysis to build a model that assists developers while committing source code. In particular, BIANCA can analyze the previous commit history along with bug reports, to identify the defect introducing commits and their associated commits that helped fix those commits. Later when a developer is introducing a new commit, BIANCA uses clone-detection mechanism to identify whether this commit matches with any of the previously defect introducing commits. If yes, BIANCA recommends the fixed commit so as to avoid defect being introduced. BIANCA can also deal with many projects that share dependencies as well. Authors applied their technique on a large set of open source projects and showed the effectiveness of BIANCA._

_The overall motivation of the paper is really nice. The idea of using pre-commit hooks to provide an immediate feedback during the commit time will be really useful in practice. Also, it is common across many projects to share dependencies. Therefore an approach that takes a holistic view of all projects that shares common dependencies is really useful, and that is one of the strong aspects of BIANCA._

_Overall, the approach is straight forward in the sense to build a repository of defect introducting commits along with the commits that fix those defects, and use this data for identifying new risky commits. Most of the work seems to be on the engineering side of putting various existing techniques together to make it scale for the large projects as demonstrated in the evaluation._

_Although there are many good aspects of the approach, there are a few major concerns that needs attention._

_First, this is a recommender system. Therefore, benefits of the system can be assessed only by using case studies with real developers. Although authors have carefully compared recommended commits with the actual commits, the real value can be assessed only by having actual developers use the tool._

- This is true and we intend to pursue an human study as future work.

_The primary advantage of BIANCA compared to other techniques is its ability to deal with projects that share dependencies. With this ability, the expectation was to assist with the correct usage of APIs of the libraries that are shared by the projects. This will be extremely valuable since often documentation is not available or outdated for libraries and frameworks. However, the code samples presented in pages 12 to 17 does not seem to indicate that it is the case. Did authors find any examples where they detect risky commits because of not using the APIs of shared libraries correctly?_

- The examples presented in pages 12 to 17 were selected randomly and, indeed, do not present any misusage of APIs. In this revision, we manually identified missusage of API in our dataset and present them.


_Another major concern is with the kinds of recommendations that are being done by BIANCA, specifically focusing on the examples presented in pages 12 to 17. Looking at those examples, even if BIANCA marks a commitas risky and suggests the example, it does not seem to be obvious to the developer to infer the issue. For example, let us take the example presented in Figures 12 and 13. Looking at the previous code in the commit in Fig 13, it is clear that the developer already knew about the append method. Authors mentioned that the code in Jsoup is modified to avoid appending of "null" string. However, looking at the modifications in Jsoup, that does not seem to be the primary intention. The change in Jsoup does much more than the simple null check. It would be great if authors can present more interesting examples that highlight the benefit of BIANCA beyond null checks and exception cases._

- Once again, presented report were selected randomly. We added some manually selected example in this revision.

_In Page 3, Section 3.2, it is not clear how authors leveraged CommitGuru for their purpose. It is intuitive to understand how CommitGuru is identifying defect introducing commits by building statistical models. Now, how are these commits linked to their fix commits is not clear. Recommend authors to clarify the following paragraph by explaining what Commit-guru identifies, and how they arrive at fix commits. " Commit-guru implements the SZZ algorithm [43] to detect risky changes, where it performs the SCM blame/annotate function on all the modiﬁed lines of code for their corresponding ﬁles on the ﬁx-commit’s parents. This returns the commits that previously modiﬁed these lines of code and are ﬂagged as the defect introducing commits (i.e., the defect-commits)._

- Emad, could you answer this one?

_Regarding Section 3, it would be nice to have one single working example to completely understand the approach, especially the phases described in Fig 1. The example can present some description of fix, how they get the changeset associated with the fix. Now using the Blame, how authors arrive at the DefectIntroducting commit which also has a changeset. At this point, there will be two changesets. Currently, it is not clear how ExtracedModifiedBlock (which seem to be composed of two changesets) looks like in Fig 1. What is the purpose of this? Request to please explain this clearly._

- At reviewer's request, we added a working example of the approach.

_In Section 3.3, authors mentioned about the improvements made for the comparison phase of NICAD, especially related to the improvement of the accuracy of clone detection. However, no evidence is presented in the evaluation section whether this helped improved the accuracy or not. It would be good to use a couple of subjects with/without the improvement and report the results._

- Can't find any claim regarding accuracy improvements.

_Sections 3 and 4 need to be rearranged a bit. It seems lot of key details that should be in Section 3 are described in Section 4 as a part of case study. For example, in Section 4.4, authors repeated how BIANCA works with a nice example using commits c1 to c3 in projects p1 and p2. An ideal place for this example is in the beginning of Section 3. Having details split like this makes the reader to feel there is lot of repetition, and also need to keep stitching details to get the complete overview._

- Reviewer's is right. We addressed this in this revision.

_There is another fundamental question about the evaluation. Authors used Commit-guru results as baseline. Now with respect to those results, their tool achieved F-measure of 52.72% (Precision: 90.75%, Recall: 37.15%). Authors explicitly mentioned the following as well in the paper: "The process used by Commit-guru to identify commits that introduce a defect is simple and reliable in terms of accuracy and computation time". So, if Commit-guru works well, why can't a developer just use Commit-guru itself? What is the main contribution or advantage of BIANCA?_

- BIANCA provide the fixes and, for the detection aspect, we make the hypothesis that a classifier based on code would be easier to trust for developers compared to a classifier based on metrics.


_Also, the results are not as good as Commit-guru given that commit-guru's results are used as baseline. Request authors to see whether they can use another source of truth as baseline to evaluate the effectiveness of the technique. That way, they can compare both Commit-guru and BIANCA to show whether BIANCA can perform better than Commit-guru._ 

- Emad, do you have another algorithm implemented and ready to go ?

_Regarding Section 5.1, it is really not clear what is purpose of this. Request authors to remove this section._

- This section has been modified to incorporate other baselines.

_Regarding Section 5.3, where authors conducted analysis of fixes proposed by BIANCA, the results show that average similarity of the first three fixes is 44.17% and similarity first five fixes is 40.78%. In practice, given such low similarity, it is certainly difficult for a developer to understand the suggested fix of BIANCA. Again this result shows that is important to conduct a case study using real developers. Otherwise, it is really hard to assess the effectiveness of BIANCA._ 

- Agreed. Future work.

_The following statement is not clear: "In other words, BIANCA is able to detect risky commits with 90.75% precision, 37.15% recall, and proposes ﬁxes that contain, on average, 40-44% of the actual code needed to transform the risky commit into a non-risky one. ". Especially how the 40-44% has been arrived. Could you please rephrase and add more details to make it clear?_

- Will do.

## Reviewer: 3

_Public Comments (these will be made available to the author) The paper proposed a bug prevention technique called BIANCA ((Bug Insertion Anticipation by Clone Analysis at commit time). BIANCA compares the code block of incoming commit with the known defect-introducing commit instead of just assessing the risky commit. BIANCA can also detect vulnerability among dependent libraries rather than just single project. For every commit by developers, BIANCA compares that with the historical defect commit in the database. If there is a match found, then the new commit is considered as risky commit. BIANCA also recommend developers about the way to fix the identified risky commit. BIANCA perform sits task in two parallel processes. In the first, process BIANCA identifies fix-commit and defect-commit. In second process, BIANCA identifies risky commit. Code block from fix commit and defect commit are extracted using TXL – a first-order functional programming language. Risky commit is detected by Commit-guru. For each match between risky commit and defect commit, BIANCA extracts fix commit from the database.  A threshold value (α = 35%) determines the match between risky commit and defect commit. The paper used NICAD to identify clones.  BIANCA is evaluated on 42 top ranked projects from GitHub that uses Java and has at least 2000 followers. The dependency analysis showed that the project has on average 77 dependency and 62 of them are dependent on at least one project of BIANCA’s list. The evaluation showed that BIANCA outperforms a baseline model by 19.48%. On average BIANCA can detect risky commits of 90.75% precision and 37.15% recall. The evaluation showed that 8.6% of risky commit showed by BIANCA match other commits for same project. This indicates that vulnerabilities among dependent libraries are needed to consider when detecting risk commit.Bug fixing is an everyday work for software developers. They spend lots of time in fixing bugs compared to the code-level works. The authors tried to prevent bugs using dependency analysis and code detection. The bug prevention strategies are still at its early stage. Lives of programmers would be lot easier if they could apply bug prevention strategies in their coding. I believe, the authors worked on a very important issue of software development process. POSITIVES:_

_Organization of contents is very good.  The explanation of BIANCA approach is clear. The authors clearly explained the three steps of BIANCA like clustering, database building and the analysis of new commits. Also, the explanation of each step in setting up case study is well organized._
_Flow and consistency is well maintained. The sections and subsections are ordered as I have expected as a reader. I did not have to go back and forth to find any particular piece of information._
_The logic behind the use of tools like NICAD and Commit-guru are explained well. NICAD has advantage of detecting Type 1, 2 and 3 software clones. Also, NICAD works in three phases of extraction, comparison and reporting. Authors explained all three phases in detail. The components of Commit-guru are also explained well._
_BIANCA algorithm, figures and tables are presented clearly and at right place. Results are presented in detail._
_Overall, the authors have done a good job in the paper. The presentation of the work and the use of terminologies are understandable. Linguistic clarity is perfect too._

_BIANCA is analyzed and evaluated on Java-based projects only. The performance of BIANCA is not guaranteed for non-java-based projects. Therefore, the applicability of BIANCA is not generalizable for all project environments._

- This is true, however, there is not technical limitation to use BIANCA on any language supported by TXL (c, c++, c#, java, ...). In addition, writting a grammar for other language is not impossible too.

_In page 2 line 9: Given statement is, “Note that we do not detect only the bugs resulting from library usage but rather leverage the fact that if two systems use the same libraries, then they are likely vulnerable to the same flaws.”. For this statement, authors haven’t given any reference or case study._

- The examples presented from pages 12 to 17 show that we also detect bugs that are not related to missuses of APIs.

_To support this sentence “BIANCA not only flags risky changes, but also provides developers with fixes that have been applied in the past”, there should be a study with examples on how previously fixes related to current code fragment. Some other dependencies and domain should be considered before suggesting fixes._

- ??

_In page 4 line 25: For this sentence “BIANCA identifies risky commits within each group to increase the chances of finding risky commits caused by project dependencies”, an explanation required to relate risky commits and dependent projects within a group._

- It increases the chances because the dataset becomes bigger. We could match every commit against every other known defect but it' ll be impracticable in terms of computation. That is why we rely on clusters.

_In this system, the dependency detection technique totally depends on Maven based managed project. There is no study for other types of project or how dependency can be generated for other projects._

- We rely on Maven because it is a well-known and heavily used dependencies management system. However, the approach is replicable with any type of structured dependencies management system. For example, graddle could be used for Java, gopkg for Golang, Nuget for C#, pip for python and so on.

_How the modified version of commit guru is used to establish link between fix-commits and related issues is not clear. Section 3.2 should be described properly._


- SZZ. Emad, could you transform a bit that section so it'll be clearer ?

_Page 4 is an image._

- ?

_Authors have used modified version of NICAD to match code fragments, even a TXL grammar has been proposed to solve their purpose and type 3 clone detection is considered to match code blocks. But less than 100 % match of code fragment how related to be a risky code is not properly explained._

- Not sure to understand.

_In the sentence “We use the commit-guru labels as the baseline to compute the precision and recall of BIANCA” authors have mentioned that commit-guru is used as a baseline for defect-commits, but they didn’t show any performance study of this tool for detecting defect-commit. So, based on that tool’s labels given precision and recall cannot be justified._

- New baseline classifier to be computed. 

_How dependency or Betweenness among projects has been calculated, should be discussed elaborately._

- Same comments by R1, to be addressed.

_Page 8 has some issues with text and image formatting_

- Can't see where.

_The paper has no explanation of why the used threshold value is 35% (α = 35%). There is no explanation of why 35% threshold of NICAD giving highest precision and recall in this defect-code match system, which is designed especially for clone detection._

- The recall and the precision are not computed with regards to NICAD discovering clones but with BIANCA discovering bugs. The rigor at which NICAD detects clones is a proxy for the precision and recall of BIANCA.

_“Can we detect risky commits using code comparison within and across related projects, and if so, what would be the accuracy?”, this important question has been asked but to answer this question CASE STUDY RESULTS do not provide sufficient information._

- Examples provided between pages 12 to 17 were trying to answer this question. More examples and discussion have been added.

_How this statement “Developers of a project are not likely to introduce the same defect twice while developers of different projects that share dependencies are, in fact, likely to introduce similar defects.” has been proven, not clear from the study. Proper explanation with examples is necessary to support above statement._

- Talk about inner project detection vs. cross-project detection.
