---
title: Measure machines as a human. Evaluating language skills of large language models with neuropsychological tools
use_math: True
layout: post
authors: Igor Buyanov, Darya Yaskova, Nafisa Valieva, Ekaterina Mazuruna
---

**Abstsract**: In this work, we look at the task of evaluating large language models (LLM) through the lens of neuropsychology. The recent trend to view modern large language models as having general artificial intelligence makes it worthwhile to look for evaluation strategies in science that evaluate natural intelligence. Specifically, we are adapting one widely used procedure in which neuropsychologists assess students' language skills, which reflect grammatical reasoning abilities. We extended it to more complex cases and tested it on adults. We show how to automatically evaluate the model test results. The experiments with several LLM families show that large enough LLM outperform human baseline . We conduct an analysis of the results revealing the difference of the performance pattern between humans and models. We open-sourced the dataset, models and the code.

Introduction
============

The large language models (LLM) have revolutionized the field of naturallanguage processing. Trained on the text mass that can be compared tothe Internet scale, they show high performance results on many tasks.The key for these models are just some start words called prompts thattune these models on a particular task to solve.

Despite that, these models frequently can deviate from the main task andlose the context easily. To deal with it, researchers applied the methodcalled Reinforcement learning from the Human feedback (RLHF) {% cite Ouyang2022TrainingLM %} that fine tune these models in the way that itsanswers are well estimated by humans. The output was a ChatGPT, a neuralchatbot that can maintain human-like conversation and show impressiveabilities and knowledge. Since the ChatGPT many other LLMs emerged.

Some researchers even consider the modern LLM goes far beyond the mereprobabilistic patterns and shows some sparks of artificial generalintelligence, as you can only tell what you want, and it tries to solveit while it was never trained to do exactly this {% cite Bubeck2023SparksOA %}. Another point is the prompt technique called chain-of-thoughts thatforces the LLM to explain how it comes to the particular conclusion stepby step {% cite Wei2022ChainOT %}. Again, this ability was not trained directly,but this multistep reasoning process is similar to how humans actuallydo.

Based on this point, the researchers propose to test the cognitiveabilities of such LLM as for humans. We take a similar approach andsearch for inspiration in neuropsychology. Neuropsychology is acomposite science discipline aimed at studying the brain mechanisms thatare responsible for certain mental processes. Recently, the agentparadigm even more drives idea "LLM can be AI" further.

The term "neuropsychology" first appeared in the work of D. Hebb {% cite Mikadze2011MethodologyON %} {% cite Mikadze2019ARLA %}. Later, the theoreticalfoundations were developed by representatives of narrow locationism andanti-localizationism {% cite Mikadze2011MethodologyON %} {% cite Mikadze2019ARLA %}. Neuropsychology became widespread in the second half of the 20th centurybecause precise instruments became available, and many injured peopleneeded help after global conflicts.

A. R Luria, the representative of scientists in the field ofneuropsychology {% cite Haggbloom2002The1M %}, inspired by earlier works of L.S. Vygotsky and P. K. Anokhin, developed a theory of systemic dynamiclocalization of higher mental functions of a person, so-called "systemapproach" {% cite Luriia2019ContemporaryNA %}. In this theory, complex forms ofmental activity are considered as functional systems that are providedby the work of the entire brain. At the same time, each part of thebrain makes a specific contribution to the construction of thesesystems.

Since we are focusing on LLM language skills in this paper, we takeinspiration from Luria's ideas. In particular, in his book {% cite Danks1982LanguageAC %}, which is dedicated to the relationship betweencognitive processes and language abilities, he described semanticaphasia, the symptom of the logical-grammatical relationshipunderstanding difficulty. Based on this description, contemporarydiagnostic tools were developed {% cite Mikadze2011MethodologyON %} {% cite Mikadze2019ARLA %}. We hypothesize that wecould adapt these tests for modern language models to see if they couldshow a normal human level performance, thus connecting the LLMevaluation skills to the neuropsychology. It becomes especially relevantwhen some research says that language models show some aspects ofcognitive abilities.

In this paper we experiment a test for grammatical abilities of languagemodels based on neuropsychological prototype. We did a survey to get ahuman baseline and compare it with language models of different sizes.We also show how to automatically estimate the result of the responses.

Related work
============

The researchers constantly create various datasets to count theprogress. Often these datasets contain some human baseline and the goalis to beat it with some model. Frequently, the datasets that are used bythe scientific community resemble one of a human cognitive ability. Forexample, SNLI {% cite Bowman2015ALA %} and its extension MNLI {% cite Williams2017ABC %}datasets designed to measure the model ability of text understanding bythe recognizing textual entailment task. Another famous dataset thattests the text comprehension abilities is SQuAD {% cite Rajpurkar2018KnowWY %} dataset that basically is a question-answering dataset. The datasetssuch as STS {% cite Cer2017SemEval2017T1 %} or Quora Question Pair {% cite Wang2017BilateralMM %} are used for testing how models understand thesimilarity between sentences. Of course, there are some common datasetsfor a classification, such as Stanford Sentiment Treebank 2 {% cite Socher2013RecursiveDM %} and Corpus of Linguistic Acceptability {% cite Warstadt2018NeuralNA %}. The research of non-English languages createsresources using English datasets as a prototype. For example, recently,the RuCoLA {% cite Mikhailov2022RuCoLARC %} was proposed for the Russianlanguage.

There are also benchmarks for text generation, such as XSUM{% cite Narayan2018DontGM %} dataset for abstract summarization and Gigaword {% cite Rush2015ANA %} for headline generation. Another interesting dataset isPersonaChat {% cite Zhang2018PersonalizingDA %} where the model is measured byhow accurately it can respond given a personal description.

With emerging multitask learning enabling models to solve several taskssimultaneously, in order to unify the model testing and propose somecommon protocol, researchers created combined benchmarks where theyinclude several datasets that measure several aspects of language usage. These benchmarks also come with the leaderboard and software that allowsresearchers to compare their models in one way. These datasets includeGLUE {% cite Wang2018GLUEAM %} that consists of 9 tasks and its complicatedversion SuperGLUE {% cite Wang2019SuperGLUEAS %} (there is also aRussianSuperGLUE {% cite Shavrina2020RussianSuperGLUEAR %}), GLGE {% cite Liu2020GLGEAN %} dataset for language generation tasks. With the rise ofLLM and its enormous language capabilities, there was a need in thebenchmark that can cover as many tasks as possible. Recently, thecollaborative benchmark was proposed called BIG-bench {% cite Srivastava2022BeyondTI %} that consists of 204 diverse language tasks.There are also area standard benchmarks on which newly released modelsare evaluated like Humanity's Last Exam {% cite phan2025lastexam %} or MERA {% cite fenogenova2024meracomprehensivellmevaluation %}.

Speaking of the datasets that mimic human-like tests, we can highlightthe Cloze-like datasets in which the task formulates as filling the gapwith words or phrases that account the context. This schema is used forassessing the language comprehension skills of native speakers, but alsocan be used as a diagnostic tool for second language learners andchildren. In fact, this test is a part of the follow-describedneuropsychological test. The notable dataset that was built by thisschema is the Story Cloze test {% cite Mostafazadeh2017LSDSem2S %}, where themodel has to choose the right ending given a number of the storysentences. In another work, authors apply mental assessment tests forsome chatbot models {% cite Shan2022MentalHA %}. They use the samequestionnaires that are for people such as PHQ-9, GAD-7, CAGE and TEQand found that models they tested have mental problems. Recently, theresearchers also propose to use the instruments from neuropsychology totest LLM as if it had a functional prefrontal cortex {% cite Loconte2023ChallengingC %}. They use a variety of tests such as verbalreasoning, cognitive estimation, metaphors comprehension, idiomscomprehension, anaphoric referencing, planning, inhibition, insight andmore. The results show that ChatGPT gets normal or low-normal tierestimation. The similar to our work is {% cite alexeyev2021recovering %} whereauthors fine-tuned the LSTM-based model to delemmatize the sentence. Themain difference is that they train the model to a specific task, whilewe estimate LLM skills that emerged during causal language modeling.

Test description
================

The original test was designed for 6-11 old year children to determinethe features of speech development. It consists of six collections oftasks called series that can be verbal or nonverbal. Each series maycontain several tasks. We focus on the one series that test theformation of the grammatical structure of speech that contains fourtasks.

The idea of the this task is to create grammatically correct sentencesfrom the list of ordered words in the normalized form. The participantshould only place each word into the right morphological form in orderto make a grammatically correct and meaningful sentence. Overall it has10 sentences. The shortest sentence has three words, while the longestone five. The answer has one point if the sentence is correct, 0.5points if the order is broken, 0.25 points if there are missed, replacedor inserted words, minor incorrect meaning or grammatical inconsistency(agrammatism), 0 if complete incorrect meaning or refusing to solve thetask.

As we indent to compare the LLM output with human one we planed toconduct a survey. Because the children audience is unavailable for us,we target adult humans. It's obvious that the source test for childrenis not a proper stimuli, moreover, we didn't find similar test foradults. Inspired by the children test, we made our version bycomplicating the tasks using longer sentences and shuffling some ofthem. We are aware that self-made test doesn't have clinical evidencesbut, given the prototype, we hypothesize that our version at least canbe used to compare the grammatical skills without any clinical values.

We design our version with respect to human abilities and intention tocomplete the survey. The too long sentences and, in particular, theshuffled ones could quickly exhaust the participants and make themrefuse to continue the survey. Thus, we tie the complexity on a wellknown working memory capacity rule {% cite Miller1956TheMN %}, that states thatin working memory 7 +/- 2 elements can be placed. So the task complexityis defined as amount of words that a human operates on with respect tothis rule. We consider 4-6 words as an easy task, 7-9 as a medium taskand 10-12 as a hard task.

We combine the tasks by sampling 4 examples from the easy group, 6 fromthe medium and 8 from the hard group, shuffling half of the examples inthe medium and hard groups. The motivation of this sampling is to givepeople some training as they usually don't perform such transformations.Thus, this order helps them not to give up the survey. In total we have18 tasks.

As the source of data we use SynTagRus {% cite Boguslavsky2016SynTagRusA %} dataset that contains the annotation on different language levels. Italso includes word lemmas, so it perfectly fits our needs. When samplingsentences for the tasks we manually validate them and remove allnon-neutral ones that relate to politics, racial, national, sexualorientation offensive and other ethically questionable sentences. Wedon't consider the sentences with outdated or special vocabulary anddirect speech, historical factual examples have also been removed. Whilechoosing sentences for medium and hard groups, we try to includecompound sentences.

The full list of all tasks is in Appendix C.

Estimation methodology
----------------------

As we change the task, we also modify the original estimation schema.The first thing is we exclude the order saving condition, because sometoken sequence can produce several valid sentences. We only need thetoken lengths of the entered sentence and the original sentence to beequal. We don't consider the token-by-token comparison for the caseswhen the token lengths are equal but vocabulary sets are not. It wouldrequire the accurate lemmatizer otherwise its mistakes can seriouslyinfluence the estimation because we would require a full vocabularymatch so one lemmatizer error makes the full mismatch. We leave it forfuture work.

The second thing we should check is whether the produced sentenceviolates language grammatical rules or not. During manual examination ofgenerated examples on grammatical correctness we discover that in somecases sentences despite being grammatically correct are illogical orhave unusual Yoda-like word order. We explicitly marked such sentencesand included those criteria in our estimation. In our methodology webinarize the estimations so each of the criterion can be either 1 or 0.We made this simplification for the sake of a post-survey annotationconsistency. To sum up, we have four criteria:

- grammatical correctness - represents morphosyntactical correctness of the the sentence;
- logical correctness - represents the semantic coherence;
- order normality - represents the correct syntagmatic organization of the sentence;
- sequence equality - represents whether the sentences was combined from what was given.

To be able to automatically assess the separate criteria we useclassifiers trained on the manually annotated data collected fromhumans. We split the entire dataset on train (70%) and test (30%) sampling on human worker level meaning that all answers from one workermust be in the same split. Next we train theRuBERT{% cite zmitrovich2023family %} classifier in each criterion. We also tryLLM-as-a-judge but they are not as good as BERT-based models.

The final per example score is a sum of separate estimations from theset $E$. The performer level score $S$ is computed as the average ofindividual task scores. $$S = \frac{1}{n}\sum_n\sum_{i}E_i$$

One can notice that our estimation task is close to what the RuCoLA dataset  {% cite @mikhailov-etal-2022-rucola %} defines, namely a linguistic acceptability. We test how the ruRoBERTa-large-rucola[^1] trained on RuCoLA dataset is aligned with our dimensions.

Human data annotation and estimators training
=============================================

We collect the data using Yandex.Toloka platform mainly and Google Formswhen we do a pilot try. You can find the instructions in Appendix A. Wemanaged to collect the results from 108 human responders leading to 1944 answers overall. Sometimes the workers refuse to answer particularquestions answering with several symbols or first several tokens (weconsider less than four tokens as a refusal to answer). There are 216such cases in total. We do not exclude them from the dataset. Eachanswer we manually assess for grammatical and logical correctness andorder normality. The assessment is performed by our pre-graduatedlinguist team member who leads the annotation process for two morepre-graduated linguists. While annotating, they should answer astraightforward question: whether the human answer is (1) grammaticallycorrect, (2) logical, and (3) has a normal (natural) word order.

The Krippendorf's alpha values of the fully annotated dataset onseparate criteria are as follows: 0.704 for the grammatical correctness, 0.736 for the logical correctness, 0.774 for the order normality. Thefinal class is determined by majority voting across annotators.

The train split contains 1350 examples shaping to 75 human workers whilethe test split contains 594 examples shaping to 33 human workers. The classifiers were trained for 5 epochs with learning rate 2e-5. For the LLM-as-a-judge we test gpt-oss-120b and claude-sonnet-4. The results forall methods on test split are presented in [Table 1](#tab_1).

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


Clearly, the ruBERT classifiers are best in all cases. We also test whether the BERT based automatic score aligns with the human score with Pearson correlation on the test set. The result value is 0.880 with thep-value is close to zero.

Also we see that RuCoLA model is not far from the ruBERT values giventhat it's actually different model. We calculate metrics using RuCoLA predictions against every tree true label sets. The result can be interpreted as the RuCoLA encodes all three dimensions pretty well, but since separate dimensions perform better by 3-8 points then RuCoLA, wewill use them.

Models to be evaluated
======================

In order to have some sense of model complexity growing, we use severalmodels that range in parameter size. We use several model families:

- Claude-3-haiku, Claude-opus-4, Claude-sonnet-4.5, Claude-opus-4.8
- GPT-3.5-turbo, GPT-4o, GPT-5, GPT-5.4,
- Qwen3.5-9B, Qwen3.5-27B, Qwen3.5-35B-A3B, Qwen3.5-122B-A10B, Qwen3.5-397B-A17B,
- GigaChat-2, GigaChat-2-Max, GigaChat-2-Pro
- Yandex-lite, Yandex-5.1

All models run with the same prompt with temperature equals to 1. Theprompt is presented in Appendix B.

In order to make a more fair comparison, we ran the models five times. As these models have non-deterministic nature ideally we would like tose that the model generates a reasonable response each time as different normal people would also generate a reasonable response. As we require the model to return the answer in predefined format, we accountthat model can make a mistake in answer formatting and in answer itself so we allow the models to try 10 times to return the answer in requiredformat. The final criteria score is aggregated across runs by the simple rule whether the model have 1 more than half times.

Results
=======

<a id="fig_1-overall_score"></a>
**Fig. 1.** *The overall score across the models. The red line is a humanbaseline.*
![The overall score across the models. The red line is a humanbaseline.](/assets/images/measure_machines_as_human/overall_score.png)



The overall evaluation result is shown on [Figure 1](#fig_1-overall_score). The human baseline is a red line computed asaverage on the test set (594 samples) which is equal 0.728. The stickson each bar represent the standard deviation of score across runs. Wesee the clear pattern: the bigger the model (we assume than newer closedmodels are also bigger) the better the result. While most modelsoutperform the human baseline we highlight the Claude Opus 4.5 thatperfectly complete the test. One can also notice that the standarddeviation is remarkably wide for most of the models implying that thesuccess of the task completion is not stable across runs. Overall we canconclude that large enough modern at the time of writing language modelshave a good understanding of Russian grammar and how to use it toconstruct meaningful sentences.

Detailed analysis
-----------------

To get better understanding we can zoom in results in several ways. The [Figure 2](#fig_2-score_deps_on_is_shuffled) show the scores depending onwhether the task was shuffled or not. Note that the human baseline nowis represented as bars also.

<a id="fig_2-score_deps_on_is_shuffled"></a>
**Fig. 2.** *The scores across the models depending on whether the task is shuffled.*
![The scores across the models depending on whether the task isshuffled.](/assets/images/measure_machines_as_human/score_deps_on_is_shuffled.png)


For the humans we see expected results that the score on non-shuffled tasks is much higher than for shuffled one. This pattern generally holdsfor the model results. As we can see all models resolve the non-shuffle dtasks better then humans sometimes perfectly without any variations. The difference comes from shuffled tasks and most likely the average overall score and its deviation heavily depends on how the models deal with shuffled tasks. Not that only three models has average score lower than human baseline.

<a id="fig_3-score_deps_on_complexity"></a>
**Fig. 3.** *The scores across the models depending on the task complexity.*
![The scores across the models depending on the task complexity.](/assets/images/measure_machines_as_human/score_deps_on_complexity.png)

The [Figure 3](#fig_3-score_deps_on_complexity) shows the scores depending on task complexity. As may be expected also, the human performance on the easy task is much higher than other. Notice that the performance on medium and hard complexity go very close to each other. Surprisingly, the opposite pattern is observed for almost all models: the more complex the task (which actually means longer) the better the model performance is .We hypothesize that this relates to the probabilistic nature of the generation mechanism. The short sequence makes the probability calculation to be unstable leading to the erroneous token selection on the sampling stage. This phenomenon may outline the difference in text processing of humans and models. This also says that our scor ecalculation formula is biased toward hard tasks is not account for different length of the task complexity. The plain averaging will favour those who better answer more complex tasks.

We also investigate which of the samples has lowest score across themodels. Using the ID column from the Appendix table, the top 3 worst samples is 7 (7 times), 3 (4 times), and 5 (3 times). In all cases the models often fails to generate grammatically correct of meaningful sentence. The word insertion here is considered to "complete" the sentence. Speaking of the worst sample 7, we believe the main cause ofthis is that another unrecognized transformation: "человек" - "люди" (man - people). Most models try to build the sentence with "человек" (man) and fail.

On a model level the notable issue is that models often break the rule of vocabulary equality: models tend to insert or remove some words .Surprisingly for us, we didn't specify this condition directly in the annotation instruction, nor in the prompt. Probably, specifying this condition in the prompt would improve the result. Also, models fail to recognize the transformations for the pronouns: "он" - "его" (he - him), "мы" --- "нам" (we - we (were told)), "тот" - "той" (that man - that woman), etc. In some samples it's acceptable like in 17 ("в деканате **нам** сказали" - "**мы** сказали в деканате"; "**We we retold** by the Dean's office" - "**We told** to the Dean's office"), but not in other.

Conclusion and future work
==========================

In this work we tried to adapt the test from the neuropsychology for large language models to get a grounded comparison of language skills with the humans. We focus on a one of the skills, namely the ability to put the lemmatized words into a morphologically coherent sentence. This task shows the ability to correctly recognize the relations between thewords. Starting from the basic setup we modify the task to make themmore complex. We conduct the survey in order to get a human baseline and get the sense whether this setting is feasible. Next, we use several models ranging by complexity to test their abilities showing that large enough LLM have good understanding of the Russian grammar in the lens of the used test. The test also shows the rough evidence of the text processing difference between human and LLMs.

The promising results of the score calculation automation allow speakingabout adaptation other tests from neuropsychology and similar fields. Moreover, the rise of multimodality models allow to use non-verbaltests.

The main drawback of this work is that the human participants that tooka survey were in an uncontrolled state that can lower their attentionlevel in thus influence the results. This supports by the fact that 11% of tasks was refused to answer. The fact that we binarize the labels also could lead to a losing the granularity. If one could investigate more resources to the annotation process, he could benefit with more fine grained estimation. For the automation point of view one could archive better results in LLM-as-a-judge if he would invest more time into prompt refinement. Also, self-constructed test doesn't have a clinical validity.

In the future work one can do an in-depth linguistic analysis of tasks and corresponding answers. Based on this analysis one can reveal the reasons why models fail or can create a new set of tasks that will be grounded to a certain linguistic phenomena. On the other hand, the fact the RuCoLA model approximates the considered linguistic dimensionspretty well, one can save time on estimators and concentrate on scale and diversity of the test set. We hope that this work also inspire another researches to investigate the potential of existed methods ofcognitive testing or adapt this method to another morphologically rich languages such as German or Finnish.

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