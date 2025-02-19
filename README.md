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

### Анализ трафика
* MAU: 400M
* 5000-7000 вопросов в день
* 300000 тем на 24 языках
* среднее время сессии 8-10 минут

### Целевая аудитория

![Распределение пользователей по странам ](./img/Quora-Users-By-Country.jpg)[[1]]

![Распределение пользователей по странам](./img/Quora-Traffic-By-Country.jpg)[[1]]

Распределение пользователей по устройствам [[1]]:
|Устройства|% пользователей|
|-|:-:|
|Mobile Web|76.62%|
|Desktop|23.38%|

[1]: https://www.grabon.in/indulge/tech/quora-statistics/

## Список источников
1. [Статистика по пользователям][1]
