# lab_04_1
## Чтение и обработка данных из файла.

## Цель:
Научиться работать с файлами, выполнять их чтение, запись и манипуляцию, а также освоить методы сериализации и десериализации данных с использованием популярных форматов хранения, таких как CSV и JSON.

## Задачи:
1.Изучить структуру файловой системы, свойства файлов, виды путей к файлам (абсолютный и относительный).
2. Освоить операции с файлами в Python: открытие, чтение, запись, закрытие.
3. Разобраться с файловыми объектами и их основными свойствами и методами.
4. Изучить методы работы с файлами: чтение файла целиком, построчное чтение, запись строк в файл.
5. Познакомиться с концепцией сериализации и десериализации данных.
6. Изучить популярные форматы сериализации: CSV и JSON.
7. Научиться использовать модуль pickle для сериализации данных в Python.
8. Решить практические задачи по обработке файлов и сериализации данных.


## 8 вариант
## Задание 4.1.8. Предоставив пользователю выбрать 2 различных года для сравнения, сформируйте JSON-файл, который содержит абсолютные значения (для второго года) и изменения по количеству заболеваний в процентах (округление до 2-х знаков после запятой). Значения должны быть сгруппированы по категории Болезнь и идти по убыванию.
## Решение:

```
import csv
import json


class NoSuchYearError(Exception):
    def __init__(self, message):
        super().__init__(message)


def load_data(filename):

    data = []
    # Загрузка файла
    with open(filename, 'r', encoding='utf-8') as file:
        reader = csv.reader(file)
        for row in reader:
            data.append(row)
    
    # Преобразование строк в числа (кроме первого столбца и первой строки)
    for i in range(1, len(data)):
        for j in range(1, len(data[i])):
            if data[i][j].strip():  # Проверяем, что значение не пустое
                try:
                    data[i][j] = int(data[i][j])
                except ValueError:
                    # Если не удается преобразовать в int, оставляем как строку
                    pass
    
    return data


def export(filename, data, year_1, year_2):

    # Проверка параметров 'year_1' и 'year_2'
    headers = data[0]
    available_years = [int(year) for year in headers[1:-1] if str(year).isdigit()]
    
    if year_1 not in available_years:
        raise NoSuchYearError(f"Год {year_1} не найден в данных. Доступные годы: {available_years}")
    if year_2 not in available_years:
        raise NoSuchYearError(f"Год {year_2} не найден в данных. Доступные годы: {available_years}")
    
    if year_1 == year_2:
        raise NoSuchYearError("Годы для сравнения должны быть разными")
    
    # Находим индексы годов в заголовках
    year_1_index = headers.index(str(year_1))
    year_2_index = headers.index(str(year_2))
    
    # Разделяем данные на взрослых и детей
    adult_data = []
    children_data = []
    
    for i in range(1, len(data)):
        row = data[i]
        disease_class = row[0]
        age_group = row[-1]  # Последний столбец - возрастная группа
        
        # Получаем значения для обоих годов
        try:
            value_year_1 = int(row[year_1_index]) if row[year_1_index] else 0
            value_year_2 = int(row[year_2_index]) if row[year_2_index] else 0
        except (ValueError, TypeError):
            continue  # Пропускаем некорректные данные
        
        # Рассчитываем процентное изменение
        if value_year_1 != 0:
            percentage_change = ((value_year_2 - value_year_1) / value_year_1) * 100
        else:
            percentage_change = 0.0  # Если в первом году было 0, изменение 0%
        
        disease_info = {
            "класс болезни": disease_class,
            "значение": value_year_2,
            "изменение": round(percentage_change, 2)
        }
        
        if "дети" in age_group.lower() or "детск" in age_group.lower():
            children_data.append(disease_info)
        else:
            adult_data.append(disease_info)
    
    # Формирование списков
    # Сортируем по убыванию значений для второго года
    data_year_2_sorted_adult = sorted(
        [{"класс болезни": item["класс болезни"], "значение": item["значение"]} 
         for item in adult_data],
        key=lambda x: x["значение"], 
        reverse=True
    )
    
    data_year_2_sorted_children = sorted(
        [{"класс болезни": item["класс болезни"], "значение": item["значение"]} 
         for item in children_data],
        key=lambda x: x["значение"], 
        reverse=True
    )
    
    # Сортируем по убыванию процентных изменений
    data_changes_sorted_adult = sorted(
        [{"класс болезни": item["класс болезни"], "изменение": item["изменение"]} 
         for item in adult_data],
        key=lambda x: x["изменение"], 
        reverse=True
    )
    
    data_changes_sorted_children = sorted(
        [{"класс болезни": item["класс болезни"], "изменение": item["изменение"]} 
         for item in children_data],
        key=lambda x: x["изменение"], 
        reverse=True
    )
    
    res = {
        "Второй год": {
            "Взрослые": data_year_2_sorted_adult,
            "Дети": data_year_2_sorted_children
        },
        "Изменения": {
            "Взрослые": data_changes_sorted_adult,
            "Дети": data_changes_sorted_children
        },
        "Информация": {
            "Период сравнения": f"{year_1} → {year_2}",
            "Всего заболеваний у взрослых": len(adult_data),
            "Всего заболеваний у детей": len(children_data)
        }
    }

    # Сохранение в файл
    with open(filename, 'w', encoding='utf-8') as file:
        json.dump(res, file, ensure_ascii=False, indent=2)


# Основная программа с обработкой исключений
try:
    filename = input("Введите имя файла: ")
    export_filename = input("Введите имя файла для экспорта: ")

    # Загрузка данных
    data = load_data(filename)
    print(f"Данные успешно загружены. Строк: {len(data)}")
    
    # Показываем доступные годы
    headers = data[0]
    available_years = [year for year in headers[1:-1] if str(year).isdigit()]
    print(f"Доступные годы для анализа: {available_years}")
    
    # Ввод годов для сравнения
    year_input = input("Введите два года для сравнения через пробел: ").split()
    if len(year_input) != 2:
        print("Ошибка: нужно ввести ровно два года!")
    else:
        year_1, year_2 = int(year_input[0]), int(year_input[1])
        
        # Экспорт данных
        export(export_filename, data, year_1, year_2)
        print(f"Данные успешно экспортированы в файл: {export_filename}")
        print(f"Сравнение: {year_1} год → {year_2} год")
        
        # Дополнительная информация
        print(f"\nСтруктура JSON-файла:")
        print("- Второй год: данные за {year_2} год (отсортированы по убыванию)")
        print("- Изменения: процентные изменения {year_1}→{year_2} (отсортированы по убыванию)")
        print("- Разделы: Взрослые и Дети")

except FileNotFoundError:
    print(f"Ошибка: Файл '{filename}' не найден!")
except PermissionError:
    print(f"Ошибка: Нет прав доступа к файлу!")
except NoSuchYearError as e:
    print(f"Ошибка выбора года: {e}")
except ValueError as e:
    print(f"Ошибка ввода: введите корректные числовые значения для годов!")
except Exception as e:
    print(f"Произошла непредвиденная ошибка: {e}")
```

Результат увидим на экране:


<img width="892" height="183" alt="Screenshot_78" src="https://github.com/user-attachments/assets/61ad6aa9-f2f5-40e5-be71-9d0bb9437bb2" />


## Вывод:
В ходе выполения лабораторной работы я научилась работать с файлами, выполнять их чтение, записывать и проводить манипуляцию, а также освоила методы сериализации и десериализации данных с использованием популярных форматов хранения, таких как CSV и JSON.
