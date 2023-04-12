---
layout: single
title:  "Monte Hall Problem With Bayesian Networks"
date:   2023-03-30 14:21:40 -0600
categories: jekyll update
classes: wide
toc: true
toc_icon: "cog"
---
Bayesian networks are a powerful tool used in probabilistic modeling to represent and reason about uncertainty. BNs have been used in various fields, including medicine, finance, and engineering, to model complex systems and make predictions. In this blog post, I explore Bayesian networks through the Monty Hall problem. I use R and the `bnlearn` package.


The Monty Hall problem is a classic probability puzzle that was popularized by the game show "Let's Make a Deal." 
> In the problem, a contestant is presented with three doors. Behind one of the doors is a prize, and behind the other two doors are goats. The contestant chooses a door, and then the host, who knows what's behind each door, opens one of the other doors to reveal a goat. The host then offers the contestant the chance to switch their choice to the other unopened door. The question is, should the contestant switch their choice or stick with their original choice?

A way of structuring a situation for reasoning under unceratining is to construct a graph representing causal relations between events. 


## Setting up the DAG
 *Directed Acyclic Graph (DAG) is a graphical representation of the probabilistic relationships between variables. A DAG is a directed graph, which means that the edges between the nodes have a direction, and it is acyclic, which means that there are no loops or cycles in the graph.*

The first step is to identify the set of variables that are relevant for the problem. In the simple network, we consider three variables:
1. Prize Door (**P**) - The door behind with the prize.
2. Door Picked (**D**) - The door picked by the contestent. 
3. Monty's Reveal (**M**) - The door revealed by Monty.

The next step is to create a node correpsonding to each of the variables indentified and to assign arcs. 

A **node** represents a random variable, such as a patient's symptom or a weather condition. Each node has a set of possible states that the variable can take on, and a probability distribution that describes the likelihood of each state given the values of its parent nodes.

**Arcs**, also known as edges, represent probabilistic relationships between nodes in the Bayesian network. An arc directed from node A to node B indicates that the probability distribution of B depends on the value of A. In other words, the presence or absence of an arc between two nodes reflects the conditional dependence or independence of the variables they represent.

In our case, all three variables will have the same three states *Door 1, Door 2, Door 3*. For the arcs, it is reasonable to consider *Prize Door* and *Door Picked* to be independent variables and *Monty's Reveal* to be dependent on the two, since Monty can only reveal the door not chosen by the participant and is not with the prize.

To complete the model, we need to specify a joint probability distribution over our variables. The natural choice is a multinomial distribution, assigning a probability to each combination of states of the variables. In BNs, this is called the **global distribution**. However, using the global distribution directly is difficult as the number of parameters are high. Instead, we use the DAG to break down  the global distribugtion into a set of smaller **local distributions**, one for each variable. Since, the arcs represent direct dependencies, variables that are not linked by an arc are *conditionally independent*. Thus,

$$
P(P, D, M) = P(P)\cdot P(D) \cdot P(M| P, D)
$$

There are two ways in which we can instantiate the network in `bnlearn`. The first is the verbose way where we create all the nodes with their states and then assign arcs. The second takes advantage of the local distribution factorization, which is what I prefer to use. 

{% highlight r linenos %}
library(bnlearn)

sdag <- model2network("[P][D][M|P:D]")
graphviz.plot(sdag, layout = "dot")
{% endhighlight %}

![Alt text](/images/montyhallsimple/simple_dag.png "DAG")

## Node Probability Tables (NPT)
We need to specify the conditonal probability tables for each of the nodes. This is usually the hardest part of the modeling process. The tables can be assigned by the modeler or learnt from data (or both). 

{% highlight r linenos %}
  # set the levels 
  levels_D <- c("door1", "door2", "door3")
  levels_P <- c("door1", "door2", "door3")
  levels_M <- c("door1", "door2", "door3")


  # NPTs
  D_prob <- array(rep(1/3, 3), dim = 3, dimnames = list(D = levels_D))
  P_prob <- array(rep(1/3, 3), dim = 3, dimnames = list(P = levels_P))
  M_prob <- array(
    c(0, 0.5, 0.5, 0, 0, 1, 0, 1, 0, 
      0, 0, 1, 0.5, 0 , 0.5, 1, 0, 0, 
      0, 1, 0, 1, 0, 0, 0.5, 0.5, 0),
    dim = c(3, 3, 3),
    dimnames = list(M = levels_M, D = levels_D, P = levels_P)
  )

  # CPT for the model
  cpt <- list(D = D_prob, P = P_prob, M = M_prob)
{% endhighlight %}

## Fitting the model
We are ready to fit the model. Since we assigned the CPT, we use `custom.fit()` function and pass our DAG along with the CPT.

{% highlight r linenos %}
  cpt <- list(D = D_prob, P = P_prob, M = M_prob)
  fitted <- custom.fit(sdag, cpt)
{% endhighlight %}

The `bnlearn` package provides useful functions to get details about the BN. For example, we can get a list of the arcs using `arc()` funciton and the total number of parameters in the model using `nparams()` function.

![Alt text](/images/montyhallsimple/initial_chart.png)

## Query the model
Now that we have set up the BN and fitted it with the CPT, we can query it. We could use the `cpquery()` function in `bnlearn` but it returns an approximate inference through sampling. Since our model is simple and discrete, I use the `gRain` package to get an exact inference. The `gRain` package creates a junction tree to speed up the computation of conditional probabilities. To query the tree, we have to set evidence or *instantiate* the nodes. Say the contestant picks door 1 (**D** = Door 1) and Monty reveals door 2 (**M** = Door 2).

{% highlight r linenos %}
  library(gRain)

  # create junction tree
  junction <- compile(as.grain(fitted))

  # set evidence. 
  evidence <- list(D = "door1", M = "door2")

  j_inst <- setEvidence(junction, 
                     nodes = names(evidence), 
                     states = unlist(evidence))
  querygrain(j_inst, nodes = "P")
{% endhighlight %}

`querygrain()` function recalculates the CPT by propogating the evidence provided and returns the NPT for the node specified.

|  P   | Initial       |   After       |
|:-----|--------------:|--------------:|
|door1 | 0.3333333     |  0.3333333 |
|door2 | 0.3333333     |  0.0000000     |
|door3 | 0.3333333     | 0.6666667     |

All of the probability that was previously associated with the two doors not chosen by the contestant is loaded onto door 3. Thus,
there is a 2/3 probability that the contestant will win by switching. Very cool. 

## Final thoughts
The `bnlearn` package provides a flexible yet powerful framework to build and query Bayesian networks. I especially liked the straighforward plotting funcitonality. It would've been
nice to have the fitted model object be updated with evidence that can then be plotted again with updated NPTs. Perhaps I just need to explore the package more. Next, I want to 
explore the Monty Hall problem with a slightly more complex, but causely purer model. 
