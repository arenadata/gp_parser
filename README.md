## Что это?

Это небольшая библиотека, которая позволяет посмотреть синтаксическое дерево разбора запроса парсером в greenplum. Для использования нужно скомпилировать библиотеку `annotate_query.so` на мастере greenplum, распространить ее по кластеру (например, через gpscp) и создать обертку в виде sql функции для работы с библиотекой.

## Как это использовать?

Все шаги ниже производятся на мастере.

* Перед компиляцией библиотеки нам потребуется файл `outfuncs.c` из исходников greenplum для отображения дерева разбора запроса в текстовом виде. Поэтому, придется рядом положить исходники greenplum с HEAD установленным на коммит или тег вашей версии сборки (так как вдруг что-то поменялось в функционале отображения дерева разбора в текст). Коммит или тег можно посмотреть выполнив `select version();` в greenplum.
```bash
git clone https://github.com/darthunix/gp_parser.git
cd gp_parser
# Создаем каталог для исходных кодов gpdb и вытягиваем их в минималистичном варианте для нашего тега или коммита
mkdir gpdb
cd gpdb
git init
git remote add origin https://github.com/greenplum-db/gpdb.git
# git fetch --depth=1 origin 2155c5a8cf8bb7f13f49c6e248fd967a74fed591
git fetch --depth=1 origin 5.8.0
git reset --hard FETCH_HEAD
cd ..
```
* Скомпилировать библиотеку с помощью `make`. Должен появиться файл `annotate_query.so`

* Нужно распространить библиотеку по всем хостам кластера в каталог с библиотеками greenplum. Возможно, придется это делать из-под sudo в зависимости от прав на целевой каталог
```bash
gpscp -f all_hosts.hosts annotate_query.so =:$(pg_config --pkglibdir)
# gpscp -h mdw -h sdw1 -h sdw2 annotate_query.so =:$(pg_config --pkglibdir)
```

* После необходимо создать функцию-обертку на sql над функциями в библиотеке
```bash
psql -f annotate_query.sql your_database
```

* Проверим синтаксическое дерево разбора тестового запроса
```sql
select annotate_query($$
  select 1
$$);
```

### Основано на проекте

https://github.com/pganalyze/queryparser
