---
title: Нейро сети для самых маленьких. Часть первая (которая после нулевой). Удобство в прокрустовом ложе оптимизации
url: https://habr.com/ru/companies/yandex/articles/1047072/?utm_source=habrahabr&utm_medium=rss&utm_campaign=corporate_blog
published: "2026-07-01T07:00:06Z"
feed: yandex
guid: https://habr.com/ru/companies/yandex/articles/1047072/
---

# Нейро сети для самых маленьких. Часть первая (которая после нулевой). Удобство в прокрустовом ложе оптимизации

![](https://habrastorage.org/getpro/habr/upload_files/9f8/aff/163/9f8aff163f19bad0ca67133b705b090b.png)

Это первая (после нулевой) статья из серии [Нейро сети для самых маленьких](https://linkmeup.ru/ndsm/), в которой мы разбираем инфраструктуру для запуска нейронных сетей.

Для обучения и инференса нейросетей и для любых видов High Performance Computing используются специализированные технологии: GPU/TPU, RDMA, Kernel bypass, NVLink, InfiniBand, RoCE и другие. Про некоторые из них большинство только что-то слышали, но сталкиваться с ними не приходилось.

Нельзя просто взять ванильный стек Linux, воткнуть в него 400Gb Ethernet+IP и получить рабочее решение. Почему?

Потому что общее решение на масштабе в большинстве случаев проигрывает специализированным как в скорости, так и в стоимости. Как бы странно последнее ни звучало.

[Читать далее](https://habr.com/ru/articles/1047072/?utm_source=habrahabr&utm_medium=rss&utm_campaign=corporate_blog#habracut)
