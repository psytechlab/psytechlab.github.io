---
title: Measure machines as a human. Evaluating language skills of large language models with neuropsychological tools
use_math: True
layout: post
authors: 
- Igor Buyanov
- Darya Yaskova
- Nafisa Valieva
- Ekaterina Mazuruna
---

**Abstract**: In this work, we examine the task of evaluating large language models (LLMs) through the lens of neuropsychology. The recent tendency to view modern large language models as possessing general artificial intelligence makes it worthwhile to look for evaluation strategies in the sciences that assess natural intelligence. Specifically, we adapt a widely used procedure by which neuropsychologists assess children's language skills that reflect grammatical reasoning abilities. We extend it to more complex cases and test it on adults. We show how to automatically evaluate the model's test results. Experiments with several LLM families show that sufficiently large LLMs outperform the human baseline. We analyze the results, revealing differences in the performance patterns of humans and models. We release the dataset, models, and code as open source.

Authors: Igor Buyanov, Darya Yaskova, Nafisa Valieva, Ekaterina Mazurina

Introduction
============

Large language models (LLMs) have revolutionized the field of natural language processing. Trained on volumes of text comparable to the scale of the Internet, they achieve high performance on many tasks. The key input for these models is simply the initial words, called prompts, that steer them toward a particular task to solve.

Nevertheless, these models can frequently deviate from the target task and easily lose context. To address this, researchers applied a method called Reinforcement Learning from Human Feedback (RLHF) {% cite Ouyang2022TrainingLM %}, which fine-tunes these models so that their answers are rated highly by humans. The result was ChatGPT, a chatbot that can maintain human-like conversation and display impressive abilities and knowledge. Since ChatGPT, many other LLMs have emerged. Recently, the agentic paradigm has pushed the idea that "LLMs can be AI" even further.

Some researchers argue that modern LLMs go far beyond mere probabilistic pattern matching and show sparks of artificial general intelligence, since one can simply state a goal and the model attempts to solve it, even though it was never explicitly trained for that specific task {% cite Bubeck2023SparksOA %}. Another example is the prompting technique called chain-of-thought, which forces an LLM to explain step by step how it reaches a particular conclusion {% cite Wei2022ChainOT %}. Again, this ability was not trained directly, yet this multi-step reasoning resembles how humans actually reason.

Building on this, researchers propose testing the cognitive abilities of such LLMs as is done for humans. We take a similar approach and seek inspiration in neuropsychology. Neuropsychology is a composite scientific discipline aimed at studying the brain mechanisms responsible for particular mental processes.

The term "neuropsychology" first appeared in the work of D. Hebb {% cite Mikadze2011MethodologyON %} {% cite Mikadze2019ARLA %}. Later, its theoretical foundations were developed by representatives of narrow localizationism and anti-localizationism {% cite Mikadze2011MethodologyON %} {% cite Mikadze2019ARLA %}. Neuropsychology became widespread in the second half of the 20th century because precise instruments became available and many injured people needed help after global conflicts.

A. R. Luria, a prominent figure in the field of neuropsychology {% cite Haggbloom2002The1M %} and inspired by the earlier works of L. S. Vygotsky and P. K. Anokhin, developed a theory of the systemic dynamic localization of a person's higher mental functions, the so-called "systems approach" {% cite Luriia2019ContemporaryNA %}. In this theory, complex forms of mental activity are considered functional systems provided by the work of the entire brain. At the same time, each part of the brain makes a specific contribution to the construction of these systems.

Since we focus on LLM language skills in this paper, we take inspiration from Luria's ideas. In particular, in his book on the relationship between cognitive processes and language abilities {% cite Danks1982LanguageAC %}, he described semantic aphasia — a difficulty in understanding logical-grammatical relationships. Based on this description, contemporary diagnostic tools were developed {% cite Mikadze2011MethodologyON %} {% cite Mikadze2019ARLA %}. We hypothesize that we could adapt these tests for modern language models to see whether they can achieve normal, human-level performance, thus linking LLM evaluation to neuropsychology. This becomes especially relevant given research suggesting that language models exhibit some aspects of cognitive abilities.

In this paper, we experiment with a test of the grammatical abilities of language models based on a neuropsychological prototype. We conducted a survey to obtain a human baseline and compare it with language models of different sizes. We also show how to automatically estimate the quality of the responses.

Related work
============

Researchers continually create datasets to measure progress. Often the goal is to beat a human baseline included in the dataset with some model. Frequently, datasets used by the community mirror a particular human cognitive ability. For example, SNLI {% cite Bowman2015ALA %} and its extension MNLI {% cite Williams2017ABC %} are designed to measure a model's text-understanding ability through the recognizing-textual-entailment task. Another well-known dataset that tests text comprehension is SQuAD {% cite Rajpurkar2018KnowWY %}, which is essentially a question-answering dataset. Datasets such as STS {% cite Cer2017SemEval2017T1 %} and Quora Question Pairs {% cite Wang2017BilateralMM %} are used to test how well models capture sentence similarity. There are also several common classification datasets, such as the Stanford Sentiment Treebank 2 {% cite Socher2013RecursiveDM %} and the Corpus of Linguistic Acceptability {% cite Warstadt2018NeuralNA %}. Research on non-English languages creates resources using English datasets as prototypes. For example, RuCoLA {% cite Mikhailov2022RuCoLARC %} was recently proposed for Russian.

There are also benchmarks for text generation, such as XSUM {% cite Narayan2018DontGM %} for abstractive summarization and Gigaword {% cite Rush2015ANA %} for headline generation. Another interesting dataset is PersonaChat {% cite Zhang2018PersonalizingDA %}, where the model is evaluated by how accurately it responds given a personal description.

As multitask learning enabled models to solve several tasks simultaneously, researchers created combined benchmarks — bundling several datasets that measure different aspects of language use — in order to unify model testing and propose a common protocol. These benchmarks also come with leaderboards and software that allow researchers to compare their models in a consistent way. They include GLUE {% cite Wang2018GLUEAM %}, which consists of nine tasks, and its more challenging version SuperGLUE {% cite Wang2019SuperGLUEAS %} (there is also a RussianSuperGLUE {% cite Shavrina2020RussianSuperGLUEAR %}), as well as GLGE {% cite Liu2020GLGEAN %} for language-generation tasks. With the rise of LLMs and their enormous language capabilities, there arose a need for a benchmark covering as many tasks as possible. Recently, the collaborative benchmark BIG-bench {% cite Srivastava2022BeyondTI %} was proposed, consisting of 204 diverse language tasks. There are also field-standard benchmarks on which newly released models are evaluated, such as Humanity's Last Exam {% cite phan2025lastexam %} and MERA {% cite fenogenova2024meracomprehensivellmevaluation %}.

Speaking of datasets that mimic human-like tests, we can highlight the Cloze-like datasets, in which the task is formulated as filling a gap with words or phrases consistent with the context. This format is used to assess the language-comprehension skills of native speakers, but it can also serve as a diagnostic tool for second-language learners and children. In fact, this test is part of the neuropsychological test described below. A notable dataset built on this format is the Story Cloze Test {% cite Mostafazadeh2017LSDSem2S %}, where the model must choose the correct ending given a number of story sentences. In another work, the authors apply mental-health assessment tests to several chatbot models {% cite Shan2022MentalHA %}. They use the same questionnaires designed for people — PHQ-9, GAD-7, CAGE, and TEQ — and find that the tested models exhibit signs of mental-health problems. Recently, researchers also proposed using instruments from neuropsychology to test an LLM as if it had a functional prefrontal cortex {% cite Loconte2023ChallengingC %}. They employ a variety of tests, including verbal reasoning, cognitive estimation, metaphor comprehension, idiom comprehension, anaphoric referencing, planning, inhibition, and insight. The results show that ChatGPT achieves normal or low-normal-tier scores. The work most similar to ours is {% cite alexeyev2021recovering %}, where the authors fine-tune an LSTM-based model to delemmatize sentences. The main difference is that they train the model for a specific task, whereas we assess LLM skills that emerge during causal language modeling.

Test description
================

The original test was designed for children aged 6–11 to characterize the features of their speech development. It consists of six groups of tasks, called series, which can be verbal or nonverbal. Each series may contain several tasks. We focus on the single series — comprising four tasks — that tests the formation of the grammatical structure of speech.

The idea of this task is to create grammatically correct sentences from a list of ordered words given in their normalized form. The participant only needs to place each word into the correct morphological form to produce a grammatically correct and meaningful sentence. In total, it contains 10 sentences. The shortest sentence has three words; the longest, five. An answer receives one point if the sentence is correct, 0.5 points if the word order is broken, 0.25 points if there are missing, replaced, or inserted words, minor incorrect meaning, or grammatical inconsistency (agrammatism), and 0 for completely incorrect meaning or refusal to complete the task.

As we intend to compare LLM output with human output, we planned to conduct a survey. Because a child audience is unavailable to us, we target adults. The children's test is clearly not a suitable stimulus for adults; moreover, we found no similar test for adults. Inspired by the children's test, we created our own version by making the tasks more complex through longer sentences and by shuffling some of them. We are aware that a self-constructed test lacks clinical validation, but, given the prototype, we hypothesize that our version can at least be used to compare grammatical skills, without any clinical claims.

We designed our version with the participants' abilities and willingness to complete the survey in mind. Overly long sentences — especially shuffled ones — could quickly exhaust participants and make them abandon the survey. We therefore base task complexity on the well-known working-memory-capacity rule {% cite Miller1956TheMN %}, which states that working memory can hold 7 ± 2 elements. Task complexity is thus defined by the number of words a person must manipulate, following this rule. We consider 4–6 words an easy task, 7–9 a medium task, and 10–12 a hard task.

We compose the tasks by sampling 4 examples from the easy group, 6 from the medium group, and 8 from the hard group, shuffling half of the examples in the medium and hard groups. The motivation for this sampling is to give participants some practice, since they do not usually perform such transformations; this ordering helps prevent them from abandoning the survey. In total, we have 18 tasks.

As the data source, we use the SynTagRus {% cite Boguslavsky2016SynTagRusA %} dataset, which provides annotation at different linguistic levels. It also includes word lemmas, so it fits our needs perfectly. When sampling sentences for the tasks, we manually validate them and remove any non-neutral sentences — those touching on politics, race, nationality, sexual orientation, or other ethically sensitive topics. We also exclude sentences with archaic or specialized vocabulary, direct speech, and historical facts. When choosing sentences for the medium and hard groups, we try to include compound sentences.

The full list of tasks is given in Appendix C.

Estimation methodology
----------------------

Since we change the task, we also modify the original estimation scheme. First, we drop the word-order condition, because a given token sequence can produce several valid sentences. We require only that the token lengths of the submitted and original sentences match. We do not perform token-by-token comparison in cases where token lengths match but the vocabulary sets differ. This would require an accurate lemmatizer; otherwise, its errors could seriously affect the estimation, since a full vocabulary match would be required, and a single lemmatizer error would cause a complete mismatch. We leave this for future work.

Second, we check whether the produced sentence violates grammatical rules. During manual examination of the generated examples, we discovered that some sentences, though grammatically correct, are illogical or have an unusual, Yoda-like word order. We explicitly marked such sentences and included these criteria in our estimation. In our methodology, we binarize the scores so that each criterion is either 1 or 0. We made this simplification to ensure consistency in post-survey annotation. In summary, we have four criteria:

- grammatical correctness — represents the morphosyntactic correctness of the sentence;
- logical correctness — represents the semantic coherence;
- order normality — represents the correct syntagmatic organization of the sentence;
- sequence equality — represents whether the sentence was assembled solely from the given words.

To assess the individual criteria automatically, we use classifiers trained on manually annotated data collected from humans. We split the dataset into train (70%) and test (30%) at the annotator level, meaning that all answers from a single worker fall into the same split. We then train a RuBERT {% cite zmitrovich2023family %} classifier for each criterion. We also tried an LLM-as-a-judge approach, but it did not match the BERT-based models.

The final per-example score is the sum of the individual estimates from the set $E$. The performer-level score $S$ is computed as the average of the individual task scores. $$S = \frac{1}{n}\sum_{j=1}^{n}\sum_{i}E_{ij}$$

One can notice that our estimation task is close to what the RuCoLA dataset {% cite @mikhailov-etal-2022-rucola %} targets, namely linguistic acceptability. We test how well ruRoBERTa-large-rucola[^1], trained on RuCoLA, aligns with our dimensions.

Human data annotation and estimators training
=============================================

We collected data mainly via the Yandex.Toloka platform, and via Google Forms for the pilot study. The instructions are given in Appendix A. We collected responses from 108 participants, yielding 1,944 answers in total. Some workers declined to answer particular questions, responding with only a few symbols or the first few tokens (we consider fewer than four tokens a refusal to answer). There are 216 such cases in total. We do not exclude them from the dataset. We manually assessed each answer for grammatical correctness, logical correctness, and order normality. The assessment was performed by an undergraduate linguist on our team, who led the annotation process together with two other undergraduate linguists. While annotating, they answered a straightforward question: whether the human answer is (1) grammatically correct, (2) logical, and (3) has a normal (natural) word order.

The Krippendorff's alpha values for the fully annotated dataset on the separate criteria are as follows: 0.704 for grammatical correctness, 0.736 for logical correctness, and 0.774 for order normality. The final class is determined by majority voting across annotators.

The train split contains 1,350 examples from 75 workers, while the test split contains 594 examples from 33 workers. The classifiers were trained for 5 epochs with a learning rate of 2e-5. For the LLM-as-a-judge, we test gpt-oss-120b and claude-sonnet-4. The results for all methods on the test split are presented in [Table 1](#tab_1).

<a id="tab_1"></a>
**Table 1:** *Results of criteria estimation performance.*

| Models                  | F1-micro | F1-macro | F1-weighted |
|-------------------------|----------|----------|-------------|
| Grammatical correctness |          |          |             |
| ruBERT                  | 0.88     | 0.84     | 0.88        |
| RuCoLA                  | 0.84     | 0.81     | 0.85        |
| gpt-oss-120b            | 0.82     | 0.73     | 0.83        |
| claude-sonnet-4         | 0.82     | 0.71     | 0.82        |
| Logical correctness     |          |          |             |
| ruBERT                  | 0.92     | 0.88     | 0.92        |
| RuCoLA                  | 0.87     | 0.84     | 0.87        |
| gpt-oss-120b            | 0.84     | 0.77     | 0.83        |
| claude-sonnet-4         | 0.78     | 0.64     | 0.76        |
| Order normality         |          |          |             |
| ruBERT                  | 0.91     | 0.85     | 0.91        |
| RuCoLA                  | 0.83     | 0.78     | 0.84        |
| gpt-oss-120b            | 0.82     | 0.71     | 0.82        |
| claude-sonnet-4         | 0.84     | 0.62     | 0.81        |


Clearly, the ruBERT classifiers perform best in all cases. We also test whether the BERT-based automatic score aligns with the human score using Pearson correlation on the test set. The resulting value is 0.880, with a p-value close to zero.

We also observe that the RuCoLA model is not far behind ruBERT, given that it is a different model entirely. We compute metrics using RuCoLA's predictions against all three sets of true labels. This suggests that RuCoLA encodes all three dimensions reasonably well; however, since the dedicated classifiers outperform it by 3–8 points, we use them instead.

Models to be evaluated
======================

To capture the effect of increasing model size, we use several models that vary in parameter count. We use the following model families:

- Claude-3-haiku, Claude-opus-4, Claude-sonnet-4.5, Claude-opus-4.8
- GPT-3.5-turbo, GPT-4o, GPT-5, GPT-5.4
- Qwen3.5-9B, Qwen3.5-27B, Qwen3.5-35B-A3B, Qwen3.5-122B-A10B, Qwen3.5-397B-A17B
- GigaChat-2, GigaChat-2-Max, GigaChat-2-Pro
- Yandex-lite, Yandex-5.1

All models were run with the same prompt at temperature 1. The prompt is given in Appendix B.

For a fairer comparison, we ran each model five times. Because these models are non-deterministic, we would ideally like each run to produce a reasonable response, just as different people would each give a reasonable answer. Since we require the answer in a predefined format, and the model may err in either formatting or content, we allow up to 10 attempts to return a correctly formatted answer. The final per-criterion score is aggregated across runs by majority vote (i.e., 1 if the criterion holds in more than half of the runs).

Results
=======

<a id="fig_1-overall_score"></a>
**Fig. 1.** *The overall score across the models. The red line is the human baseline.*
![The overall score across the models. The red line is the human baseline.](/assets/images/measure_machines_as_human/overall_score.png)



The overall evaluation result is shown in [Figure 1](#fig_1-overall_score). The human baseline is the red line, computed as the average over the test set (594 samples), equal to 0.728. The error bars on each bar represent the standard deviation of the score across runs. We see a clear pattern: the larger the model (we assume that newer closed models are also larger), the better the result. While most models outperform the human baseline, we highlight Claude Opus 4.5, which completes the test perfectly. The standard deviation is notably large for most models, implying that task success is unstable across runs. Overall, we conclude that sufficiently large language models — at the time of writing — have a good command of Russian grammar and can use it to construct meaningful sentences.

Detailed analysis
-----------------

To gain a better understanding, we can zoom in on the results in several ways. [Figure 2](#fig_2-score_deps_on_is_shuffled) shows the scores depending on whether a task was shuffled. Note that the human baseline is now also represented as bars.

<a id="fig_2-score_deps_on_is_shuffled"></a>
**Fig. 2.** *The scores across the models depending on whether the task is shuffled.*
![The scores across the models depending on whether the task is shuffled.](/assets/images/measure_machines_as_human/score_deps_on_is_shuffled.png)


For humans, we see the expected result that the score on non-shuffled tasks is much higher than on shuffled ones. This pattern generally holds for the models as well. As we can see, all models solve the non-shuffled tasks better than humans, sometimes perfectly and without any variation. The difference comes from the shuffled tasks, and the average overall score and its deviation most likely depend heavily on how the models handle shuffled tasks. Note that only three models have an average score below the human baseline.

<a id="fig_3-score_deps_on_complexity"></a>
**Fig. 3.** *The scores across the models depending on the task complexity.*
![The scores across the models depending on the task complexity.](/assets/images/measure_machines_as_human/score_deps_on_complexity.png)

[Figure 3](#fig_3-score_deps_on_complexity) shows the scores depending on task complexity. As expected, human performance on the easy tasks is much higher than on the others. Note that performance on the medium and hard tasks is very close. Surprisingly, the opposite pattern holds for almost all models: the more complex (i.e., longer) the task, the better the model performs. We hypothesize that this is related to the probabilistic nature of the generation mechanism. Short sequences make the probability estimates unstable, leading to erroneous token selection during sampling. This phenomenon may highlight a difference in text processing between humans and models. It also indicates that our score-calculation formula is biased toward hard tasks and does not account for differing task lengths. Plain averaging favors those who perform better on more complex tasks.

We also investigate which samples receive the lowest scores across models. Using the ID column from the Appendix table, the three worst samples are 7 (7 times), 3 (4 times), and 5 (3 times). In these cases, the models often fail to generate a grammatically correct or meaningful sentence. Word insertion here is treated as an attempt to "complete" the sentence. Regarding the worst sample, 7, we believe the main cause is another unrecognized transformation: "человек" – "люди" (man – people). Most models try to build the sentence with "человек" (man) and fail.

At the model level, a notable issue is that models often violate the vocabulary-equality rule, tending to insert or remove words. Surprisingly, we did not specify this condition explicitly in the annotation instructions or in the prompt; specifying it in the prompt would probably improve the results. Models also fail to recognize transformations of pronouns: "он" – "его" (he – him), "мы" – "нам" (we – us), "тот" – "той" (that [masc.] – that [fem.]), etc. In some samples this is acceptable, as in sample 17 ("в деканате **нам** сказали" – "**мы** сказали в деканате"; "the Dean's office **told us**" – "**we told** the Dean's office"), but not in others.

Conclusion and future work
==========================

In this work, we adapt a test from neuropsychology for large language models in order to obtain a grounded comparison of language skills with humans. We focus on one of these skills, namely the ability to arrange lemmatized words into a morphologically coherent sentence. This task probes the ability to correctly recognize the relations between words. Starting from the basic setup, we modify the tasks to make them more complex. We conduct a survey to obtain a human baseline and to gauge whether this setting is feasible. We then evaluate several models of varying size, showing that sufficiently large LLMs have a good command of Russian grammar as measured by this test. The test also provides preliminary evidence of a difference in text processing between humans and LLMs.

The promising results of automating the score calculation suggest that other tests from neuropsychology and related fields can be adapted. Moreover, the rise of multimodal models allows the use of non-verbal tests.

The main limitation of this work is that the survey participants were in an uncontrolled state, which may have lowered their attention and thus affected the results. This is supported by the fact that 11% of tasks were left unanswered. The fact that we binarize the labels could also lead to a loss of granularity. With more resources devoted to the annotation process, one could obtain more fine-grained estimates. From an automation standpoint, one could achieve better results with an LLM-as-a-judge by investing more time in prompt refinement. Finally, our self-constructed test lacks clinical validity.

In future work, one could carry out an in-depth linguistic analysis of the tasks and their corresponding answers. Based on such an analysis, one could identify why models fail, or design a new set of tasks grounded in specific linguistic phenomena. Furthermore, given that the RuCoLA model approximates the considered linguistic dimensions fairly well, one could save time on estimators and concentrate on the scale and diversity of the test set. We hope this work also inspires further research into the potential of existing cognitive-testing methods, and into adapting this approach to other morphologically rich languages such as German or Finnish.

References
==========

{% bibliography %}

# Appendix A. The instruction for the human workers

```
Перед вами список предложений. Слова в них были переведены внеопределённую форму: глаголы отвечают на вопрос "что делать",существительные и прилагательные стоят в именительном падеже и т. д.Ваша задача переписать предложения так, чтобы они были правильными иимели смысл. Например:

кошка сидеть на стол и играть с мяч -\> кошка сидит на столе и играет смячом.

Важные детали:

- В некоторых предложениях порядок слов изменен. Пример: купить дочка вкусный в папа шоколадка магазин купить -\> папа купил дочке вкусную шоколадку в магазине

- Можно согласовывать в разных временах и количествах. Главное, чтобы предложение было правильным с точки зрения русского языка и имело смысл.

- Начальная форма глагола может быть и причастием, и деепричастием. Пример: забыть об все, он помчаться на встреча к он -\> забыв обо всем, он помчался на встречу к нему.

Введите ваш вариант в поле под списком.
```
# Appendix B. The LLM prompt for getting answers for the test

```

Ты учитель русского языка. Тебе дано предложение, в котором словаприведены к начальной форме: глаголы --- в неопределённой форме,существительные и прилагательные --- в именительном падеже и т. д.Задача: переписать каждое предложение так, чтобы оно было грамматическиправильным и осмысленным на русском языке. Пример: \"кошка сидеть настол и играть с мяч\" → \"кошка сидит на столе и играет с мячом\" Важно:

- В некоторых предложениях порядок слов может быть нарушен. Пример: \"купить дочка вкусный в папа шоколадка магазин купить\" → \"папа купил дочке вкусную шоколадку в магазине\"

- Допустимо менять время, число и грамматические формы слов, если итоговое предложение звучит естественно и правильно.

- Начальная форма глагола может соответствовать причастию или деепричастию. Пример: \"забыть об все, он помчаться на встреча к он\" → \"забыв обо всем, он помчался на встречу к нему\"

Верни результат строго в формате JSON:

\"results\": \"\<исправленное предложение\>\"

Требования к ответу:

- Только валидный JSON

- Без пояснений, комментариев и дополнительного текста

- Для каждого входного предложения должен быть один объект в массиве \"results\"

Слова для предложений: {words}

```

# Appendix C. The stimuli materials. The shuffling flag was omitted.

| ID | Task | Original sentence | Complexity |
|---|---|---|---|
| 1 | добавить и бы нехитрый пара . мысль | и добавил бы пару нехитрых мыслей. | easy |
| 2 | то ни ни . другой не произойти , | ни того, ни другого не произошло. | easy |
| 3 | . много решение быть вопрос наметить путь | были намечены пути решения многих вопросов. | easy |
| 4 | свой пусть сначала устранить недостаток . | пусть сначала устранит свои недостатки. | easy |
| 5 | но часто сгладить . оставаться все что , время как-то надежда | но часто остается надежда, что время как-то все сгладит. | medium |
| 6 | вскоре весь команда перейти на он использование . | вскоре все команды перешли на его использование. | medium |
| 7 | с , который человек . сострадательный время браться но помогать найтись | но со временем нашлись сострадательные люди, которые взялись помогать. | medium |
| 8 | чижик с интерес смотреть на он , склонить голова . | чижик с интересом посмотрел на него, склонив голову. | medium |
| 9 | он неожиданно разлучить с тот , который он любить . | его неожиданно разлучили с той, которая его любила. | medium |
| 10 | . выносливость он повышение к физический применять также для нагрузка | его применяют также для повышения выносливости к физическим нагрузкам. | medium |
| 11 | конец в сезон . производить выявлять победитель номинация и очко в оба подсчет | в конце сезона производится подсчет очков и выявляются победители в обеих номинациях. | hard |
| 12 | между условие и . что на тот ранее же , себя они соревноваться | они соревнуются между собой на тех же условиях, что и ранее. | hard |
| 13 | для гран-при Монако разработать специальный промежуточный шина , отличаться повышенный мягкость . | для гранпри монако разработаны специальные промежуточные шины, отличающиеся повышенной мягкостью. | hard |
| 14 | в итог никакой особый причина относиться к мы плохо у они нет . | в итоге никаких особых причин относиться к нам плохо у них нет. | hard |
| 15 | поэтому донорский кровь при начаться тромбоз мочь ухудшить состояние больной . | поэтому донорская кровь при начавшемся тромбозе может ухудшить состояние больного. | hard |
| 16 | костюм оркестровый и старательно свой яма . чистить в пойти черный он музыкантский | он старательно почистил свой черный музыкантский костюм и пошел в оркестровую яму. | hard |
| 17 | в деканат мы сказать , что она уйти с работа и куда-то уехать . | в деканате нам сказали, что она ушла с работы и куда-то уехала. | hard |
| 18 | Марс возможно в , . на стать биорегенерация жизнеобеспечение метод будущее основной | возможно, в будущем биорегенерация станет основным методом жизнеобеспечения на марсе. | hard |

[^1]: <https://huggingface.co/RussianNLP/ruRoBERTa-large-rucola>
