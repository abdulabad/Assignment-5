Install & call libraries
========================

    library(rpart)
    library(party)

    ## Loading required package: grid

    ## Loading required package: mvtnorm

    ## Loading required package: modeltools

    ## Loading required package: stats4

    ## Loading required package: strucchange

    ## Loading required package: zoo

    ## 
    ## Attaching package: 'zoo'

    ## The following objects are masked from 'package:base':
    ## 
    ##     as.Date, as.Date.numeric

    ## Loading required package: sandwich

Part I
------

    D1 <- read.csv("intelligent_tutor.csv", header = TRUE)

Classification Tree
-------------------

First we will build a classification tree to predict which students ask
a teacher for help, which start a new session, or which give up, based
on whether or not the student completed a session
(D1*c**o**m**p**l**e**t**e*)*a**n**d**w**h**e**t**h**e**r**o**r**n**o**t**t**h**e**y**a**s**k**e**d**f**o**r**h**i**n**t**s*(*D*1hint.y).

    c.tree <- rpart(action ~ hint.y + complete, method="class", data=D1) #Notice the standard R notion for a formula X ~ Y

    #Look at the error of this tree
    printcp(c.tree)

    ## 
    ## Classification tree:
    ## rpart(formula = action ~ hint.y + complete, data = D1, method = "class")
    ## 
    ## Variables actually used in tree construction:
    ## [1] complete hint.y  
    ## 
    ## Root node error: 250/378 = 0.66138
    ## 
    ## n= 378 
    ## 
    ##      CP nsplit rel error xerror     xstd
    ## 1 0.052      0     1.000  1.084 0.035034
    ## 2 0.012      1     0.948  1.012 0.036587
    ## 3 0.010      2     0.936  1.000 0.036803

    #Plot the tree
    post(c.tree, file = "tree.ps", title = "Session Completion Action: 1 - Ask teacher, 2 - Start new session, 3 - Give up")

Part II
-------

Regression Tree
===============

We want to see if we can build a decision tree to help teachers decide
which students to follow up with, based on students' performance in
Assistments. We will create three groups ("teacher should intervene",
"teacher should monitor student progress" and "no action") based on
students' previous use of the system and how many hints they use. To do
this we will be building a decision tree using the "party" package. The
party package builds decision trees based on a set of statistical
stopping rules.

Take a look at our outcome variable "score"
===========================================

    hist(D1$score)

![](Assignment_5_Abdul_Abad_Final_Submission_files/figure-markdown_strict/unnamed-chunk-4-1.png)

Create a categorical outcome variable based on student score to advise the teacher using an "ifelse" statement
==============================================================================================================

    D1$advice <- ifelse(D1$score <=0.4, "intervene", ifelse(D1$score > 0.4 & D1$score <=0.8, "monitor", "no action"))

Build a decision tree that predicts "advice" based on how many problems students have answered before, the percentage of those problems they got correct and how many hints they required
=========================================================================================================================================================================================

    score_ctree <- ctree(factor(advice) ~ prior_prob_count + prior_percent_correct + hints, D1)

Plot tree
=========

    plot(score_ctree)

![](Assignment_5_Abdul_Abad_Final_Submission_files/figure-markdown_strict/unnamed-chunk-7-1.png)

Please interpret the tree, which two behaviors do you think the teacher
should most closely pay attemtion to?

Test Tree
=========

Upload the data "intelligent\_tutor\_new.csv". This is a data set of a
differnt sample of students doing the same problems in the same system.
We can use the tree we built for the previous data set to try to predict
the "advice" we should give the teacher about these new students.

    #Upload new data

    D2 <- read.csv("intelligent_tutor_new.csv", header = TRUE)

    #Generate predicted advice for new students based on tree generated from old students

    D2$prediction <- predict(score_ctree, D2)

Part III
--------

Compare the predicted advice with the actual advice that these students
recieved. What is the difference between the observed and predicted
results?

    #All students in the new sample achieved a score of 1, and therefore would need no action. Therefore the accuracy of your model on the new data would be:
    mean(ifelse(D2$prediction == "no action", 1, 0))

    ## [1] 0.58
