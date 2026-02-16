
--- 
layout: post 
title: Devlog 7. To Generate or Not to Generate? Intermediate Results with Synthetics for Psychology Domain
use_math: false
--- 

Our domain has one big problem — there's little data and/or it's hard to obtain. This is related to therapy ethics or medical confidentiality if the diagnosis is psychiatric. Data generation through LLMs can ease the pain of all ML engineers, but how do we know to what extent? We're trying to answer this question and in this post we'll briefly share intermediate results.

We took three datasets: sentiments, emotions, and the anti-suicidal part of our dataset. We chose these specifically because articles about them contain class descriptions. Without that, you can't compose a prompt. Well, you can, of course, but we wanted the prompt to be connected to the dataset creation instruction, to build a bridge, so to speak.

A bit more about the prompt. We tested three variants: zero-shot, few(8)-shot, and few-shot with keywords (fskw). We bet you've never heard of the third one. Because it's our brainchild. Here's the idea. No matter how hard you try, you can't squeeze various lexical features of a class into some reasonable number of examples for few-shot mode. Moreover, some works note that quality starts degrading with increasing examples. So we decided to add such class-significant words to the prompt.

We also had six models, three closed and three open, and two temperature values: 0.7 and 1.

Using each combination of these parameters, we generated for each class of each dataset no fewer unique examples than in the original dataset. There's already something to say here: some configs generated exactly as much as needed, while others generated up to 15k texts, damn it!, before getting the needed couple thousand unique ones among them. Once we generated everything, we started mixing them with different data in various ways and training classifiers.

We started simple: train only on generated data, on half real and half generated, on all real and the same amount of generated. We won't consider the second and third variants in this post (tl;dr: results are more or less the same as in the original variant where all data is real), we'll stick with the first one.

If we look at the final classifier quality by f1-macro across each configuration part, on average the best model was llama-4-maveric (remember that one?) across all three settings. Temperature change affected quality only on the anti-suicidal dataset: value 0.7 was on average 4 points better than value 1.0. Our invention fskw also showed the best results across all three datasets.

We know from the second half-and-half setting that classification quality is very close to the original where all data is real. But what quality can we achieve if we add generated data to a minimal amount of real data?

First, we found this conditional minimum. For all three datasets it turned out to be 10 percent of the real volume. Then we added generated data to these 10 percent in volumes of 25, 50, 75, and 100 percent of the original volume of each class. It turned out that already in the first case for two out of three datasets there's a sharp increase in model quality (generally, the same growth is observed between 10 and 20 percent of real data). After the takeoff, there's a less noticeable but still quality improvement. The best mix variants lag behind the original model by 3-5 points. By the way, it's far from certain that the best mix will be the variant with 100 percent generated data. The sentiment dataset was the one where this trick didn't work.

Next, we have a series of experiments with different data metrics. Our goal is to find a metric that could tell from a couple thousand model samples what's promising to use and what's not so much.

