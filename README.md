#  <center >Исследование и работа с данными С сайта HH.ru <center>
## <center>Описание проекта<center> 
В данном проекте выполнены ключевые этапы обработки данных:
* Исследование структуры данных
* Преобразование данных
* Исследование зависимостей в данных
* Очистка данных

Далее подробно рассмотрим каждый из этих этапов

##  <center>Исследование структуры данных<center>

На данном этапе были изучены особенности структуры **базы данных**. Основные действия:
* Определение размеров таблицы
    ```python 
    shape = data_resume.shape
    print(shape)
    ```
* Проверка типов данных в столбцах
    ```python
    # Вывод информации о таблице
    data_resume.info()
    ```
* Поиск пропусков в данных
    ```python
    # Поиск столбцов с пропусками
    columns_with_nulls = data_resume.columns[data_resume.isnull().any()].tolist()

    # Вывод столбцов с пропусками
    print("Столбцы с пропусками:")
    print(columns_with_nulls)
    ```
## <center>Преобразование данных<center>

**Преобразование данных** на этом этапе была проведена значительная работа по преобразованию данных, чтобы привести их к удобному для анализа формату. Рассмотрим ключевые изменения: 

1. Преобразовали признак **«Образование и ВУЗ»** в признак **«Образование»** где удалена лишняя информация, оставлен только уровень образования:

    * «высшее», «неоконченное высшее», «среднее специальное» и «среднее».

2. Разделен столбец **«Пол и возраст»** на 2 новых признака **«Пол»** и **«Возраст»**, с учётом следующих условий:

    * Пол представлен в виде строковых значений: "М" — мужчина, "Ж" — женщина.
    * Признак возраста должен быть представлен в виде целых чисел.

3. Следом преобразуем признак **«Опыт работы»**. Из столбца выделен общий стаж работы в месяцах, новый столбец назовём **«Опыт работы (месяц)»**. Учитывая следующие условия:

    * В данном признаке встречаются пропущенные значения, которые оставлены без изменений (NaN) (функция-преобразование возвращает NaN).
    * В данном столбце есть скрытые пропуски. В столбце стоят значения Не указано. Их тоже обозначим как NaN (функция-преобразование возвращает NaN)
    * Учитывается только общий стаж без разбивки по компаниям. Нас не интересует информация, которая описывается после указания опыта работы (периоды работы в различных компаниях).
    * В-четвёртых, у нас есть проблема: опыт работы может быть представлен только в годах или только в месяцах. Переведены года и месяцы в единую систему (в месяцах).

4. Из признака **«Город, переезд, командировки»** выделены три отдельных столбца **«Город»**, **«Готовность к переезду»** и **«Готовность к командировкам»**. При этом важно учесть:
    * Информация о метро, рядом с которым проживает соискатель, не интересует.
    * Интересует только сам факт возможности/желания переезда.
    * Интересует только сам факт готовности к командировке.

5. Далее признаки **«Занятость»** и **«График»**. Столбцы представляют собой категории желаемой занятости (полная занятость, частичная занятость, проектная работа, волонтёрство, стажировка) и желаемого графика работы (полный день, сменный график, гибкий график, удалённая работа, вахтовый метод). Воспользуемся методом преобразования категориальных признаков One Hot Encoding. И взглянем на нижеприведенную таблицу:

![](https://lms-cdn.skillfactory.ru/assets/courseware/v1/5461aeb8c39a8cb9731eb5f52b66f5a3/asset-v1:SkillFactory+DSPR-2.0+14JULY2021+type@asset+block/dst-3.0_16_3_1.png)

6. Признак заработной платы **«ЗП»**. Проблема в том, что соискатель указывает не только желаемую заработную плату, но и предпочитаемую валюту. Нам бы хотелось, чтобы зарплата была указана в единой валюте, например, в рублях. Возникает вопрос: где можно получить актуальный курс валют к рублю?

***Пропорция*** — это число, отражающее, скольким единицам валюты соответствует курс в таблице с курсами. Например для казахстанского тенге курс на 20.08.2019 составляет 17.197 руб. за 100 тенге, тогда итоговый курс равен: 17.197 / 100 = 0.17197 руб. за 1 тенге.
Пропорции с которыми будем работать в этом проекте.

![alt text](image.png)

Алгоритм преобразования:

    * Преобразовать признак "Обновление резюме" из таблицы с резюме в формат datetime, оставив только дату и отбросив время. Аналогичное преобразование выполнить для признака date в таблице с валютами. Важно учитывать исходный формат даты (день-месяц-год) при конвертации.
    * Выделить из столбца «ЗП» сумму желаемой заработной платы и наименование валюты, в которой она исчисляется. Название валюты привести к стандарту ISO в соответствии с таблицей выше.
    * Присоединить к таблице с резюме таблицу с курсами по столбцам с датой и названием валюты (подумайте, какой тип объединения надо выбрать, чтобы в таблице с резюме сохранились данные о заработной плате, изначально представленной в рублях). Значение close для рубля заполнить единицей 1 (курс рубля самого к себе).
    * Рассчитать сумму заработной платы в рублях, умножив указанную сумму на соответствующий курс валюты (close) и разделив на пропорцию. Обратить внимание на возможные пропуски после объединения. Полученное значение записать в новый столбец «ЗП (руб)».

## <center>Исследование зависимостей в данных<center>

Теперь у нас есть всё необходимое, чтобы провести первичный анализ зависимостей в наших данных о резюме. Такой анализ называют, разведывательный анализ (EDA), направленный на выявление взаимосвязей между признаками, закономерностей, анализ распределений, а также поиск аномалий.

Подробные результаты и графики представлены в ноутбуке (Project_data_base) и папке проекта (graph).

## <center>Очистка данных<center>

Для очистки данных выполнены следующие шаги: 

* Начнём с дубликатов в наших данных.
    ```python
    # Найдем и удалим полные дубликаты
    duplicates_removed_df = data_resume.drop_duplicates()

    # Определим количество удаленных дубликатов
    num_duplicates_removed = len(data_resume) - len(duplicates_removed_df)

    print("Количество полных дубликатов, найденных и удаленных:", num_duplicates_removed)
    ```

* Займёмся пропусками. Выводим информацию о числе пропусков в столбцах.
    ```python
    # Вывод информации о числе пропусков в столбцах
    missing_values_info = duplicates_removed_df.isnull().sum()

    # Вывод числа пропусков в столбце "Опыт работы (месяц)"
    experience_missing_values = missing_values_info['Опыт работы (месяц)']

    print("Информация о числе пропусков в столбцах:")
    print(missing_values_info)
    print("\nКоличество пропусков в столбце 'Опыт работы (месяц)':", experience_missing_values)
    ```

* Среднее значение в столбце «Опыт работы (месяц)» после заполнения пропусков.
    ```python
    # Удаление строк с пропусками в столбцах с местом работы и должностью
    data_resume.dropna(subset=['Последнее/нынешнее место работы', 'Последняя/нынешняя должность'], inplace=True)

    # Заполнение пропусков в столбце с опытом работы медианным значением
    median_experience = data_resume['Опыт работы (месяц)'].median()
    data_resume['Опыт работы (месяц)'].fillna(median_experience, inplace=True)

    # Расчет среднего значения в столбце "Опыт работы (месяц)"
    average_experience = round(data_resume['Опыт работы (месяц)'].mean())

    print("Результирующее среднее значение в столбце 'Опыт работы (месяц)' после заполнения пропусков:", average_experience)
    ```

* Находим выбросы:

    ``` python
    # Фильтрация данных для удаления выбросов
    outliers_removed_df = data_resume[
        (data_resume['ЗП (руб)'] >= 1000) & 
        (data_resume['ЗП (руб)'] <= 1000000)
    ]

    # Подсчет количества выбросов
    num_outliers = len(data_resume) - len(outliers_removed_df)

    print("Количество выбросов, удаленных из данных:", num_outliers)
    ```

* В процессе разведывательного анализа мы обнаружили резюме, в которым **опыт работы в годах превышал возраст соискателя**. Найдем количество таких значений.

    ```python
    # Найти резюме, где опыт работы в годах превышает возраст соискателя
    outliers = data_resume[(data_resume['Опыт работы (месяц)'] / 12) > data_resume['Возраст']]

    # Подсчитать количество найденных выбросов
    num_outliers = len(outliers)

    # Удалить выбросы из данных
    cleaned_df = data_resume.drop(outliers.index)

    print("Количество выбросов, найденных и удаленных:", num_outliers)
    ```

* Заключительный этап очистки данных — определение количества выбросов с использованием метода z-отклонений. Прежде чем применять этот метод, важно разобраться, что такое z-отклонения и как они работают.
На примере графика рассмотрим z-отклонения.

![](https://www.drdawnwright.com/wp-content/uploads/2019/09/IQ4b.png)

***Метод Z-отклонений*** — это статистический метод выявления выбросов в данных. Он включает в себя вычисление стандартного отклонения данных, а затем определение количества стандартных отклонений от среднего значения каждой точки данных. Точки данных, которые отличаются от среднего значения более чем на определенное количество стандартных отклонений, считаются выбросами.

# <center>Используемые библиотеки<center>

Все что использовалось находтися в файле [requirements]

# <center>Ссылка на проект на сервисе Google Диск <center>

[google_disk](https://drive.google.com/drive/folders/1QVXau_XPwjjet4Cl2eyiuZNLSa512M0Y?usp=sharing)