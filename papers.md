---
layout: page
title: Статьи
permalink: /papers
---

# Перевод на русский язык датасета для оценки эмпатии

**Abstract:** Эмпатия играет важную роль в коммуникации между людьми, и в частности, в психологической помощи. Чатботы для оказания такой помощи получили широкое распространение. Для них способность эмпатично отвечать становится критически важным навыком, иначе сеанс может показаться безэмоциональным, что может навредить пользователю, находящемуся в уязвимом психологическом состоянии. Поскольку для русского языка нет датасетов для оценки эмпатии, в рамках данной работы мы предлагаем метод перевода англоязычного датасета EPITOME на русский язык с помощью больших языковых моделей (LLM). Этот датасет включает тексты пользователей Reddit, размеченные по трем типам эмпатии и уровнями выраженности: слабый и сильный. Кроме самих типов датасет содержит аннотированные подстроки — носители эмпатии. Они указывают, какие именно части текста отражают эмпатичный отклик.

([paper](./assets/pdfs/emiptome_translation_into_rus.pdf), [code](https://github.com/psytechlab/empathy_dataset_transfer), [dataset](https://huggingface.co/datasets/psytechlab/epitome-reddit-ru))


```bibtex
@inproceedings{mypaper,
  title        = {Translation of the Dataset for Empathy Assestment into Russian},
  author       = {Nafisa Valieva and Igor Buyanov and Anastasiya Goncharova},
  year         = 2025,
  booktitle    = {Control Processes and Stability},
  volume       = 12,
  pages        = {229-230},
  url          = {https://www.elibrary.ru/item.asp?id=82857874}
}
```
