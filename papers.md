---
layout: page
title: Статьи
permalink: /papers
---

# Перевод на русский язык датасета для оценки эмпатии

**Abstract:** Эмпатия играет важную роль в коммуникации между людьми, и в частности, в психологической помощи. Чатботы для оказания такой помощи получили широкое распространение. Для них способность эмпатично отвечать становится критически важным навыком, иначе сеанс может показаться безэмоциональным, что может навредить пользователю, находящемуся в уязвимом психологическом состоянии. Поскольку для русского языка нет датасетов для оценки эмпатии, в рамках данной работы мы предлагаем метод перевода англоязычного датасета EPITOME на русский язык с помощью больших языковых моделей (LLM). Этот датасет включает тексты пользователей Reddit, размеченные по трем типам эмпатии и уровнями выраженности: слабый и сильный. Кроме самих типов датасет содержит аннотированные подстроки — носители эмпатии. Они указывают, какие именно части текста отражают эмпатичный отклик.

([paper](./assets/pdfs/emiptome_translation_into_rus.pdf), [code](https://github.com/psytechlab/empathy_dataset_transfer), [dataset](https://huggingface.co/datasets/psytechlab/epitome-reddit-ru))


```bibtex
@inproceedings{epitome25,
  title        = {Translation of the Dataset for Empathy Assestment into Russian},
  author       = {Nafisa Valieva and Igor Buyanov and Anastasiya Goncharova},
  year         = 2025,
  booktitle    = {Control Processes and Stability},
  volume       = 12,
  pages        = {229-230},
  url          = {https://www.elibrary.ru/item.asp?id=82857874}
}
```

# The methodology of constructing the large-scale dataset for detecting presuicidal and anti-suicidal signals in social media texts in Russian

**Abstract:** The suicide is a terrifying act of a person who is misled by his own mental state. This problem arises across many countries. Sadly, Russia also has quite high number of persons who committed suicide. Luckily, a subset of these people writes their struggles in social media, allowing a way to find them and help. However, these valuable texts disappearing in many irrelevant texts which is considerably slowing down the decision process about person's suicidal risk. To tackle this problem, in this work we have presented a detailed methodology of building the dataset for detecting texts that describe presuicidal and anti-suicidal signals. This methodology describes the process of instruction and class table creation, the process of annotation, verification and post-annotation correction. Guiding by this methodology, we collect and annotate a large-scale Russian dataset with more than 50 thousand texts from social media. We provide a count statistic of the dataset as well as common problems in annotation. We also conduct basic experiments of building the classification models to show the on go performance on different levels of annotation. Furthermore, we make the dataset, code and all materials publicly available. 

([paper](./assets/pdfs/dataset_methodology_presui_antisui.pdf), [code](https://github.com/psytechlab/kitoboy-ml))

```bibtex
@inproceedings{dataset_meth_25,
  title        = {The methodology of constructing the large-scale dataset for detecting presuicidal and anti-suicidal signals in social media texts in Russian},
  author       = {Buyanov I.O., Yaskova D.V., Serenko D.S., Shkereda D.N., Yaskov A.D., Sochenkov I.V.},
  year         = 2025,
  booktitle    = {Proceedings of the Institute for System Programming},
  volume       = 37,
  number       = {issue 6, part 2},
  pages        = {191-210},
  url          = {https://www.ispras.ru/en/proceedings/isp_37_2025_6-2/isp_37_2025_6-2_191/}
}
```
