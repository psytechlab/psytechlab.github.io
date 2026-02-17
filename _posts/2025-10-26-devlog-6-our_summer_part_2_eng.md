
--- 
layout: post 
title: Devlog 6. How We Spent Our Summer, Part 2
use_math: false
--- 

We spent a lot of time on the translation pipeline, while the question of how to improve translation quality remained unresolved. We saw various examples — from questionable to completely worthless — when we evaluated LLMs separately. We decided to test a simple hypothesis: if we calculate perplexity for translated text using a small trained language model, then texts with higher perplexity would represent poor translations.

As our base language model, we used [ai-forever/rugpt3small_based_on_gpt2](https://huggingface.co/ai-forever/rugpt3small_based_on_gpt2) because it's easy to work with in terms of running on available hardware. We took this model, our translations, ran everything through, and got perplexity scores. We quickly realized that it's better to work with logarithmized perplexity because for short texts its values sometimes reach Venus. As a result, we got this general distribution and, further, a distribution depending on text length.

![](/assets/images/devlog_6-perplexity_all.png)
![](/assets/images/devlog_6-perplexity_dep_len.png)

In the general distribution, we can see that the bell curve is shifted left, and on the right we have a long tail. This picture gave us hope for the plausibility of our hypothesis. In the second distribution, it's clearly visible that the perplexity range changes with text length. Since perplexity is inadequate for very short texts, we decided to consider only texts longer than five tokens. We decided to use the fourth quartile as the boundary: everything above it we consider a "bad" translation due to high perplexity. To account for the variability of the range, we distributed length values into "bins" by deciles and calculated perplexity quartiles within each bin.

Here are the translations the algorithm considered "bad":
| **text_eng**                                                                                                                                                                                                                                                                                                                                       | **text_rus**                                                                                                                                                                                                                                                                                                        |
|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| But even when there were a lot of cases, there are some people on my floor that walked freely in and out many times in one day when  it wasn't safe for anyone. And I didn't get that at all and that was when I was angry and anxious of their actions.                                                                                       | Но даже тогда, когда было много случаев, некоторые люди на моем этаже ходили свободно в и из дома много раз в день, когда это не было безопасно для никого. И я совсем не понимал этого, и тогда я был зол и тревожен из-за их действий.                                                                            |
| am based on a quota.  I am suppose to be able to move 10 pallets per hour.  that's almost impossible as it is a 1 million square foot warehouse.  If i get a pallet on the west side and have to move it to the east side that takes 7-10 minutes even if your going dangerously fast.                                                         | работаю по квоте. Мне нужно перемещать 10 паллет в час. Это почти невозможно в складе площадью в 1 миллион квадратных футов. Если паллет находится на западной стороне, а его нужно перевести на восточную, это занимает 7-10 минут, даже если едешь с опасной скоростью.                                           |
| I think I pretty much got everything I need from you- I just needed to vent I think. So we can stop talking now or whenever you have to leave                                                                                                                                                                                                  | Я думаю, я получил все, что мне нужно от тебя - мне просто нужно было выговориться, думаю. Так что мы можем закончить разговор сейчас или когда тебе нужно уйти                                                                                                                                                     |
| She had a desire to do marine biology and I wanted to pursue law enforcement as a police officer, however due to my back injury that fell through recently. She would spend most of her time doing field work, which would require her to spend time out at sea. She was working in a nursing home at the time and was not a marine biologist. | Она хотела заниматься морской биологией, а я собирался посвятить себя службе в полиции, однако моя травма спины недавно все эти планы сорвала. Большую часть времени ей пришлось бы проводить в полевых условиях, что требует работы в море. В то время она работала в доме престарелых и не была морской биологом. |
| I can understand how you would feel like he doesn't care about the strain it's placed. Can I ask, are your daughters aware or involved in these rumors? This could make a difference in how you could respond.                                                                                                                                 | Я понимаю, как ты можешь чувствовать, что он не заботится о твоем состоянии. Можно спросить, знают ли или участвуют ли твои дочери в этих слухах? Это может повлиять на то, как ты можешь отреагировать.                                                                                                            |

And how do we remove the quotes around the word "bad"? We need to conduct a statistical test with people! Here's how we organized it:

1. We randomly selected 25 translations that we identified as bad.
2. We randomly selected 25 translations from the entire dataset.
3. We made sure the samples don't overlap.
4. We mixed all examples into one pile in such a way that they could later be separated by original groups.
5. We annotated translations for quality on a five-point scale.
6. We divided the scores according to the two original groups.
7. We passed the resulting groups to the Mann-Whitney U-test function `scipy.mannwhitneyu(perplexity_selected, random_selected)`.

Here's what we understood by "translation quality":
```
Translation quality should be evaluated by their naturalness from the perspective of the Russian language. To make it clearer, here's a list of some criteria we understand by this:
- Correctness of grammatical constructions (declensions, case agreement, etc.)
- The translation should accurately convey the meaning of the original text
- Even if the text's meaning is conveyed correctly, the text should not "hurt the eye"
- The translation style corresponds to the original style
- Phrases within the translated text should be logically and grammatically connected to each other
```

We looked for annotators on Profi.ru. The criterion was either translator education or at least 2 years of experience working as one. We found three people in total. The final score for an example was calculated as the average of three scores. Aaaaand... p-value=0.08. This can be read roughly as: "it might even work, but it's better to look for something more reliable."

Besides finding bad translations, we also need to remake them. We all know that if your LLM doesn't handle the task, you just need to take a bigger one. Since we already had hired people, we conducted another statistical experiment: will translation quality be better if we use a larger model? To check this, we did the following:

1. We selected 200 "bad" texts.
2. We translated them using a powerful model (in our case, gpt-4o).
3. We asked a person to compare which translation is better (or the same). Quality criteria see above.
4. We stuffed the results into a binomial test `binomtest(new_win, n=n, p=0.5, alternative='greater')`.

As a result, we got p-value=0.001. The rule "just take a bigger model" works.

The last thing we touched on tangentially was LLM-as-a-judge. Since we're an independent team, careful resource management is our absolute foundation. Checking every invented selector algorithm with people doesn't align well with this. It would be great to filter out completely bad variants using LLM.

We deliberately didn't study anything in this area, just decided to conduct another simple statistical test on the data we have: are there any associations between different LLM ratings and human ratings for translation quality? We tested these fellows: 
* gpt-4o
* claude-sonnet-4
* gemini-2.5-flash
* qwen3-235b-a22b
* deepseek-chat-v3-0324
* llama-4-maverick

We tested in two variants: 

* standard — asked the LLM to give a rating from 1 to 5.
* simplified — asked the LLM to simply say whether the translation is bad or not, and we collapsed human ratings according to the scheme {1,2,3} — "bad", {4,5} — "good".

We checked for associations using chi-square. As a result, we got this table:

|                            | Annotator 1 | Annotator 2 | Annotator 3 | Average  |
|:--------------------------:|:-----------:|:-----------:|:-----------:|:--------:|
| gpt-4o__15                 | 1.000000    | 0.175658    | 0.001253    | 0.999996 |
| claude-sonnet-4__15        | 1.000000    | 0.387769    | 0.005886    | 1.000000 |
| gemini-2.5-flash__15       | 0.999996    | 0.118529    | 0.000040    | 0.999956 |
| qwen3-235b-a22b__15        | 0.999993    | 0.160640    | 0.000288    | 0.999976 |
| deepseek-chat-v3-0324__15  | 0.999997    | 0.101087    | 0.000426    | 0.999987 |
| llama-4-maverick__15       | 1.000000    | 0.214498    | 0.000771    | 0.999997 |
| gpt-4o__12                 | 1.000000    | 1.000000    | 1.000000    | 1.000000 |
| claude-sonnet-4__12        | 1.000000    | 1.000000    | 1.000000    | 1.000000 |
| gemini-2.5-flash__12       | 1.000000    | 1.000000    | 1.000000    | 1.000000 |
| qwen3-235b-a22b__12        | 1.000000    | 1.000000    | 1.000000    | 1.000000 |
| deepseek-chat-v3-0324__12  | 1.000000    | 1.000000    | 1.000000    | 1.000000 |
| llama-4-maverick__12       | 1.000000    | 1.000000    | 1.000000    | 1.000000 |

And here are the oddities. No LLM correlates at all with the first annotator, with the second, of course, not zero, but not statistically significant, but with the third it already correlates. The average score and simplified variant also show absolutely "nothing". Here we decided to calculate annotation quality agreement and got no less than -0.09 according to Krippendorff.

![](/assets/images/devlog_6-meme.png)

It turns out that the selector evaluation is also unreliable since we have three people who, despite the criteria, judge translation quality each according to their own vibes. It's interesting that LLMs' vibes match one of the people. It's also quite likely that the reliability of the quality improvement test also turned out so-so for the same reason. In general, we need to look for a more reliable way to establish translation quality. We'll think about it.

