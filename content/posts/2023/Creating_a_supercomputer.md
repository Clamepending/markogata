+++ 
draft = true
date = 2023-07-18T20:04:29-07:00
title = "Supercomputer and Gene Connection Finding"
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

In summer 2023 I did an internship at [Nobias Theraputics](https://www.nobiastx.com/). They develop computational tools for drug discovery. The company culture is very relaxed and supportive. I would strongly recommend applying if any student is looking for a challenging internship in biotech!

I will talk about building a supcercomputering cluster and making a Gene Connection Finder for them in this post.

To read about my failed projects during this internship, [click here.](/content/posts/2023_Internship_fails.md)

# Supercomputer

## The setup

Nobias has 4 desktop workstations connected via ethernet. Each workstation has two 3090 GPUs.

Many interns and employees SSH into a workstation to run code, but Nobias would eventually like to train bigger models that either don't fit into a single GPU or would take a long time to train on one GPU.

So the task is to build and test parallel computing on all 4 workstations (8 GPUs).

In other words, build a HPC cluster.

## FSDP vs DDP vs Nothing

My first challenge was to use both GPUs on a single machine. 
Luckily, there was a lot of work done by pyTorch on using multiplle GPUs, so the implimentation in code was extremely simple.

```python
    torch.cuda.set_device(rank)

    model = Net().to(rank)

    model = FSDP(model)

```

I followed [this tutorial.](https://pytorch.org/tutorials/intermediate/FSDP_tutorial.html)

The compute time results were follows:



## SLURM

in progress...



# Gene Connection Finder

The aim of this project is to find the strongest GENE patth from a certain gene to another.

Example: I want to know the best gene pathway from ABCA9 to MTOR:

Output of program:

'ABCA9' => 'DEDD' => 'CASP3' => 'MTOR' with total connection strength 0.8764361955555557


The connection strengths between genes were calulated with a Neural Network and provided by an api (although it is super slow...).

So the problem was implimenting Dijkstra's algorithm withthe Gene strength API.

## speed!

My first implimentation was extremely slow (ran for 30 minutes and still didnt com eup with an answer), so I had to make some improvements.

Here are the ones that worked:
- downloading connection strengths so that repeated calls for connection strengths would be quicker (bypass the API and use local csvfile instead)
- having a certainty threshold
- having intelligent search direction

Intelligent search direction means chosing the start ad end Gene smartly. For example lets input ABCA9 and MTOR: MTOR has many connections and so Dijkstra's would take an extremely long time starting at MTOR compared to starting at ABCA9.

A real test I ran was follows:

```python
getStrongestPath("ABCA9", "MTOR")
```
took 2.3 seconds

```python
getStrongestPath("MTOR", "ABCA9")
```
took 90 minutes


I found the API calls and downloading gene connectiond that was taking the most time, so I made a script that downloaded gene connection data and made a hashmap so that accessing connections became much faster.

Doing this changed runtime from 90 minutes to under a minute.

I also modified the programto take in multiple arguments for gene sources and gene targets to find the strongest path between any two pairs.

The algorithm to do that was just a nested for loop (M*N) where N and M are the sizes of the source and target genes.















