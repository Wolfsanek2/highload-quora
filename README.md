# Проектирование высоконагруженного сервиса Quora
## 1. Тема и целевая аудитория
Quora - это социальный сервис вопросов и ответов, предназначен для обмена знаниями.

### MVP
1. Задание вопросов
2. Ответы на вопросы
3. Посты
4. Оценки ответов и постов
5. Комментарии
6. Сообщества

### Ключевые продуктовые решения
Анонимные вопросы, тематические сообщества (Spaces) и алгоритмы рекомендации

### Целевая аудитория

![Распределение пользователей по странам ](./img/Quora-Users-By-Country.jpg)[[1]]

![Распределение пользователей по странам](./img/Quora-Traffic-By-Country.jpg)[[1]]

Распределение пользователей по устройствам [[1]]:
|Устройства|% пользователей|
|-|:-:|
|Mobile Web|76.62%|
|Desktop|23.38%|

## 2. Расчет нагрузки
### Продуктовые метрики
* MAU: 400M [[2]]
* DAU: 27M [[4]]
* Вопросов в день: 5000-7000 [[4]]
* Среднее количество ответов на вопрос: 5 [[4]]
* Количество тем: 300000 [[2]]
* среднее время сессии:  2 минуты 39 секунд [[3]]

### Технические метрики
* Размер хранения в разбивке по типам данных
* Сетевой трафик
* RPS в разбивке по типам запросов

### Расчет RPS
Расчитывать будем 2 основные операции: создание и чтение. Влияние запросов изменения и удаления на общую нагрузку системы является незначительной.
1. Вопросы
    1. Создание \
В день задается 7000 вопросов. Значит RPS на создание вопроса равен: \
RPS = 7000 / 24 / 3600 = 0.081/с
    1. Чтение \
Данных о количестве просматриваемых вопросов нет. Поэтому примем, что средний пользователь просматривает 10 вопросов в день. Получим RPS на чтение вопросов: \
RPS = 27*10^6 * 10 / (24 * 3600) = 3125/с

1. Ответы \
Среднее количество ответов на вопрос - 5. Для расчета RPS ответов умножим это значение на RPS вопросов.
    1. Создание \
RPS = 5 * 0.081 = 0.405/с
    1. Чтение \
RPS = 5 * 3125 = 15630/с

1. Посты \
Количество постов примерно равно количеству вопросов, поэтому RPS постов примем равным RPS вопросов.
    1. Создание \
RPS = 0.081/с
    1. Чтение \
RPS = 3125/с

1. Оценки \
Среднее количество оценок на пост в ленте - 1823, на ответ - 83.7
    1. Создание \
RPS = 1823 * 0.081/с + 83.7 * 0.405/с = 181.6/с
    1. Чтение \
RPS на чтение оценок примем равным сумме RPS на чтение постов и ответов
RPS = 3125/с + 15630/с = 18750/с

1. Комментарии
В среднем на один пост в ленте пишется 142.1 комментариев, на ответ - 17.4
    1. Создание \
RPS = 142.1 * 0.081/с + 17.4 * 0.405/с = 18.6/с
    1. Чтение \
Будем считать, что пользователь в среднем просматривает 10 комметариев к посту или ответу: \
Комментариев/с = 10 * 3125/с + 10 * 15630/с = 187500/с \
При этом будем считать, что в запросе отправляется пачка из 10 комментариев: \
RPS = 18750/с

### Расчет трафика
1. Вопросы \
Среднее количество символов в вопросе - 120. В кодировке UTF-8 каждый символ занимает максимум 4 байта. Тогда размер одного вопроса равен 120 * 4 = 480 байт. 
Трафик создания вопросов = 480 байт * 0.081/с = 38.9 байт/с \
Трафик чтения вопросов = 480 байт * 3125/с = 1.4 Мб/с \
Суммарный трафик вопросов = 1.4 Мб/с

1. Ответы \
Среднее количество символов в ответе - 918, изображений - 0.6. Примем размер одного изображения равным 1 Мб. Тогда размер одного ответа равен 918 * 4 байт + 0.6 * 1 Мб = 618 Кб. \
Трафик на создание ответа = 618 Кб * 0.405/с = 250 Кб/с
Трафик на чтение ответов = 618 Кб * 15630/с = 9.2 Гб/с
Суммарный трафик ответов = 250 Кб/с + 9.2 Гб/с = 9.2 Гб/с

1. Посты \
Среднее количество символов в одном посте - 968, изображений - 1.8. Тогда размер одного поста равен 968 * 4 байт + 1.8 * 1 Мб = 1.8 Мб. \
Трафик на создание поста = 1.8 Мб * 0.081/с = 150 Кб/с \
Трафик на чтение постов = 1.8 Мб * 3125/с = 5.5 Гб/с \
Суммарный трафик постов = 150 Кб/с + 5.5 Гб/с = 5.5 Гб/с

1. Оценки \
При оценивании в теле запроса передается приблизительно 512 байт данных. \
Трафик на оценивание = 512 байт * 181.6/с = 90.8 Кб/с \
Данные об оценках приходят вместе с запросом поста или ответа. Поэтому размер одного лайка можно принять равным 20 байт. \
Трафик чтения оценок = 20 байт * 187500/с = 366 Кб/с \
Суммарный трафик оценок = 457 Кб/с

1. Комментарии \
Среднее количество символов в одном комментарии - 87. Тогда размер одного комментария = 87 * 4 байт = 348 байт. \
Трафик на создание комментариев = 348 байт * 18.6/с = 6.3 Кб/с \
Трафик на чтение комментариев = 348 байт * 187500/с = 62.2 Мб/с \
Суммарный трафик комментариев = 62.2 Мб/с

### Расчет хранилища

Расчитаем объем, на который нужно расширять хранилище каждый год. Для этого трафик на создание хранимой сущности будем умножать на число секунд в году:

1. Вопросы \
Рост хранилища = 38.9 байт/с * 365 * 86400 = 1.1 Гб/год

1. Ответы \
Рост хранилища = 250 Кб/с * 365 * 86400 = 7.4 Тб/год

1. Посты \
Рост хранилища = 150 Кб/с * 365 * 86400 = 4.4 Тб/год

1. Комментарии \
Рост хранилища = 6.3 Кб/с * 365 * 86400 = 189.7 Гб/год

### Результаты расчета

* Размер хранения

|Тип данных|Размер, Тб/год|
|-|:-:|
|Вопросы|$1.1 \cdot 10^{-3}$|
|Ответы|7.4|
|Посты|4.4|
|Комментарии|$189.7 \cdot 10^{-3}$|

* Сетевой трафик

Пиковый трафик примем в 2 раза больше среднего

|Тип данных|Средний|Пиковый|Суммарный суточный|
|-|:-:|:-:|:-:|
|Вопросы|11.4 Мбит/с|22.8 Мбит/с|120.7 Гб|
|Ответы|73.7 Гбит/с|147.4 Гбит/с|777 Тб|
|Посты|44 Гбит/с|88 Гбит/с|464.4 Тб|
|Оценки|3.57 Мбит/с|7.14 Мбит/с|37.7 Гб|
|Комментарии|498 Мбит/с|996 Мбит/с|5.1 Тб|

* RPS

|Тип данных|RPS|
|-|:-:|
|Вопросы|3125|
|Ответы|15630|
|Посты|3125|
|Оценки|18750|
|Комментарии|18750|

## 3. Глобальная балансировка нагрузки

Распределение MAU по регионам [[2]]:
![Распределение пользователей по странам](./img/400-million-monthly-users-on-quora-e1691174358978-980x521.webp)

Получим следующее распределение пользователей по частям света:
* Северная Америка: 41.6 %
* Азия: 40.4 %
* Европа: 23.2 %
* Австралия: 2.7 %
* Южная Америка: 0.7 %

Целесообразно располагать датацентры в Северной Америке, Азии и Европе. Нагрузка на датацентры распределена неравномерно, поэтому необходимо балансировать нагрузку с учетом географического расположения клиентов и датацентров. Для этого будем использовать latency-based DNS.

## 4. Локальная балансировка нагрузки

Для локальной балансировки нагрузки будем использовать L7 балансировку с помощью nginx. Это повысит надежность сервиса и она будет зависеть только от надежности железа, поскольку нагрузка будет балансироваться с упавших бэкендов на рабочие, а nginx способен долго работать без перезапуска. Также это обеспечит SSL termination внутри датацентра. Помимо этого можно обеспечить раздачу статики и кэширование ответов динамики на популярные запросы. Также будем использовать сжатие контента (gzip) для уменьшения трафика.

nginx позволит экономить количество блокирующих соединений к application серверам. В качестве алгоритма балансировки будем использовать least connections.

## Список источников
1. [https://www.grabon.in/indulge/tech/quora-statistics/][1]
1. [https://business.quora.com/resources/reach-over-400-million-monthly-unique-visitors-on-quora/][2]
1. [https://startupbonsai.com/quora-statistics/][3]
1. [https://www.demandsage.com/quora-statistics/][4]

[1]: https://www.grabon.in/indulge/tech/quora-statistics/
[2]: https://business.quora.com/resources/reach-over-400-million-monthly-unique-visitors-on-quora/
[3]: https://startupbonsai.com/quora-statistics/
[4]: https://www.demandsage.com/quora-statistics/

