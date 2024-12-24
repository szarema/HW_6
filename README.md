# Инструкция по загрузке данных в Greenplum через gpfdist

## 1. Загрузка датасета

1. Скачываем датасет из Kaggle. Например, воспользуемся файлом `Movies.csv` из скаченного архива which-movie-should-i-watch-today.zip:

   ```bash
   curl -L -o ~/Downloads/which-movie-should-i-watch-today.zip  https://www.kaggle.com/api/v1/datasets/download/hassanelfattmi/which-movie-should-i-watch-today
   ```

2. Вручную разархивируем архив в папку ~/Downloads/movies

3. Копируем файл на сервер с Greenplum

   ```bash
   scp ~/Downloads/movies/Movies.csv user@91.185.85.179:/home/user/team-5-data
   ```

## 2. Настройка и загрузка данных через gpfdist

### 2.1 Настройка gpfdist

1. Подключаемся к серверу Greenplum:

   ```bash
   ssh user@91.185.85.179
   ```

2. Запуск gpfdist в папке, где хранится файл `Movies.csv`:

   ```bash
   gpfdist -d /home/user/team-5-data -p 8080 &
   ```

   Здесь:
   - `-d` указывает директорию, где хранятся файлы.
   - `-p` задает порт (например, `8080`).

### 2.2 Создание EXTERNAL таблицы

1. Подключаемся к базе данных Greenplum:

   ```bash
   psql -d idp
   ```

2. Создадим внешнюю таблицу:

   ```sql
   DROP FOREIGN TABLE IF EXISTS team_5_movies;

   CREATE EXTERNAL TABLE team_5_movies (
        id INT,
        title TEXT,
        genres TEXT,
        language CHAR(2),
        user_score DECIMAL(3,1),
        runtime_hour INT,
        runtime_min INT,
        release_date DATE,
        vote_count INT
   )
    LOCATION ('gpfdist://localhost:8080/Movies.csv')
    FORMAT 'CSV' (HEADER DELIMITER ',' NULL 'NULL')
    ENCODING 'UTF8';
   ```

### 2.3 Загрузка данных во внутреннюю таблицу

1. Создание таблицу:

   ```sql
   DROP TABLE IF EXISTS team_5_movies_internal;

   CREATE TABLE team_5_movies_internal (
        id INT,
        title TEXT,
        genres TEXT,
        language CHAR(2),
        user_score DECIMAL(3,1),
        runtime_hour INT,
        runtime_min INT,
        release_date DATE,
        vote_count INT
   )
   DISTRIBUTED BY (id);
   ```

2. Загрузка данные:

   ```sql
   INSERT INTO team_5_movies_internal
   SELECT * FROM team_5_movies;
   ```

## 3. Проверка загруженных данных

### 3.1 Проверка внешней таблицы

Выполнение SELECT на внешней таблице:

```sql
SELECT * FROM team_5_movies LIMIT 10;
```

### 3.2 Проверка внутренней таблицы (если данные были загружены)

```sql
SELECT * FROM team_5_movies_internal LIMIT 10;
```

## 4. Завершение работы

### 4.1 Остановка gpfdist

После завершения работы останавливаем gpfdist:

```bash
pkill -f gpfdist
```
