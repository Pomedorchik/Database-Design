# Домашняя работа - Нормализация таблицы StudentGrades

## Часть 1. Теоретический анализ

### 1. Первичный ключ

В исходной таблице нет отдельного столбца с номером записи, поэтому в качестве первичного ключа можно использовать составной ключ:

`student_id + subject_id + exam_date`

Он означает, что один студент не может иметь две одинаковые записи по одному предмету в один и тот же день.

`student_id` отдельно не подходит, потому что один студент может сдавать несколько предметов.

`subject_id` тоже не подходит, потому что один предмет могут сдавать разные студенты.

`exam_date` также не подходит, потому что в один день могут проходить несколько экзаменов.

Поэтому для исходной таблицы подходит составной первичный ключ из трех полей.

### 2. Функциональные зависимости

В таблице есть следующие основные функциональные зависимости:

```text
student_id -> student_name
student_id -> group_id
group_id -> group_name
teacher_id -> teacher_name
subject_id -> subject_name
subject_id -> teacher_id
```

Оценка зависит от конкретного студента, предмета и даты экзамена:

```text
student_id + subject_id + exam_date -> grade
```

### 3. Транзитивные зависимости

В таблице есть несколько транзитивных зависимостей.

Например:

```text
student_id -> group_id
group_id -> group_name
```

Поэтому получается:

```text
student_id -> group_name
```

Название группы определяется через `group_id`, поэтому хранить его вместе со студентом не нужно.

Также существуют зависимости:

```text
teacher_id -> teacher_name
subject_id -> subject_name
```

Из-за таких зависимостей в одной таблице появляется много повторяющейся информации.

### 4. Нормальная форма исходной таблицы

Исходная таблица находится в первой нормальной форме.

Все значения являются атомарными, то есть в одной ячейке хранится одно значение.

Но таблица не соответствует второй нормальной форме, потому что некоторые поля зависят только от части составного ключа.

Например:

```text
student_id -> student_name
subject_id -> subject_name
```

Также таблица не соответствует третьей нормальной форме, потому что в ней есть транзитивные зависимости:

```text
student_id -> group_id -> group_name
```

Итог:

`Исходная таблица - 1НФ`

## Часть 2. Практическая нормализация

Для приведения исходной таблицы к третьей нормальной форме я разделил ее на несколько связанных таблиц.

Итоговый список таблиц:

- `Groups`
- `Students`
- `Teachers`
- `Subjects`
- `StudentGrades`

### Схема связей

```text
Groups
   |
   | 1:N
   |
Students
   |
   | 1:N
   |
StudentGrades
   |
   | N:1
   |
Subjects
   |
   | N:1
   |
Teachers
```

Одна группа может содержать много студентов.

Один студент может иметь много оценок.

Один предмет может встречаться во многих записях с оценками.

Один преподаватель может преподавать предмет.

## Таблица Groups

Хранит информацию о группах.

```sql
CREATE TABLE Groups (
    group_id VARCHAR(10) PRIMARY KEY,
    group_name VARCHAR(100) NOT NULL UNIQUE
);
```

`group_id` - первичный ключ.

`group_name` - название группы. Оно не может быть пустым и должно быть уникальным.

## Таблица Students

Хранит информацию о студентах.

```sql
CREATE TABLE Students (
    student_id INT PRIMARY KEY,
    student_name VARCHAR(100) NOT NULL,
    group_id VARCHAR(10) NOT NULL,

    FOREIGN KEY (group_id) REFERENCES Groups(group_id)
);
```

`student_id` - первичный ключ.

`group_id` - внешний ключ, который связывает студента с его группой.

## Таблица Teachers

Хранит информацию о преподавателях.

```sql
CREATE TABLE Teachers (
    teacher_id INT PRIMARY KEY,
    teacher_name VARCHAR(100) NOT NULL
);
```

`teacher_id` - первичный ключ.

`teacher_name` - имя преподавателя.

## Таблица Subjects

Хранит информацию о предметах и преподавателях.

```sql
CREATE TABLE Subjects (
    subject_id INT PRIMARY KEY,
    subject_name VARCHAR(100) NOT NULL UNIQUE,
    teacher_id INT NOT NULL,

    FOREIGN KEY (teacher_id) REFERENCES Teachers(teacher_id)
);
```

`subject_id` - первичный ключ.

`subject_name` - название предмета.

`teacher_id` - внешний ключ, который связывает предмет с преподавателем.

## Таблица StudentGrades

Хранит результаты тестирования студентов.

```sql
CREATE TABLE StudentGrades (
    student_id INT NOT NULL,
    subject_id INT NOT NULL,
    exam_date DATE NOT NULL,
    grade INT NOT NULL,

    PRIMARY KEY (student_id, subject_id, exam_date),

    FOREIGN KEY (student_id) REFERENCES Students(student_id),
    FOREIGN KEY (subject_id) REFERENCES Subjects(subject_id)
);
```

`student_id` показывает, какой студент сдавал экзамен.

`subject_id` показывает, какой предмет сдавался.

`exam_date` показывает дату экзамена.

`grade` содержит полученную оценку.

Составной первичный ключ:

```text
student_id + subject_id + exam_date
```

не позволяет создать две одинаковые записи для одного студента, одного предмета и одной даты.

## Итог

После нормализации исходная таблица была разделена на пять связанных таблиц:

| Таблица | Назначение |
|---|---|
| `Groups` | Хранит группы |
| `Students` | Хранит студентов и их группы |
| `Teachers` | Хранит преподавателей |
| `Subjects` | Хранит предметы и преподавателей |
| `StudentGrades` | Хранит оценки студентов |

В результате мы убрали повторяющиеся данные и сделали структуру базы данных более удобной для работы. Теперь, если нужно изменить название группы или имя преподавателя, достаточно изменить его в одном месте.
