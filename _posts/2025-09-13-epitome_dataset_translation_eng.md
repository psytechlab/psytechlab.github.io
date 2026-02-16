
--- 
layout: post 
author: Nafisa Valieva 
title: Translation of an empathy assessment dataset to Russian. Approach, challenges, results
use_math: false
--- 

Hi. My name is Nafisa Valieva. I am a junior developer at MWS AI and a 3rd year student of Applied Mathematics and Control Processes at SPbGU. This post is a text version of my presentation at [Data Fest](https://youtu.be/ZkydkhQvO64). I will tell you how we at the Psytechlab team translated an interesting dataset from English to Russian using large language models (LLMs). The approach itself is based on early work [1] by our supervisor. The difference is that here we analyze the behavior of various LLMs in detail.

_Originally, the post was [published on Habr](https://habr.com/ru/articles/946264/), but for the completeness of our work overview, we are placing it in our blog as well._

# What's the point of this anyway and what kind of dataset is this?
Empathy plays an important role in human communication, and particularly in psychological support services. In the online environment, where such assistance is increasingly provided in text format, many different services are emerging that provide psychological support based on chatbots. For them, the ability to respond empathetically becomes a critically important skill. Otherwise, it's good if the session turns out to be simply useless and doesn't exacerbate existing problems.

The success of LLMs encourages developers to use them as a foundation for such chatbots. Various benchmarks are being developed to evaluate their capabilities, particularly for tasks with a focus on psychotherapy. One such benchmark is PsyEval [2].

However, there are simply no annotated datasets for automatic evaluation of empathy in Russian-language texts. We, Russian-speaking ML practitioners, cannot say how LLMs currently cope with tasks related to empathy detection and generation of empathetic responses. Yet these tasks directly affect the quality of psychological support tools.

To somehow remedy this, we adapted large language models to translate the dataset from English to Russian. The target dataset was [EPITOME](https://aclanthology.org/2020.emnlp-main.425.pdf), which consists of texts from Reddit and includes annotation for three types of empathy:
* Emotional reactions - expression of empathy (warmth, compassion, support) in response to the interlocutor's message
* Interpretations - Showing understanding of the interlocutor's feelings and experience.
* Explorations - Active interest in the interlocutor's unexpressed experiences

Each type of empathy has two levels of intensity: weak and strong. In addition to the types themselves, the dataset contains annotated substrings - empathy bearers. They indicate which specific parts of the text reflect empathetic response. Here is an image from the original paper that clearly shows all these types.

![](/assets/images/epitome/epitome_explanation.png)

In brief, the entire work can be broken down into several steps:
1. Select an LLM that will best handle translation.
2. Develop a prompt for translation.
3. Implement a complete dataset translation procedure.
4. Train models for empathy classification in Russian, using the original model.

# LLM selection and prompt development
For testing, we selected several LLMs: GPT-4o, Qwen-2.5 of various scales, Mistral-Small-24B-Instruct-2501, YandexGPT Pro. Additionally, we tested Yandex Translate as an industrial and specialized solution. Along with model testing, we iteratively developed a prompt that incorporated ideas from the analysis of model errors.

At the initial stage, we created a simple translation prompt to obtain a preliminary quality assessment. For test material, we manually selected 20 texts from the dataset containing typical features of Reddit language: slang, non-standard punctuation, emotional expressions, informal constructions, and abbreviations (e.g., "OP", "DAE", "yeet", "tmblr").

Translations were manually cross-checked by two developers. Special attention was paid to preserving meaning and style. For additional verification, we used the L1-diff metric on [LaBSE/en-ru](https://huggingface.co/cointegrated/LaBSE-en-ru) embeddings to measure semantic distance between the original and translation. In general, the metric can be represented by the formula

$$L1= |f(t_{src}) - f(t_{trg})|$$,

where $f(x)$ is the embedder (in this case, the LaBSE model), $t_{eng}$ is the text in the source language (English), $t_{trg}$ is the text in the target language (Russian). As a result, we got the following top-3 models: GPT-4o, Qwen-2.5-72b-instruct, and YandexGPT Pro.

To summarize the error analysis, the main obstacle to good translation is social media style.
Abbreviations and acronyms are natural companions of social networks because users strive to write text faster. Some abbreviations are transferred as is, like VR, tmblr (social network name), and this seems acceptable. For some abbreviations, it's difficult to decide whether to translate them or not, for example, OP — ОП, thread author. There are those that definitely need to be expanded: rn (right now), asap (as soon as possible). Some abbreviations are missed by models (wtf, ish), and the abbreviation DAE (does anyone else) turned out to be so complex that only gpt-4o handled it.
Social media texts are also full of interjections from simple to super elaborate. People often try to imitate conversational speech with them (aaaaand, иииии). They are often either transferred in Latin as is or transliterated.

Users also sometimes decorate texts with peculiar formatting, which confuses some models and causes them to lose context. Of course, wordplay is also sometimes not caught by models: in the sentence "i'm 24 old old man" some models simply skipped the second word "old" instead of translating the sentence as «я 24 летний старик».

The table below shows typical cases (bold text everywhere except the header row should be read as two asterisks before and after the text highlighted in bold)

| **Translation test texts** | **gpt-4o** | **YandexGPT Pro** | **qwen-2.5-7b-instruct** | **qwen-2.5-72b-instruct** | **qwen2.5-32b-instruct** | **qwen2.5-vl-72b-instruct** | **yandex-translate** |
|---|---|---|---|---|---|---|---|
| rn im in the area of not going to sleep so tmrw doesn't come | сейчас я в состоянии не ложиться спать, чтобы завтра не наступило | Я сейчас не сплю, чтобы завтрашний день не наступил | rn я нахожусь в зоне бодрствования, чтобы завтра не приходилось спать | Сейчас я в состоянии, когда не собираюсь спать, чтобы завтрашний день не наступил. | я сейчас в состоянии, когда не хочу спать, чтобы завтра не наступило | Сейчас я в том состоянии, что не могу уснуть, чтобы завтрашний день не наступил. | рн, я как раз собираюсь не ложиться спать, так что tmrw не придет |
| and im sooo tired :( | и я так устал :( | А я так устал :( | и я так устал :( | и я так устала :( | и я так устала :( | и я так уставший :( | и я ооочень устала :( |
| Meh. What's the point.... | Эх. В чем же смысл.... | Эх. В чём смысл... | Мeh. Какая разница.... | Ну и что... какая разница... | Мeh. В чем смысл…. | М-да. Какой в этом смысл.... | Какой в этом смысл?... |
| Yeet yeet. I'm gonna hang myself with the charger ( | Йет йет. Я собираюсь повеситься на зарядке ( | Еееее. Я собираюсь повеситься с зарядкой ( | Мне надо повеситься с зарядным устройством( | Йит йит. Я повешусь наушниками зарядки ( | Yeet yeet. Я собираюсь повеситься шнурком от зарядки ( | Йит йит. Я собираюсь повеситься на зарядном устройстве ( | Да-да-да. Я собираюсь повеситься на зарядном устройстве( |
| POEM:Hollow. ***EMPTY*** *rooms and* ***EMPTY*** ***SOULS*** ***FORGOTTEN*** *places and* ***HEART*** *shaped holes* ***DEPRESSION*** *comes with days of* ***STRUGGLE*** *and often times a* ***BLOODY PUDDLE*** UP\-VOTES CURE MY DEPRESSION | ПОЭМА: Пустота. ***ПУСТЫЕ*** *комнаты и* ***ПУСТЫЕ*** ***ДУШИ*** ***ЗАБЫТЫЕ*** *места и* ***СЕРДЦЕ*** *образные дыры* ***ДЕПРЕССИЯ*** *приходит с днями* ***БОРОТЬБЫ*** *и часто* ***КРОВАВАЯ ЛУЖА*** ВВЕРХ\-ГОЛОСА ИСЦЕЛЯЮТ МОЮ ДЕПРЕССИЮ | СТИХ: Пусто. ***ПУСТО*** комнаты и ***ПУСТО*** ***ДУШИ*** ***ЗАБЫТЫЕ*** места и дыры в форме ***СЕРДЦА*** ***ДЕПРЕССИЯ*** приходит с днями ***БОРЬБЫ*** и часто с ***КРОВАВЫМ ПЯТНОМ*** ВВЕРХ-ГОЛОСУЮТ ИСЦЕЛЯЮТ МОЮ ДЕПРЕССИЮ | БОЛЕТАЯ: Пусто. ***ПУСТО*** *комнаты и* ***ПУСТЫЕ*** ***ДУСЫ*** ***ЗАБЫТЫЕ*** *места и* ***СЕРДЦЕВИКИ*** *депрессия сопоставима с днями* ***МЕРТВЫХ СИЛАВ*** *и часто с* ***КРОВАВЫМ ПОЛЯМЕРОМ*** ЛАУД\-ВИТС ПОЛУЧАЮТ ЗАЩИТУ ОТ МОЕЙ ДЕПРЕССИИ | POEM:Пустота. ***ПУСТОТА*** *помещения и* ***ПУСТОТА*** ***ДУШИ*** ***ЗАБЫТЫЕ*** *места и* ***СЕРДЦЕ*** *формированные дыры* ***ДЕПРЕССИЯ*** *приходит с днями* ***БОРЬБЫ*** *и часто вместе с* ***КРОВАВОЙ ЛУЖЕЙ*** ГОЛОСОВАНИЕ РЕШАЕТ МОЮ ДЕПРЕССИЮ | СТИХ:Пустота. ***ПУСТО*** *комнаты и* ***ПУСТОТА*** ***Забытые*** *мещества и* ***СЕРДЦА*** *образуют пустоты* ***ДЕПРЕССИЯ*** *приходит с днями* ***УСТРЕКОВ*** *и часто с* ***КРОВАВЫМ ОТСЕКОМ*** ЛАЙКИ ЛЕЧАТ МОЮ ДЕПРЕССИЮ | БОЛЕТАЯ: Пусто. ***ПУСТО*** *комнаты и* ***ПУСТЫЕ*** ***ДУСЫ*** ***ЗАБЫТЫЕ*** *места и* ***СЕРДЦЕВИКИ*** *депрессия сопоставима с днями* ***МЕРТВЫХ СИЛАВ*** *и часто с* ***КРОВАВЫМ ПОЛЯМЕРОМ*** ЛАУД\-ВИТС ПОЛУЧАЮТ ЗАЩИТУ ОТ МОЕЙ ДЕПРЕССИИ | СТИХОТВОРЕНИЕ: Пустота. ***ПУСТЫЕ****комнаты и****ПУСТЫЕ****ДУШИ*****ЗАБЫТЫЕ***места и****Дыры в СЕРДЦЕ*****ДЕПРЕССИЯ**** приходит с днями* ***БОРЬБА****и часто ****КРОВАВАЯ ЛУЖА*** ГОЛОСОВ "ЗА" ИЗЛЕЧИВАЕТ МОЮ ДЕПРЕССИЮ. |
| How can I accept my intrusive depressive thoughts?. Long story short: Was 335, Am 175. Aiming for 130/140ish. Body destroyed. | Как мне принять мои навязчивые депрессивные мысли? Короче говоря: было 335, сейчас 175. Цель - примерно 130/140. Тело разрушено. | Как мне принять свои навязчивые депрессивные мысли? Короче говоря: был 335, стал 175. Стремлюсь к 130/140. Тело разрушено. | Как я могу принять навязчивые депрессивные мысли? Краткий обзор: был 335, сейчас 175. Цель - 130/140. Тело полностью уничтожено. | Как я могу принять свои навязчивые депрессивные мысли? Кратко: Было 335, Теперь 175. Цель 130/140. Тело разрушено. | Как мне принять мои навязчивые депрессивные мысли? Коротко говоря: Был 335, Стал 175. Цель - 130/140. Тело разрушено. | Как я могу принять свою навязчивую депрессивные мысли? Короткая история: Был 335, стал 175. Цель 130/140. Тело разрушено. | Как я могу смириться со своими навязчивыми депрессивными мыслями?. Короче говоря, мне было 335, а сейчас 175. Стремлюсь к росту 130/140. Тело разрушено. |
| .... Feeling hopeless and useless at the moment... Empty, lost, darkness | .... Чувствую себя безнадежно и бесполезно в данный момент... Пустота, потерянность, тьма | ... Сейчас я чувствую себя безнадежным и никчёмным... Пустым, потерянным, в темноте | .... Чувствую себя безнадежным и бессилен в данный момент... Пустота, потерялся, тьма. | .... Чувствую себя безнадежным и бесполезным в этот момент... Одиноким, потерянным, во тьме | .... Чувствую себя безнадежным и бестолковым в данный момент... Пустота, потерянность, темнота | .... Чувствую себя сейчас бессмысленным и бесполезным... Пустым, потерянным, в темноте | .... Чувствую себя безнадежным и бесполезным в данный момент... Пустота, потерянность, темнота |

Other features in brief:
* Some models handle punctuation better than others.
* Sometimes models correct obvious logical errors in the text:
  * compare ur self to others because u will loose and forget who u r — **don't** compare yourself to others because you will lose and forget who you really are.

Originally, we expected that for this task we could manage with a small LLM, which in our case was Qwen2.5-7b. The results showed that we couldn't manage and here's why:
* Translated texts contain a lot of Latin script.
* In complex cases where text is written carelessly, the model starts word creation,
  * "How are you?" "I'm F.I.N.E.". **Fucked up**, insecure, neurotic, and emotional. ("How are you?" "I'm F.I.N.E.". Fucked up, insecure, neurotic, and emotional.)
* On long texts may lose meaning.
  * "I'm 32 years old and I've never had a girlfriend. Very sad that I've been trying to find someone for a long time, even using online dating, but I feel like I'll remain single forever."
* The model often uses not quite the right words (not "want to die" but "want to perish"), confuses parts of speech and word declensions.

Besides model selection, we also improved the prompt based on typical translation errors. Additionally, we included known general prompt engineering practices. Notable parts include:
* Requirement to preserve the original style of messages, including emotional and suicidal expressions.
* Instructions for bypassing model filters that prevent translation of texts with suicidal and depressive themes,
* Requirement to accurately transfer semantic accents,
* Requirement to necessarily include translated empathy carriers as substrings in the main text.
We also tested the model in batch translation mode — simultaneous translation of multiple texts. This approach presents a choice between speed and quality, because the more texts need to be translated, the more likely the LLM will do something wrong. We specifically tested how LLMs would work in this mode, but we translated the final dataset one text at a time. The table below shows the results of our top-3 models when translating in batches of 32 texts. It's visible that while in regular translation all models perform well, batch translation of explanations doesn't go as smoothly.

![](/assets/images/epitome/firefox_XC1tyndrMU.png)

# The entire pipeline
The dataset translation takes place in two stages: general translation and empathy bearer translation. For both stages, YandexGPT Pro was used as the main translation model, as it demonstrated the best quality-to-price ratio for most examples. For problematic cases where YandexGPT made distortions or foreign language character insertions, Qwen-2.5-72b-instruct was applied. If the problem persisted, GPT-4o was used. This stepped process allowed achieving the best balance between cost and quality.

In the second stage, a special script checked that each translated empathy bearer substring was included in the main translation without changes. If at least one fragment could not be accurately matched, the text was retranslated using another more successful model.

![](/assets/images/epitome/epitome_translation_pipeline.png)

Budget
The total budget for dataset translation was up to 5,000 rubles, including test translations, the main part of the dataset, and empathy bearer processing. Many thanks to Yandex Cloud for the 3,000 ruble certificate upon registration. The detailed breakdown looks like this:
* Translation testing - 500 rubles (200 - YandexGPT Pro, 300 - Bothub (GPT-4o, qwen*))
* 1,800 seeker posts - 1,500 rubles Bothub (GPT-4o)
* Translation of 2,943 bearers, 1,284 seeker posts and 3,084 response posts - 3,000 rubles (YandexGPT Pro - 2,300 rubles, Bothub (GPT-4o, qwen-2.5-72b-instruct) - 700 rubles)

# Model Training
To understand whether our translated dataset is actually worth anything, we trained the original classification and empathy bearer extraction model. As base encoders, we tested rubert-base-cased and xlm-roberta-base. Quality was also measured using a set of original metrics:
* Accuracy — proportion of correct predictions,
* F1-score — harmonic mean of precision and recall,
* Token-level F1 (T-F1) — F1 at the token level for extraction tasks,
* IOU (Intersection over Union) — measure of overlap between predicted and reference fragments

The experimental results are shown in the table below. The "authors' metrics" row shows metrics from the original paper [3] for indirect comparison of the model's performance. It can be seen that the order of values and quality distribution across subtasks generally match, which indicates the adequacy of the translated dataset, and therefore the effectiveness of the described translation method. It is interesting to note that rubert-base-cased consistently outperforms xlm-roberta-base on most metrics.

![](/assets/images/epitome/firefox_CoOIRceq7V.png)

# Conclusions and what's next
Finally, let's list all the problems we encountered and which require further development:
* Many models by default refuse to work with potentially sensitive content (suicidal or depressive texts). Sometimes this can be bypassed with prompt engineering, sometimes not.
* Communication style in social networks includes many features. Not every model can understand and preserve it during translation.
* Cultural ambiguity and subjectivity of annotations: manifestations of empathy in English-speaking and Russian-speaking contexts may differ, and the annotations themselves for empathy levels and subtypes depend on annotators' perception, which affects model interpretability and training.
Besides solving the described problems, additional directions can also be proposed:
* Dataset expansion through additional sources, including real dialogues from psychotherapeutic platforms.
* Fine-tuning and additional training of LLMs on tasks of generating empathetic responses in dialogue conditions.
* Building an open benchmark for evaluating LLMs' ability to recognize and generate empathetic responses in Russian.
The translated dataset can be found [here](https://huggingface.co/datasets/psytechlab/epitome-reddit-ru), the pipeline can be found [here](https://github.com/psytechlab/empathy_dataset_transfer). Our team's channel is [here](https://t.me/psytechlab).

See you soon.

[1] D. Popov, E. Terentev, D. Serenko, I. Sochenkov, and I. Buyanov, "Transferring natural language datasets between languages using large language models for modern decision support and Sci-Tech analytical systems," Big Data and Cognitive Computing, vol. 9, no. 5, p. 116, Apr. 2025, doi: 10.3390/bdcc9050116.

[2] H. Jin, S. Chen, M. Wu, and K. Q. Zhu, "PsyEVAL: a comprehensive large language model evaluation benchmark for mental health," arXiv (Cornell University), Jan. 2023, doi: 10.48550/arxiv.2311.09189.

[3] A. Sharma, A. S. Miner, D. C. Atkins, and T. Althoff, "A computational approach to understanding empathy expressed in Text-Based Mental Health support," arXiv (Cornell University), Jan. 2020, doi: 10.48550/arxiv.2009.08441.

