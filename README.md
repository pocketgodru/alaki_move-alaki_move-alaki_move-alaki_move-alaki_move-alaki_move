
# Подскажем

# Содержание

1. [Сбор базы фильмов](#сбор-базы-фильмов)
   - [Анализ открытых источников и ресурсов с фильмами](#анализ-открытых-источников-и-ресурсов-с-фильмами)
   - [Сравнение с аналогами](#сравнение-с-аналогами)
   - [Сбор данных](#сбор-данных)
2. [Реализация](#реализация)
   - [Принцип работы системы](#принцип-работы-системы) (+ДИАГРАММА)
   - [Структура проекта](#структура-проекта)
   - [Файловая структура](#файловая-структура)  (БУДУТ ПРАВКИ)
   - [Алгоритм поиска](#алгоритм-поиска)
3. [Используемый стек технологий](#используемый-стек-технологий)
   - [Языки программирования и фреймворки](#языки-программирования-и-фреймворки)
   - [Базы данных](#базы-данных)
   - [Машинное обучение](#машинное-обучение)
   - [Дополнительные библиотеки](#дополнительные-библиотеки)
4. [Детали реализации моделей машинного обучения](#детали-реализации-моделей-машинного-обучения)
   - [Векторизация текста](#векторизация-текста)
   - [Индексация и поиск](#индексация-и-поиск)
   - [Ранжирование результатов](#ранжирование-результатов)
   - [Обработка LLM](#обработка-llm)
5. [Создание сайта и пользовательский путь](#создание-сайта-и-пользовательский-путь)  
   - [Опрос](#опрос)

# Сбор базы фильмов

## Анализ открытых источников и ресурсов с фильмами

В качестве основного источника данных был выбран **Кинопоиск**. Этот ресурс отличается высоким уровнем детализации информации — здесь представлены не только рейтинги с различных площадок, постеры в разных форматах и подробные описания фильмов, но и данные о сериалах и аниме.

## Сравнение с аналогами

- **IMDb:**  
  IMDb является мировым стандартом для поиска фильмов и сериалов, однако его база данных ориентирована на международную аудиторию. Кинопоиск, напротив, имеет более глубокое покрытие отечественного кинопроизводства, а также предоставляет дополнительные локальные рейтинги и отзывы, что делает его незаменимым для пользователей, интересующихся российским кино.

- **The Movie Database (TMDb):**  
  TMDb обладает открытым API и широкими возможностями для разработчиков, но его информация часто носит более универсальный характер и может быть менее адаптирована для русскоязычной аудитории. Кинопоиск предлагает данные, специально подобранные для российского рынка, с акцентом на локальные реалии и предпочтения зрителей.

- **Rotten Tomatoes:**  
  Rotten Tomatoes сосредоточен в первую очередь на критических отзывах и мнениях пользователей из США, что может не всегда отражать вкусы и особенности отечественного кинематографа. Кинопоиск, напротив, включает рейтинги как международных, так и российских критиков, что позволяет получить более комплексную оценку фильма с учётом местного контекста.

| **Критерий**              | **Кинопоиск**                                                | **IMDb**                                               | **TMDb**                                          | **Rotten Tomatoes**                          |
|---------------------------|--------------------------------------------------------------|--------------------------------------------------------|---------------------------------------------------|----------------------------------------------|
| **Ориентация аудитории**  | Русскоязычная     | Международная, глобальная база данных                  | Международная, универсальные данные               | Американская, с упором на критические обзоры   |
| **Детализация информации**| Высокая: подробные описания, рейтинги, постеры, новости        | Обширная информация, но меньше локальной специфики      | Хорошая: большое количество медиа-материалов       | Фокус на рейтингах и критических отзывах      |
| **Локальные данные**      | Да, включает российский кинематограф и локальные рейтинги       | Нет, преимущественно глобальные данные                  | Нет, универсальные данные                         | Нет, ориентировано на американский рынок       |
| **API доступность**       | Да, через [kinopoisk.dev](https://kinopoisk.dev/)              | Да, но с ограничениями                                 | Да, но с ограничениями                                   | Да, но с существенными ограничениями           |
| **Дополнительные функции**| Новости, локальные рейтинги, аналитика                        | Биографии, история производства, пользовательские рейтинги | Сообщество, пользовательские обзоры                | Метакритика, система оценок критиков           |


Таким образом, выбор Кинопоиска обусловлен его уникальной способностью предоставлять подробные и актуальные данные, адаптированные под нужды российской аудитории, а также возможностью параллельного доступа к данным с помощью API (через [kinopoisk.dev](https://kinopoisk.dev/)). Это делает его **более подходящим** для проектов, ориентированных на русскоязычных пользователей, чем глобальные аналоги.

## Сбор данных

Для доступа к данным используется сервис [kinopoisk.dev](https://kinopoisk.dev/). Благодаря возможности использовать несколько API-ключей и параллельной обработке запросов, удалось получить данные о 218750 произведениях, выпущенных до 2025 года и имеющих непустые описания.

> **Важно:** При парсинге данных также получались картины, которые ещё не вышли или имели пустое описание. Такие данные не подходят для обучения модели поиска по описанию, жанрам и для работы рекомендательной системы, поэтому они были убраны из общего списка фильмов.

Для получения данных были созданы **8 API-ключей**.  
Всего в Кинопоиске доступно **32 жанра**. Данные получались постранично: за один запрос можно получить до 250 записей, после чего производится переход на следующую страницу.  

Также были реализован **скрипт для очистки данных**: проверка наличия обязательных полей (`description`, `name`, `year`) и устранение дублирующихся записей (одна и та же картина может попадать в несколько жанров).

**Результаты сбора данных:** 
Общее количество фильмов: **102341**

<details><summary>Количество фильмов по жанрам:</summary>
	
	   • аниме: 4213 фильмов
	   • биография: 5071 фильмов
	   • боевик: 10240 фильмов
	   • вестерн: 1764 фильмов
	   • военный: 4193 фильмов
	   • детектив: 9941 фильмов
	   • детский: 2094 фильмов
	   • для взрослых: 479 фильмов
	   • документальный: 6812 фильмов
	   • драма: 7211 фильмов
	   • игра: 113 фильмов
	   • история: 2132 фильмов
	   • комедия: 6729 фильмов
	   • концерт: 100 фильмов
	   • короткометражка: 4323 фильмов
	   • криминал: 4220 фильмов
	   • мелодрама: 5163 фильмов
	   • музыка: 1761 фильмов
	   • мультфильм: 2019 фильмов
	   • мюзикл: 1614 фильмов
	   • новости: 33 фильмов
	   • приключения: 3789 фильмов
	   • реальное ТВ: 509 фильмов
	   • семейный: 3657 фильмов
	   • спорт: 932 фильмов
	   • ток-шоу: 78 фильмов
	   • триллер: 3009 фильмов
	   • ужасы: 5589 фильмов
	   • фантастика: 2565 фильмов
	   • фильм-нуар: 153 фильмов
	   • фэнтези: 1813 фильмов
	   • церемония: 22 фильмов
	  
</details>

<details><summary>Пример данных: </summary>

```json
{
      "id": 7109663,
      "name": "Парадоксальный навык «Мастер фруктов»: Навык, позволяющий есть бесконечное число фруктов (правда, вы умрёте, лишь откусив их)",
      "alternativeName": "Hazure Skill «Kinomi Master»: Skill no Mi (Tabetara Shinu) wo Mugen ni Taberareru You ni Natta Ken ni Tsuite",
      "type": "anime",
      "typeNumber": 4,
      "year": 2024,
      "description": "Есть мир, где любой человек может получить особую способность, съев фрукт навыка. Но сделать это можно лишь один раз в жизни, а во второй раз обязательно умрёшь от отравления. \nЛайт Андервуд мечтал стать лучшим на свете авантюристом, однако ему, как назло, попался навык «Мастер фруктов» — совершенно не боевая способность, которая сгодится разве что сад выращивать. Его подруге детства Лене попался редкий и мощный навык «Святая меча», и её сразу же отправили в столицу и назначили авантюристкой S-ранга, а Лайт остался дома и начал заниматься фермерством. Однажды он случайно съедает второй фрукт навыка, но не умирает. Оказывается, его способность позволяет есть сколько угодно фруктов навыка. Так начинается его история успеха и путь к исполнению мечты.",
      "shortDescription": "Фермер узнает, что может мгновенно овладеть любым мастерством. Фэнтези-аниме о начинающем искателе приключений",
      "status": null,
      "rating": {
        "kp": 7.418,
        "imdb": 6.2,
        "filmCritics": 0,
        "russianFilmCritics": 0,
        "await": null
      },
      "votes": {
        "kp": 4069,
        "imdb": 198,
        "filmCritics": 0,
        "russianFilmCritics": 0,
        "await": 0
      },
      "movieLength": null,
      "totalSeriesLength": null,
      "seriesLength": 23,
      "ratingMpaa": null,
      "ageRating": 18,
      "poster": {
        "url": "https://image.openmoviedb.com/kinopoisk-images/4716873/fdd65c27-9937-4a71-b3d2-144098b3d80a/orig",
        "previewUrl": "https://image.openmoviedb.com/kinopoisk-images/4716873/fdd65c27-9937-4a71-b3d2-144098b3d80a/x1000"
      },
      "genres": [
        {
          "name": "аниме"
        },
        {
          "name": "мультфильм"
        },
        {
          "name": "фэнтези"
        },
        {
          "name": "боевик"
        },
        {
          "name": "приключения"
        }
      ],
      "countries": [
        {
          "name": "Япония"
        }
      ],
      "releaseYears": [
        {
          "start": 2024,
          "end": null
        }
      ],
      "top10": null,
      "top250": null,
      "isSeries": true,
      "ticketsOnSale": false,
      "backdrop": {
        "previewUrl": "https://image.openmoviedb.com/kinopoisk-ott-images/374297/2a00000194d147908e013bba964ea52f4012/x1000",
        "url": "https://image.openmoviedb.com/kinopoisk-ott-images/374297/2a00000194d147908e013bba964ea52f4012/orig"
      }
    }
```
</details>

# Реализация 

## Принцип работы системы

![image](https://github.com/user-attachments/assets/61c2c437-76ba-4c43-b7ae-0b361919b788)

**Подготовка данных:**
- Сбор информации о фильмах из открытых источников
- Нормализация и очистка данных
- Генерация эмбеддингов для всех фильмов
- Индексация данных в MongoDB и Redis

**Обработка запроса пользователя:**
- Пользователь вводит текстовое описание желаемого фильма в интерфейсе
- Запрос преобразуется в векторное представление
- Система выполняет поиск по косинусному сходству
- Топ-20 фильмов обрабатываются LLM для проверки релевантности
- Результаты отображаются пользователю

**Дополнительная обработка:**
- Если релевантных результатов недостаточно, система расширяет поиск
- Исключаются уже проверенные фильмы
- Процесс повторяется до получения достаточного количества релевантных результатов


## Структура проекта
Проект состоит из трех основных микросервисов:

- **База данных:**
  MongoDB для хранения полной информации о фильмах
Redis с модулем RediSearch для быстрого поиска и кэширования

- **Поисковый сервис:**
  Реализует векторное представление текстовых описаний
Осуществляет семантический поиск на основе косинусного сходства
Выполняет дополнительную обработку результатов через LLM

- **Веб-интерфейс:**
  Flask-приложение для реализации API
Клиентская часть для взаимодействия с пользователем

## Файловая структура
```app.py``` - Основное Flask-приложение, обрабатывающее запросы пользователей

```search_service.py``` - Сервис для семантического поиска фильмов

```mongo_client.py``` - Клиент для работы с MongoDB

```redis_client.py``` - Клиент для работы с Redis/RediSearch

```migrate_mongo_to_redis.py``` - Скрипт для миграции данных между БД (MongoDB -> Redis)

```templates/``` - Шаблоны HTML для веб-интерфейса

```static/``` - Статические файлы (CSS, JavaScript, изображения)

```model_cache/``` - Каталог для кэширования моделей

## Алгоритм поиска
**1. Предобработка данных:**
- Загрузка базы фильмов из MongoDB
- Генерация векторных представлений для каждого фильма с помощью SentenceTransformer
- Индексация эмбеддингов в FAISS для быстрого поиска по косинусному сходству

**2. Обработка запроса пользователя:**
- Преобразование запроса в векторное представление
- Анализ запроса на наличие фильтров (год, жанр)
- Семантический поиск по базе фильмов

**3. Уточнение результатов:**
- Ранжирование результатов с учетом текстового сходства (85%), года выпуска (5%) и жанра (10%)
- Отбор топ-20 результатов для дальнейшей обработки
- Обработка LLM-моделью для проверки соответствия между запросом и описанием фильма

**4. Дополнительный поиск (при необходимости):**
- Если количество релевантных результатов после обработки LLM недостаточно, система расширяет поиск
- Исключаются уже обработанные фильмы и процесс повторяется

![image](https://github.com/user-attachments/assets/691e5c17-71b0-4930-9a60-5a4754843f27)

##  Используемый стек технологий
### Языки программирования и фреймворки
- **Python 3.10**
- **Flask** для реализации веб-приложения

### Базы данных
- **MongoDB** - документоориентированная БД для хранения данных о фильмах
- **Redis** с модулем RediSearch - для быстрого поиска и кэширования

### Машинное обучение
- **SentenceTransformer** (multilingual-e5-large-instruct) - модель для создания векторных представлений текста
- **FAISS** (Facebook AI Similarity Search) - библиотека для эффективного поиска по косинусному сходству
- **Языковая модель** (LLM) для уточнения результатов поиска

### Дополнительные библиотеки
- **NumPy** - для вычислений с векторами
- **PyMongo** - клиент для работы с MongoDB
- **Redis-py** - клиент для работы с Redis
- **scikit-learn** - для предобработки данных и нормализации

## Детали реализации моделей машинного обучения
### Векторизация текста
Для преобразования текстовых описаний фильмов в векторные представления используется модель **SentenceTransformer** (paraphrase-multilingual-mpnet-base-v2). Эта модель особенно эффективна для многоязычных текстов и способна улавливать семантическое сходство между фразами, даже если они используют разные слова для описания одной и той же концепции.
**Процесс векторизации:**
- Загрузка предобученной модели SentenceTransformer
- Обработка названий и описаний фильмов для получения эмбеддингов
- Нормализация векторов для обеспечения эффективного косинусного сходства
- Сохранение эмбеддингов в файл для повторного использования

### Индексация и поиск
Для быстрого поиска по векторным представлениям используется библиотека **FAISS**, которая эффективно находит ближайшие векторы по **косинусному сходству**:
- Создание индекса FAISS типа IndexFlatL2
- Добавление нормализованных эмбеддингов в индекс
- Поиск ближайших соседей при обработке запросов пользователей

### Ранжирование результатов
В системе применяется **комбинированный алгоритм ранжирования**, учитывающий:
- Семантическое сходство текста (85% веса)
- Соответствие года выпуска (5% веса)
- Соответствие жанра (10% веса)

### Обработка LLM
Для уточнения релевантности результатов используется **языковая модель**, которая:
- Анализирует соответствие между запросом пользователя и описанием фильма
- Оценивает релевантность каждого фильма из топ-20 результатов
- Отфильтровывает нерелевантные результаты, повышая точность поиска

### Сравнение эмбеддинг моделей 

Мы сравнили эмбеддинг модели на бенчмарке [MTEB(Multilingual, v1)](https://huggingface.co/spaces/mteb/leaderboard)

Сравнение было проведено для русского языка и задач:

- Классификации
- Кластеризации
- Мультилейбловая классификация
- Поиска


| Модель| Количество параметров | Среднее значение по задаче | Среднее значение по типу задачи | Классификация | Кластеризации | Мультилейбловая классификация | Поиск |
|--------------------------------------------|-------------|-------------|-----------------|----------------|------------|---------------------------|-----------| 
| multilingual-e5-large-instruct | 560M | **67.63** | **64.25** | **74.30** | **58.17** | **50.01** | 74.51  |
|jina-embeddings-v3 | 572M | 61.33| 57.57| 58.93 | 45.17 | 47.42 | **78.75** |
| paraphrase-multilingual-mpnet-base-v2 | 278M | 48.37 | 46.29 | 48.48 | 40.05 | 39.98 | 56.62 | 
| ru-en-RoSBERTa | 404M| 58.69 | 55.52 | 60.6 | 47.34 | 44.69 | 69.47 | 
| rubert-tiny-turbo | 29M | 51.81 | 48.59 | 59.61 | 38.93 | 38.95 | 56.87 | 
| sbert_large_nlu_ru | 427M | 39.89 | 40.19 | 57.61 | 46.34 | 35.84 | 20.97 | 
| rubert-tiny2 | 29M | 38.44 | 38.02 | 50.42 | 36.67 | 36.87 | 28.13 |

На основе данного сравнения нами была выбрана модель `multilingual-e5-large-instruct`,так как при небольшом количестве параметров(не больше 1B) она показала наилучший результат для русского языка и выбранных нами задач.

## Создание сайта и пользовательский путь
Пользователь заходит на главную страницу, вводит название фильма в строку поиска, после чего происходит перенаправление на страницу с результатами (`/dml?query=...`).  
На странице результатов пользователь может дополнительно фильтровать фильмы по жанрам, году выпуска и типу, а при клике на карточку фильма открывается модальное окно с подробной информацией и списком рекомендованных фильмов.

![Studio-Display](https://github.com/user-attachments/assets/344b9dec-e724-42b3-91ae-df05a2c5a15a)

### Опрос
Мы провели опрос с целью выяснения насколько удобно пользоваться нашим поисковиком и насколько правильно он подбирает фильмы.
- 94 участника ответили, что поиск работает `Хорошо`
- 15 участников ответили, что поиск требуется доработки

![screenshot](https://github.com/user-attachments/assets/cd5b2ee3-b001-4266-b29e-e5b1fb544cca)


---

