--- 
layout: post 
title: How we collected a dataset for developing an ML tool that helps save lives
use_math: false
--- 

![](/assets/images/presui_antisui_dataset_methodology/poster.png)

_Originally posted on [Habr](https://habr.com/ru/companies/mts_ai/articles/985412/) in the MWS AI blog. As the authors, we are duplicating it in our blog._

Hello, Habr! This post is about a specific dataset designed to solve a very important task — developing an ML tool that helps timely identify preconditions and prevent suicides. My team and I from "Psytechlab", specializing in AI solutions for psychotherapy, collected it in the evenings. This is a dissertation project, it's not part of my responsibilities within my work at MWS AI, but the experience gained at the company became the foundation without which it wouldn't exist.

We wrote a [scientific article](/assets/pdfs/dataset_methodology_presui_antisui.pdf) on creating this dataset. If you use our dataset, please cite it.

# Let's start with the context. Why this project is so important
According to WHO data, the world annually loses more than 700 thousand people due to suicide. Imagine, an entire city, and not a small one, disappears every year. I haven't been able to find fresh statistics for Russia yet, but in 2019 there were 17 thousand suicide cases, and in 2022 — 13.5 thousand (this is Rosstat data). The good news is that the number of such tragedies in our country is decreasing year by year: since the peak that occurred in 1994 and amounted to a record 61.8 thousand cases — a drop of almost six times! But we really want this number to fall even faster. And in general, for it to be zero.

This is what our initiative is aimed at. We want to help provide timely support to people who are on the edge. There are specialized NGOs that search for such people on social networks and help them within their rights and capabilities, as well as cooperating with the Ministry of Internal Affairs and FSB. And the task of the 'Psitechlab' team is to facilitate this search.

# How it works
In social networks, there are regular users whose texts are mostly irrelevant to our topic. And there are those who show signs of suicidal tendencies — our target audience. There are also bots, fakes, and people who don't show obvious signs, and we don't yet know how to work with them.

![General model of social network analysis](/assets/images/presui_antisui_dataset_methodology/1.png)

All posts on social networks are analyzed by volunteers from NGOs: based on the posts, they assign users so-called suicidal statuses. It's important to note that suicidal status is not a diagnosis, but rather a marker indicating whether attention should be paid to a specific account. If the status is high, then it's a reason to look for additional information about the user, find out if it's even a human, possibly take some measures.

The problem is that we have to analyze huge amounts of texts. According to the statistics we collected, out of 100 posts, only 20 are such that contain target information — these are various negative situations, expressions of emotions, calls for help, and so on.

# Volunteer ML
We propose a system that will help filter irrelevant posts, thereby reducing the load on volunteers and increasing their efficiency. To achieve this, we are collecting a dataset to build a machine learning model that will perform this task. Moreover, we have quite an ambitious goal: to collect 50 thousand labeled texts. This is more than any other dataset on a related topic even in English.

*Important disclaimer: every time I talk about my project, for some reason there's an impression that we're building a model that specifically predicts suicide, which is not the case. Therefore, I regularly hear the question "Did psychologists do your labeling?". No, not psychologists. We only identify texts that describe certain factors (I gave an example above). To determine that a person is writing that they feel bad or that they experienced violence or bullying, you don't need to be a psychologist. It's enough to simply have common sense. Our idea is that any person can become a volunteer. With all this disclaimer and context, let's move on directly to dataset collection.*

# How we collected the dataset
We used open sources — for example, existing datasets, as well as (surprisingly) old-school forums from the 2000s, where people discussed the topic of suicide. There people often shared their stories and sometimes received psychological support. In their posts, almost every sentence could contain some suicidal factor. And there are hundreds of such stories.

After collecting the data, we enriched it with various features: presence and type of pronouns, indications of family relationships, word count, emotions, sentiment, and so on. With these features, you can select texts for annotation so that the required classes appear with greater probability. We applied both various heuristics and existing open models. We are very grateful to the developers who made them publicly available — this is very valuable.

Interestingly: a regular sentiment model that predicts neutral, negative, or positive sentiment classified texts with suicidal thoughts as neutral. For example, the phrase 'I want to die' was rated as neutral.
For the anti-suicidal part of the dataset, which I'll tell you about a bit later, the emotion model helped us a lot. This is logical, because anti-suicidal texts, as you can guess from the name, are often associated with emotions such as joy or surprise.
So, we collected and enriched the data. Now let's move on to the heart of any dataset — the instructions.

# Instructions
Our instruction consists of just two parts: the main part and the class table.
The main part standardly described what, why, and how to annotate. It also outlined various annotation principles. Let's carefully look at the two most important principles:

* The content of the text should relate to the author. We want to predict only what relates to the text author, not what relates to a third party. Example:
    * **I** want to escape from this oppressive external world into myself.
    * **I** don't have the strength to endure this.
    * <u>He</u> has such chaos happening at home.
* Avoid unfounded interpretations. This is easier to show with an example. Read the text and say what scene it describes.

"*I stepped away briefly, and when I returned, she had managed to read our entire candid correspondence*".

Many would think that a wife or girlfriend read her partner's correspondence with a mistress with the expected outcome. According to our instruction, this could be interpreted as a pre-suicidal signal: relationships that didn't work out and broke down. However, this is incorrect because you invented the context. It doesn't directly follow from this text that the relationship fell apart. Strictly speaking, it doesn't even follow that these are romantic relationships. What if this is about a mother who read her teenage son's correspondence? Most likely, the previous or next sentence contains the complete information.

Now let's touch on the class table. More precisely, how we created it. The global goal is to annotate texts into three large groups of signals:

1. Pre-suicidal signals: factors inclining toward suicide.
2. Anti-suicidal signals: factors that conditionally preserve.
3. Irrelevant signals: most texts that are uninteresting to us and that we want to exclude from consideration.

We found several existing class systems, broke them down, and combined them so that the new classes would satisfy two conditions:
* Atomicity — a factor cannot be "broken down" into components.
* Semantic independence — texts of different classes should overlap as little as possible in meaning.

This is done so that any person who worked in any one of the systems could adapt our system to themselves.

![Mood board for different systems of suicidal factors](/assets/images/presui_antisui_dataset_methodology/2.png)

The picture above shows an example of how we analyzed existing class systems. I call this technique the "Mood Board". We simply write out all the factor cards and mark them with the same color if we think they are similar. Then we try to combine them: we discard some, add some.

Here's an example of signals we identified: red — pre-suicidal, green — anti-suicidal. In total, we got 33 pre-suicidal classes combined into seven groups, and 12 anti-suicidal classes without division into groups.

![Signal examples](/assets/images/presui_antisui_dataset_methodology/3.png)

# Test annotation
After we collected the basic class grid, we decided to conduct test annotation with our own resources. Its scheme consisted of two rounds:
1. **The first round** was performed by me and my colleague, who participated in dataset collection from the beginning.
2. **The second round** was performed by members of our team who had never seen the data and previously had no relation to our topic. This was done to model a situation when a newly arrived volunteer joins the process.

What did we discover as a result of test annotation?

First, we discovered that our texts often contain multiple classes. This is **multi-label**. This directly raises some technical questions:

* How to combine annotations when one text is annotated by more than one person?
* How to calculate the degree of agreement?

We solved the first problem with soft majority voting. We compile a unified list of all classes that people assigned, and from it we select those that occurred more than n times. On our test annotation, this approach gave good performance in terms of coverage — the number of texts that have at least one class, and the final quality of labels. With simple majority voting, too many examples simply received no label.

We solved the second problem using Krippendorff's alpha and a special MASI metric (Measuring agreement of set-valued items), which is used as a kernel. This metric is actually a Jaccard metric with a special coefficient. Out of the box, it can be calculated using — drumroll — [NLTK](https://www.nltk.org/api/nltk.metrics.distance.html#nltk.metrics.distance.masi_distance).

Another feature of our data is **subjectivity**. When we talked with team members at the feedback stage, we couldn't always dispute their class choice. That is, they assigned some class, we thought it was wrong, asked "why?", they answered, and we sort of agreed that "it seems okay". This makes our task very close to sentiment and emotions. Despite the fact that classes are defined quite well, life experience still plays a big role. Someone might ask, why didn't we specify such examples as edge cases (corner cases)? The problem is that there were many such texts and my colleague and I simply couldn't handle maintaining a long consistent list of such cases.

# Where we looked for annotators
We wrote instructions, prepared the data, and wanted to start annotation. For annotation, we usually have two paths: crowdsourcing and individual annotators. Each has its pros and cons, which are shown in the image below. We initially thought of using crowdsourcing since it's cheaper and faster, plus we had experience working with Yandex.Toloka — the most well-known crowdsourcing platform until recently.

![Comparison of crowdsourcing and individual annotators](/assets/images/presui_antisui_dataset_methodology/4.png)

The problem is that Yandex.Toloka shut down in early 2024, and Yandex.Tasks appeared in its place. What could go wrong, you might think? Well, what went wrong was that this platform doesn't work with individuals: you can't simply be a client — you absolutely need a legal entity. We spent a lot of time organizing such a legal entity.

This was happening parallel to test annotation, and when we finished it, we realized that subjectivity multiplied by low motivation in crowdsourcing wouldn't lead to anything good. So we decided to work with individual annotators. Especially since I myself worked for two years in annotation at MWS AI, where I streamlined process automation.

Among all the platforms where we looked for annotators, Freelans.ru really surprised us — we got a whole 30 responses there. We even had to choose based on cover letters. We structured the hiring process so that even people without annotation experience could learn and minimize errors as a result. Overall, as you can see, we succeeded: 27% errors at the beginning is certainly a lot, but 4.6% at the end of the process is already acceptable.

Interesting point: during the review of annotators' test tasks, we most often encountered errors related precisely to violations of those two principles I mentioned: the relationship of text content to the author and the inadmissibility of interpretations. Another curious fact: for some reason, a text with reasoning like "If I don't rest, I turn into a walking zombie," all eight people considered as an accomplished fact and assigned the corresponding class.

Our complete annotation process overall doesn't differ from any other, except that we have feedback collection and psychological venting. I'll tell you more about this later. And now, a bit about the tools.

# What tools we used
We annotated the data in Label Studio. It's a very good platform whose key feature is that it allows you to create an interface for almost any task. I haven't yet encountered a task for which I couldn't create an interface. Plus, I have extensive experience working with it. This is what the annotation interface we used looks like.

![Annotation interface in Label Studio](/assets/images/presui_antisui_dataset_methodology/5.png)

The next important question: where did we store the data? We used ClearML. If you've never encountered data loss or version confusion, that's good. To avoid encountering it in the future, use ClearML or similar platforms that allow dataset versioning. Trust me, this is a very important aspect.

# Taking care of annotators
As you know or at least might guess, the annotators were not working with content about cats and dogs, but with emotionally heavy texts. We were concerned that this might somehow affect their psychological state. Therefore, we monitored our annotators both instrumentally (using tests) and through ventilation sessions. This is an interview format where we build safe, trusting relationships and discuss things that might have happened to the annotators while they were annotating something.

During these interviews, we identified several interesting points. First, some annotators indeed had reactions to a number of texts at the beginning of their work, but by the middle, everyone managed to distance themselves from what they were reading. Moreover, by the end, some people no longer even needed these interviews. Second, it was pleasant to learn that as a side effect of the work, several annotators gained a deeper understanding of their teenage children.

# How we verified and corrected the dataset
Before discussing the quality of our dataset, let's recall that we wanted to annotate 50,000 examples. That's a lot. To ensure adequate quality measurements, we annotated the test portion of the dataset with overlap, meaning several people annotate the same example, and the final result is obtained through aggregation of individual annotations. Unfortunately, we couldn't afford to annotate the entire dataset this way, as it would have taken too much time.

To check the dataset globally, we ourselves annotated 185 examples from each batch in parallel with the main annotation process. Perfectionists are probably wondering: why not 200? This is an artifact from a validation scheme that ultimately didn't work out, and changing it would have been too costly. After completing a batch, we compared the annotations with each other. If the number of errors exceeded a predetermined threshold, we reviewed the discrepancies to assess the disputable nature of the annotation. If after such verification the number of errors still exceeded the threshold, such a batch was returned for rework.

Our error threshold was 15% of the number of checks. We formulated this number based on similar works. If we calculate the overall percentage of errors between annotations, it turns out we walked on thin ice: for the test set we got 13.99%, and for the train set — 14.55%.

When we started training models, it turned out that our anti-suicidal model was terrible in quality. We expected this, but not at the level we saw. We expected it because colleagues during test annotation noted that the anti-suicidal part was more difficult. After analyzing the situation, we decided to rebuild the classes for the anti-suicidal part and re-annotate it. How we did this, as well as how we searched for errors in the presuicidal part, we wrote about separately in our [devlog](https://psytechlab.github.io/2025/02/23/%D0%94%D0%B5%D0%B2%D0%BB%D0%BE%D0%B3-1.-%D0%A1%D0%B4%D0%B5%D0%BB%D0%B0%D0%B5%D0%BC-%D1%80%D0%B0%D0%B7%D0%BC%D0%B5%D1%82%D0%BA%D1%83-%D0%BB%D1%83%D1%87%D1%88%D0%B5.html).

Looking ahead, I'll say that the final quality of the anti-suicidal model after rebuilding the classes turned out worse than the first tests from the devlog showed, but significantly better than the original version.

# What we got in the end
In total, we collected 57,810 examples. The presuicidal part contains 38,406 examples, the anti-suicidal part contains 9,702 examples, and we also obtained 9,702 irrelevant examples. Inter-annotator agreement according to Krippendorff's alpha for the presuicidal part for the test is 0.542. Slightly more than a quarter of examples contain more than one signal. The tables below show the distribution of examples in the corresponding parts.

Distribution in the anti-suicidal part

|    Class name                                            |    Count   |
|--------------------------------------------------------------------|-------------------|
|     Presence of positive social connections                        |     1,650         |
|     Expression of love                                             |     1,384         |
|     Expression of happiness, joy, satisfaction                     |     858           |
|     Positive self-esteem                                           |     595           |
|     Expression of love / presence of positive social connections   |     486           |

Distribution in the presuicidal part

|     Class name                                                                                                 |     Count     |
|----------------------------------------------------------------------------------------------------------------|---------------|
|     Death / thoughts of death                                                                                  |     4,205     |
|     Problems in the external world / unhappy love, problems with friends, difficulties in building relationships    |     3,236     |
|     Feelings: helplessness, hopelessness, despair                                                          |     2,602     |
|     Feelings: mental emptiness, depression, melancholy, sadness                                            |     2,359     |

# Model Training and Results
It's time to talk about the model training results - what we've been working towards all this time. As a baseline, we used BERT, since over its six years of existence it has become such a peculiar 'Baseline Baselineovich' for such tasks and is very convenient to work with. We tried other models like DeBERTa and RoBERTa, but good old ruBERT showed the best results.

In the baseline version that went into the scientific paper, we did minimal preprocessing: converted texts to lowercase and removed any non-Cyrillic characters. We also filtered out classes that had fewer than one hundred examples and removed texts with multiple classes. The class structure allows us to build models at different levels of granularity (detail):

* exact — we use all available classes;
* group — we use only groups;
* binary — whether there is a signal or not;
* ternary — whether there is an antisuicidal or presuicidal signal.

Here are the results we obtained.

| Model Type   |     Granularity      |     Number of Classes   |     Precision   |     Recall     |     F1-score (macro)     |
|--------------|----------------------|-------------------------|-----------------|----------------|--------------------------|
| Presuicidal  |     Group            |     8                   |     0.65        |     0.65       |     0.65                 |
| Presuicidal  |     Exact            |     26                  |     0.61        |     0.51       |     0.53                 |
| Antisuicidal |     Exact            |     9                   |     0.70        |     0.59       |     0.63                 |
| All          |     Binary           |     2                   |     0.71        |     0.71       |     0.71                 |
| All          |     Ternary          |     3                   |     0.71        |     0.69       |     0.70                 |

By the way, our formal goal is 70 points on F1-macro for both models with three to seven classes. To achieve this, we did some magic with the class structure, and additionally, we discarded some noisy examples from the training set. By magic, we mean merging some classes into one both at the group level and among themselves. For example, we decided to keep emotion classes as a group because this is perhaps the most difficult class for the model. The classes 'thoughts about death' and 'intentions about death' were decided to be merged into one, because the second turned out to be small and the model could not grasp its essence. However, this class is very important, we could not discard it.

As a result, we were able to find a configuration where we have the maximum number of classes at a given F1 threshold of 70 points. For the presuicidal model, we got 15 classes, for the antisuicidal model - 10. It should be noted that each model also includes an antagonist class, meaning the presuicidal model can identify antisuicidal signals, and the antisuicidal model - vice versa.

# About our platform
It's not enough to just develop models. You also need to make it possible for people to use them. For this, we developed the "Kitoboy" platform with a focus on analyzing user texts. It acts as an intermediary between volunteers and models. You can upload posts to it, and the platform will collect all predictions for each post. By default, it has presuicidal and antisuicidal models connected, but any other models can be connected. Here's what the interface for viewing posts with predictions looks like:

![](/assets/images/presui_antisui_dataset_methodology/6.png)

Besides the post "feed" format, you can look at the temporal graph of predictions to assess trends in users' mood and state — a special feature of our platform. Example interface:

![](/assets/images/presui_antisui_dataset_methodology/7.png)

The platform is open-source and you can try it here: [https://github.com/psytechlab/kitoboy](https://github.com/psytechlab/kitoboy). You'll also find links to related repositories, models, and datasets there — we'd be grateful if you give it a star :)

# Future plans
After analyzing the model errors, we still have several unpleasant things that need to be fixed in the dataset. And in general, we want Krippendorff's alpha value >0.7. For this, we need to somehow improve the annotation methodology without reducing it to a lengthy list of corner cases.

Also, to keep up with the times, we want to incorporate LLM into the process. We have several ideas:
* Use LLM for summarizing texts that were identified as some kind of signals.
* Teach LLM to identify suicidal statuses with explanation. Given that models now reason out of the box, this is not difficult to do.
* Include LLM in the annotation process.

From the platform perspective, we also have ideas on where to grow and what to do:

* Create a social media parsing service. Currently, data needs to be loaded from csv.
* Implement observer notes where a volunteer can record some conclusions. This is also an interface for the LLM functions above.
* Add a role-based system for platform users with access control. In theory, there should be at least two roles: a volunteer who performs user analysis, and a supervisor who checks and evaluates the volunteers themselves.

We want to thank everyone who participated in data annotation: Zhanna Naskhulyan, Anastasia Tyukaeva, Artem Zagidullin, Irina Khmeleva, Leonid Fomin, Alina Ryabusheva, Natalia Matveeva, Tatiana Soloshenko, Denis Martynov, Natalia Soloshenko.

Let's make this world a little bit better.

