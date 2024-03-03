+++
draft = false
date = 2024-03-02T17:43:44-08:00
title = "Logic Tokenizer"
description = ""
slug = ""
authors = []
tags = ["essay"]
categories = []
externalLink = ""
series = []
+++


I watched [Andrej Karpathy's video on tokenization](https://www.youtube.com/watch?v=zduSFxRajkE) and decided to build a tokenizer based on a rapper I enjoy: [Logic](https://music.youtube.com/channel/UCKC7Xqy0UXT370RvFR2kXYg).

The tokenizer is hosted on hugging face so [please have a try!](https://huggingface.co/spaces/clamepending/logic_tokenizer)

I manually copy-pasted the lyrics of the top 10 songs from logic into a text file, and then ran the BPE algorithm to find a tokenization. I set the vocab size to 512 with the original being 256 (utf-8).

One funny thing I noticed was usually the compression ratio is below 2, but if you input a rap sounding line (maybe with some profanity), the compression ratio goes to around 3. 

Thats expected because the training set has lots of that kind of data, but still pretty funny.

