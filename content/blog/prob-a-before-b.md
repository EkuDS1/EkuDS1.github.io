---
title: "What's the probability that A happens before B?"
date: 2020-09-15T11:30:03+00:00
math: true
tags: ["probability"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
disableShare: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
---

## Intro
Most aspects of conditional probability seem simple enough. You assume that an event will occur or has already occurred and decide the probability of an event based on that assumption. For example, when picking two cards from a deck of 52 cards without replacement if you assume that the first card was a heart, the probability of picking a second heart would be dependent on that information about the first card.

The formula $P(A|B)=\dfrac{P(A\cap B)}{P(B)}$ is also relatively easy to understand by itself. It makes sense how it applies to the example above and to examples involving things like contingency tables.

## Problem
But what about the probability of an event $A$ occurring before an event $B$? Assuming that we interpret this to mean an event $A$ happening exactly once before an event $B$ happens exactly once, the probability is supposedly $P(A|A\cup B)=\dfrac{P(A)}{P(A\cup B)}$. *From Hogg and Tanis' Probability and Statistical Inference, Example 1.3-4, Chapter 1.3 Conditional Probability*

Let's break this down.
## An Attempt at a Solution
What does it mean for an event $A$ to happen before an event $B$? Let's assume both events are disjoint for now. What does this indicate?

Events are sets of outcomes. An outcome is the result of an experiment. If the events are disjoint, then the outcomes in $A$ are different from $B$ which implies that the question is asking about some outcomes happening before other outcomes. But how is it possible for one outcome to happen before another if a single trial of an experiment can only have one outcome? It's impossible, unless you have multiple trials of the same experiment. 

In that case, we can start talking about one outcome happening before another. Or about one event happening before another. We would then have a composed experiment. Just repeating two trials multiple times can give us that information.

And if we have multiple trials, what does it mean for $A$ to happen before $B$? If our experiment was defined as having 2 trials, our sample space of outcomes might look like $S=\{AA,AB,BA,BB,CA,CB,AC,BC,CC\}$ assuming we also have some event $C$ which is the union of all possible events other than $A$ and $B$. There could be five different interpretations of the phrase "$A$ happens before $B$":
1. $A$ must occur exactly once and the trials must end at $B$. $\{AB\}$ fits this description.
2. $A$ must occur exactly once as long as $B$ doesn't appear before $A$. $\{AB,CA,AC\}$ fits this description.
3. $A$ can occur more than once as long as $B$ doesn't appear before $A$. $\{AA,AB,CA,AC\}$ fits this description.
4. $A$ must occur exactly once and $B$ must not occur. $\{CA,AC\}$ fit this description.
5. $A$ can occur more than once and $B$ must not occur. $\{AA,CA,AC\}$ fits this description.

Here's how I'm thinking about it. Look at the outcomes and for each outcome, ask the question, "Did $A$ happen before $B$?". So for $AB$, yes $A$ did in fact happen before $B$. For $AA$, $A$ did happen before $B$ because in this outcome there's no way it could happen at the same time as or after $B$. And even if you continued doing trials after getting $AA$ (e.g. getting $AAC$ or $AAB$), it would not change the fact that in this outcome $A$ already happened before seeing a $B$. Similar reasoning leads us to believe interpretation 3 makes the most sense. So let's go with that.

But why consider just two trials? We could have any number of trials. A countably infinite amount. In other words, $CCAB$ is just as valid an example of "$A$ happened before $B$" as $AACBCBC$(under interpretation 3). How do we deal with this? Well, notice that when we had two trials, $AA$, $AB$ and $AC$ were examples of $A$ happening before $B$ in the case where $A$ happens in the first trial. So assuming that we have disjoint events, we use the law of total probability: 
$$P(AA\cup AB \cup AC)=P(AA)+P(AB)+P(AC)=P(A)(P(A|A)+P(B|A)+P(C|A))=P(A)(1)=P(A)$$

We can apply similar to logic to the cases where $A$ doesn't happen on the first trial but does happen on the second trial. $P(CA)$ is the same as $P(CAA\cup CAB \cup CAC)$. Same for $CCA$ and so on. Thus the probability of $A$ happening before $B$ is $P(A)(1+P(C)+P(C)^2...)$ assuming that all events are disjoint and independent. Since $|P(C)|\leq 1$, we get the geometric series sum to infinity $P(A)\dfrac{1}{1-P(C)}$. Since $P(C)=1-P(A\cup B)$, we can substitute and get $\dfrac{P(A)}{P(A\cup B)}$.

## Remarks
It's weird how despite the fact that I got a closed form solution, I don't really get how this formula directly connects with the concept of $A$ coming first. I can think of an intuitive explanation(that the only events of interest are $A$ and $B$ so we condition on that) but that explanation isn't all that satisfying. It makes me think that our solution above is expressing information about multiple trials using information about just one trial. And I find that strange.

I think I know in which situation I would use such a probability but I'm not sure if I would call that "understanding".