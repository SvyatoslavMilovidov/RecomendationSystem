# Рекомендательная система

<img src="https://github.com/SvyatoslavMilovidov/RecomendationSystem/blob/main/plots_and_diagrams/diagram_1.png" alt="Header">

* #### __Решаемая задача__ — по запросу рекомендовать пользователю 5 постов, максимизируя при этом hitrate.
* #### __Используемый стек__ — `Python` `FastAPI` `Psycopg2` `SQLAlchemy` `Pandas` `Numpy` `Sklearn` `CatBoost` `Transformers`

## [`Подключение к базе и выгрузка данных`](https://github.com/SvyatoslavMilovidov/RecomendationSystem/blob/main/connect_database.py "посмотреть код")

* ❗ Перед использованием кода необходимо указать URI вашей базы данных в переменной DATABASE_URI.
* С целью практики в этом файле реализовано два способа подключения к базе данных: через psycopg и sqlalchemy.
* `get_data_with_psycopg` используется для выгрузки уже предобработанных данных.
* `get_data_with_sqlalchemy` используется для выгрузки исходных данных с последующей предобработкой. 

## [`Предобработка данных`](https://github.com/SvyatoslavMilovidov/RecomendationSystem/blob/main/preprocessing_data.py "посмотреть код")

1. Подготовка данных про пользователей:
    - Все признаки, в которых менее 15 уникальных значений будем кодировать через `OneHotEnсoding`.
    - Те признаки, в которых больше через `LabelEncoder`.
      
2. Подготовка текстов:
    - Удаляем лишние символы (пробелов, знаков препинания).
    - Удаляем стоп-слова, с помощью библиотеки `nltk` импортируем список неважных слов для английского языка и убираем их.
    - Лемматизируем, используем `WordNetLemmatizer` из библиотеки `nltk` для приведения всех слов к общей форме.
      
3. Достаем признаки из TF-IDF:
    - Формируем большую матрицу с TF-IDF коэффициентами для каждого слова по всем текстам, в итоговой таблице более чем 60 тысяч признаков.
    - Вычитаем из них среднее и уменьшаем их размерность с помощью метода главных компонент `PCA`, на выходе имеем 50 признаков.
    - Кластеризуем их с помощью `KMeans`, получим разбиение на 15 кластеров.
    - Считаем расстояние до каждого кластера.
      
4. Извлекаем признаки из эмбеддингов:
    - Загружаем эмбеддинги (процесс их извлечения рассмотрен в следующей главе).
    - Вычитаем среднее и уменьшаем размерность до 50 колонок.
    - Выделяем 15 кластеров через `KMeans` и считаем расстояния до кластеров.
   
## [`Получение эмбеддингов постов с помощью BERT`](https://github.com/SvyatoslavMilovidov/RecomendationSystem/blob/main/get_embeddings_with_BERT.py "посмотреть код")

* Подготавливаем тексты.
* Ипользуем предобученую модель и токенайзер.
* Токенизируем слова на стадии создания датасета.
* Проходимся по текстам и получаем эмбеддинги.
* Загружаем их в папку "BERT_embeddings".

## [`Обучение моделей`](https://github.com/SvyatoslavMilovidov/RecomendationSystem/blob/main/Recommender_system/train_models.ipynb "посмотреть ноутбук")

1. **Первая модель**
   
   <img src="https://github.com/SvyatoslavMilovidov/RecomendationSystem/blob/main/plots_and_diagrams/control_model.jpeg" width="500">
   
      - для обучения использованы только признаки из TF-IDF.
      - кластеризация с помощью KMeans.
  
3. **Вторая модель** - использованы TF-IDF и фичи из эмбеддингов.
      
   <img src="https://github.com/SvyatoslavMilovidov/RecomendationSystem/blob/main/plots_and_diagrams/test model.jpeg" width="500">
   
      - Использованы признаки из эмбеддингов и TF-IDF.
      - Кластеризация с помощью DBSCAN и KMeans.
        
* Данные про пользователей одинаковы.

## [`Анализ результатов A/B тестирования`](https://github.com/SvyatoslavMilovidov/RecomendationSystem/blob/main/Recommender_system/analysis_of_ab_test_results.ipynb "посмотреть ноутбук")

* Необходимо было понять значимо ли различаются модели по целевым метрикам:
     - Hitrate

       <img src="https://github.com/SvyatoslavMilovidov/RecomendationSystem/blob/main/plots_and_diagrams/hitrate_models.jpeg">

     - Число лайков на пользователя
 
       <img src="https://github.com/SvyatoslavMilovidov/RecomendationSystem/blob/main/plots_and_diagrams/count_likes_per_user.jpeg">
  
* В результате анализа задетектированы статистически значимые различия.
* Hitrate лучшей модели на тестовых данных составил __0.6__.
