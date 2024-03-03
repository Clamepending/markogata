+++
draft = false
date = 2024-03-02T17:43:44-08:00
title = "Logic_tokenizer"
description = ""
slug = ""
authors = []
tags = ["essay"]
categories = []
externalLink = ""
series = []
+++


I watched [Andrej Karpathy's video on tokenization](https://www.youtube.com/watch?v=zduSFxRajkE) and decided to build a tokenizer based on a rapper I enjoy: Logic.

The tokenizer is hosted on hugging face so [please have a try!](https://huggingface.co/spaces/clamepending/logic_tokenizer)

I manually copy-pasted the lyrics of the top 10 songs from logic into a text file, and then ran the BPE algorithm to find a tokenization.

One funny thing I noticed was usually the compression ratio is below 2, but if you input a rap sounding line (maybe with some profanity), the compression ratio goes to around 3. 

Thats probably expected because the training set has lots of that kind of data.

