---
title: The FLP Impossibility, Asynchronous Consensus Lower Bound via Uncommitted Configurations
date: 2019-12-15 09:15:00 -08:00
tags:
- dist101
- lowerbound
author: Ittai Abraham
---

In this third post, we conclude with the celebrated Fischer, Lynch, and Paterson impossibility result from 1985. It is the fundamental lower bound for consensus in the [asynchronous model](https://decentralizedthoughts.github.io/2019-06-01-2019-5-31-models/).

**[Theorem 1 (FLP85)](https://groups.csail.mit.edu/tds/papers/Lynch/jacm85.pdf)**: Any protocol $\mathcal{P}$ solving consensus in the *asynchronous* model that is resilient to even just one crash failure must have an infinite execution.


* **Bad news**: Deterministic asynchronous  consensus is *impossible*.
* **Good news**: With randomization, asynchronous consensus is possible in constant expected time. See [this paper](https://research.vmware.com/files/attachments/0/0/0/0/0/7/8/practical_aba_2_.pdf) for a recent result. Note that randomization does not circumvent the  existence of a non-terminating execution, it just reduces the probability measure of this event to have [measure zero](https://en.wikipedia.org/wiki/Almost_surely).



This post assumes you are familiar with the [definitions of the first post](https://decentralizedthoughts.github.io/2019-12-15-consensus-model-for-FLP/) and with Lemma 1 that we proved in the first post:


**Lemma 1: (Lemma 2 of FLP85)**: $\mathcal{P}$ has an initial uncommitted configuration.

Recall that given a configuration $C$ there is a set $M$ of pending messages. These are messages that have been sent but not delivered yet. For $e \in M$ we write $e=(p,m)$ to denote that party $p$ has been sent a message $m$. Also recall that an [uncommitted configuration](https://decentralizedthoughts.github.io/2019-12-15-consensus-model-for-FLP/) is a configuration where no party can decide because the adversary can still change the decision value.


Given an initial uncommitted configuration, our goal will be to build an infinite execution such that:
1. The sequence is uncommitted: every configuration of the infinite execution is uncommitted.
2. The sequence is fair: every message sent is eventually delivered.

To prove the theorem we will prove the following technical Lemma:

**Lemma 2: Uncommitted Configurations Can Always be Extended ([Lemma 3 of FLP85](https://groups.csail.mit.edu/tds/papers/Lynch/jacm85.pdf))**: If $C$ is an uncommitted configuration and $e=(p,m)$ is any pending message, then there exists some $C \stackrel{\pi}{\rightsquigarrow}  C' \xrightarrow{e=(p,m)} C''$ such that $e \notin \pi$ and $C''$ is uncommitted.

#### Proof of Theorem 1 from Lemma 1 and Lemma 2:

Start with Lemma 1, to begin with, an uncommitted configuration. Repeat Lemma 2 infinitely often; each time apply it to the pending messages in a FIFO order. Clearly, from Lemma 2, the sequence is uncommitted. For fairness, due to FIFO, a message $e$ that has $\|M\|$ pending messages before it will be derived after at most $\|M\|+1$ applications of Lemma 2.


#### Proof of Lemma 2:

Recall the **proof pattern** for showing the existence of an *uncommitted configuration*:
1. Proof by *contradiction*: assume all configurations are either 1-committed or 0-committed.
2. Find a *local structure*: two adjacent configurations $C$ and $C'$ such that $C$ is 1-committed and $C_0$ is 0-committed.
3. Reach a contradiction due to an indistinguishability argument between the two adjacent configurations, $C$ and $C'$ using the adversary's ability to crash one party.


**Proof of Lemma 2** follows this pattern exactly:
1. The contradiction of the statement of Lemma 2 is that: for all $C'$, such that  $C \rightsquigarrow C'$, let  $C' \xrightarrow{e=(p,m)} C''$, then either $C''$ is 1-committed or $C''$ is 0-committed ($C''$ is not uncommitted).
2. Define two configurations $X,X'$ as *adjacent* if $X \xrightarrow{e'=(p',m')} X'$ and $e'$ is a pending message in $X$.

    *Claim*: there must exist two adjacent configurations $Y \xrightarrow{e'} Y'$ and a pending message $e'=(p',m')$ in $Y$ such that:
    1. $C \rightsquigarrow Y \xrightarrow{e} Z$ and Z is 1-committed.
    2. $C \rightsquigarrow Y \xrightarrow{e'} Y' \xrightarrow{e} Z'$ and $Z'$ is 0-committed.

    *Proof*: seeking a contradiction, assume a value $b \in \{0,1\}$ such that for *any* $C \rightsquigarrow Y$ and *any* $e'$ such that $Y \xrightarrow{e'} Y'$ it is the case that  $Y \xrightarrow{e} Z$ and $Y' \xrightarrow{e} Z'$ are committed to the same value $b$. This implies that *all*  $C \rightsquigarrow C'$ have the property that  $C' \xrightarrow{e=(p,m)} C''$ is $b$-committed.

    Since $C$ is uncommitted, there must exist $C \stackrel{\pi}{\rightsquigarrow} D$ such that $D$ is $(1-b)$-committed. This is a contradiction to the above (both if $e \in \pi$ and if $e \notin \pi$).


3. Let $Y,Y'$ be these two adjacent configurations. There are two cases to consider about $e=(p,m)$ and $e'=(p',m')$:

    3.1. Case 1 (trivial case): $p \neq p'$. This implies that processing $e$ and then $e'$ will lead to a different outcome than processing $e'$ and only then $e$. But since $e,e'$ reach different parties there is no way to distinguish these two worlds.

    Formally $Y \xrightarrow{e=(p,m)} Z$ is 1-committed and so  $Y \xrightarrow{e=(p,m)} Z \xrightarrow{e'=(p',m')} Z''$ is  1-committed. But $Y \xrightarrow{e'=(p',m')} Y' \xrightarrow{e=(p,m)} Z'$ is 0-committed. This is a contradiction because $Z''$ and $Z'$ have exactly the same configuration and pending messages.


    3.2. Case 2:  $p=p'$. This implies  that the committed value must change between the world where $p$ receives $m$ before it receives $m'$ relative to the world where $p$ receives $m'$ before it receives $m$! But what if $p$ crashes? These two worlds will be indistinguishable to the rest of the parties! Moreover, $p$ does not need to crash; it can just be slow!

    Formally, consider some execution where party $p$ crashes at $Y$.  So there must be some $Y \stackrel{\sigma}{\rightsquigarrow} D$ where $D$ is a deciding configuration and $\sigma$ does not contain party $p$. But if party $p$ was just slow then $Y \stackrel{\sigma}{\rightsquigarrow} D \xrightarrow{e} D'$ must be a configuration where all parties other than $p$ have decided.

    Since $Y \xrightarrow{e} Z$ is 1-committed then $Y \xrightarrow{e} Z \stackrel{\sigma}{\rightsquigarrow} D'$ must be 1-committed.


    Since $Y \xrightarrow{e'} Y' \xrightarrow{e} Z'$ is 0-committed then  $Y \xrightarrow{e'} Y' \xrightarrow{e} Z' \stackrel{\sigma}{\rightsquigarrow} D''$ must be 0-committed.

    For parties other than $p$, $D'$ and $D''$ are indistinguishable. This is a contradiction.

This completes the proof of Lemma 2, and that completes the proof of the FLP Theorem.


<p align="center">
    <img src="/uploads/FLP post 3 - lemma 2.png" width="600" title="lemma 2">
</p>


**Discussion.**
We started from an uncommitted configuration (Lemma 1) and then showed that we could extend this to another uncommitted configuration infinitely many times and do this while eventually delivering every pending message  (Lemma 2).

This proof is non-constructive; it shows that an infinite execution must exist. Using randomization, there are protocols that are *almost surely terminating* (their probability measure of terminating is one). There exist asynchronous consensus protocols that terminate in an expected constant number of rounds. More on that in later posts.

**Acknowledgment.** We would like to thank Nancy Lynch, Kartik Nayak, Ling Ren, [Nibesh Shrestha](https://twitter.com/NibeshShrestha1) for helpful feedback on this post.


Please leave comments on [Twitter](https://twitter.com/ittaia/status/1206298743823355905?s=20)
