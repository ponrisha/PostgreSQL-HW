# Домашнее задание №1
1. Развернута ВМ *Ubuntu Server 22.04*
2. На клиенте (*windows 10*) сгенерирован ssh-ключ:
<br>*C:\Users\Iris>ssh-keygen*
<br>*ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC6u8O7B9CQANv8wtNNr5GBvqdmeYbJVCp2m8XE15hoAwzWBVXxUQxBhl0ZB9VMUcxEclTVi3IxtL0CEFDiEHHTDKRPideDxTrvVudd5ZVfc/pq36OBEE7svdsXhM2l5tgmhXtMxE0W3RvI0yqkptKaP3dfOD7O5pbaJicamRBCEebJ07eMj0WmNKwDt4sqz9wS8sVzzJCxM2+RNNVRkTI52+TlSgr/ZsZpFxBiiBpoFy+vFrhKXwMrVSv5QuoleuyAnYQN5Kv54vj21e5QXsNBbfjP3KEg0yFN5Dn6+HGBj3Kwm1rUshKblrR7oWpgbrqeiLnuM+7pe9dIiTVbZjiAXRy/oopyN/lD1XEW2trTzYdmS0UBiYapCc8I2iSlbabiRoAY0Edez/9Z2WwJcv9ktB2fnFefEUD/Kj2MTilwY4WSrdL1IwmUTmGQNfKNirOIsQKbRxAWDhss5+zj18zzWnePU54gkhXaSQPmz1SOcJz2sGoWn/Fxyytypg/bzbs= iris@IrisPC*
3. В домашнем каталоге пользователя ВМ создан файл «authorized_keys»:
<br>*iris@ubuntu2204:~$ mkdir .ssh && touch .ssh/authorized_keys*
4. Сгенерированный публичный ключ загружен на ВМ:
<br>*C:\Users\Iris>type .ssh\id_rsa.pub | ssh iris@10.100.110.124 "cat >> .ssh/authorized_keys"*
5. Ssh-сессия установлена – далее «первая сессия».
<br>*C:\Users\Iris>ssh iris@ubuntu2204*
6. Установлена PostgreSQL (psql (PostgreSQL) 15.2 (Ubuntu 15.2-1.pgdg22.04+1)):
<br>*sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql-15*
7. Ssh-сессия установлена – далее «вторая сессия».
<br>*sC:\Users\Iris>ssh iris@ubuntu2204*
8. Psql запущен из-под пользователя «postgres» в обоих сессиях:
<br>*ssudo -u postgres psql*
9. Отключен auto commit
    ```
    \set AUTOCOMMIT OFF
    ```
10. В первой сессии создана новая таблица, которая наполнена тестовыми данными:
    ```
    create table persons(id serial, first_name text, second_name text);
    insert into persons(first_name, second_name) values('ivan', 'ivanov');
    insert into persons(first_name, second_name) values('petr', 'petrov');
    commit;
    ```
11. Посмотреть текущий уровень изоляции:
    ```
    show transaction isolation level
    ```
> Текущий уровень изоляции: read committed
12. Не изменяя уровень изоляции, в обоих сессиях выполнено:

- в первой сессии добавлена новая запись:
    ```
    insert into persons(first_name, second_name) values('sergey', 'sergeev');
    ```
- получено 3 строки: 
    ```
    select * from persons;
    ```
| id | first_name | second_name|
|----|------------|------------|
|  1 | ivan       | ivanov |
|  2 | petr       | petrov |
|  3 | sergey     | sergeev |
|*(3 rows)*|

- во второй сессии получено только 2 строки:
    ```
    select * from persons;
    ```
| id | first_name | second_name|
|----|------------|------------|
|  1 | ivan       | ivanov|
|  2 | petr       | petrov|
|*(2 rows)*|
> Во второй сессии не будет видно новой записи, т.к. транзакция первой сессии не подтверждена (отсутствует commit)
13. Первая транзакция завершена – выполнен *commit*.
14. Во второй сессии теперь получено 3 строки:
    ```
    select * from persons;
    ```
 | id | first_name | second_name|
|----|------------|------------|
|  1 | ivan       | ivanov|
|  2 | petr       | petrov|
|  3 | sergey     | sergeev|
|*(3 rows)*|
> Новая запись отображается во второй сессии, т.к. транзакция из первой сессии была подтверждена (выполнен commit)
15. Для новых транзакций установлен уровень изоляции «repeatable read»: 
    ```
    set transaction isolation level repeatable read;
    ````
> Текущий уровень изоляции: repeatable read.

- в первой сессии добавлена новая запись:
    ```
    insert into persons(first_name, second_name) values('sveta', 'svetova');
    ```
- получено 4 строки:
    ```
    select * from persons;
    ```
| id | first_name | second_name|
|----|------------|------------|
|  1 | ivan       | ivanov|
|  2 | petr       | petrov|
|  3 | sergey     | sergeev|
|  4 | sveta      | svetova|
|*(4 rows)*|

- во второй сессии получено 3 строки:
    ```
    select * from persons;
    ```
| id | first_name | second_name|
|----|------------|------------|
|  1 | ivan       | ivanov|
|  2 | petr       | petrov|
|  3 | sergey     | sergeev|
|*(3 rows)*|
> Во второй сессии не будет видно новой записи, т.к. транзакция первой сессии не подтверждена (отсутствует *commit*)
16. Первая транзакция завершена – выполнен *commit*.
17. Во второй сессии снова получено 3 строки:
    ```
    select * from persons;
    ```
| id | first_name | second_name|
|----|------------|------------|
|  1 | ivan       | ivanov|
|  2 | petr       | petrov|
|  3 | sergey     | sergeev|
|*(3 rows)*|
> Во второй сессии не будет видно новой записи, т.к. на уровне «repeatable read» транзакция во второй сессии не видит изменений, внесённых и/или зафиксированных другими транзакциями, после запуска текущей транзакции.
18. Вторая транзакция завершена – выполнен *commit*.
19. Во второй сессии получено 4 строки:
    ```
    select * from persons;
    ```
| id | first_name | second_name|
|----|------------|------------|
|  1 | ivan       | ivanov|
|  2 | petr       | petrov|
|  3 | sergey     | sergeev|
|  4 | sveta      | svetova|
|*(4 rows)*|
> Новая запись отображается во второй сессии, т.к. транзакции из первой и второй сессий были подтверждены (выполнен commit).

## Вывод
> В транзакции, работающей на уровне «read committed», запрос SELECT видит только те данные, которые были зафиксированы до начала запроса; он никогда не увидит незафиксированных данных или изменений, внесённых в процессе выполнения запроса параллельными транзакциями. По сути запрос SELECT видит снимок базы данных в момент начала выполнения запроса. Однако SELECT видит результаты изменений, внесённых ранее в этой же транзакции, даже если они ещё не зафиксированы. Также стоит отметить, что два последовательных оператора SELECT могут видеть разные данные даже в рамках одной транзакции, если какие-то другие транзакции зафиксируют изменения после запуска первого SELECT, но до запуска второго.
> <br>
> <br>Уровень «repeatable read» отличается от «read committed» тем, что запрос в транзакции данного уровня видит снимок данных на момент начала первого оператора в транзакции (не считая команд управления транзакциями), а не начала текущего оператора. Таким образом, последовательные команды SELECT в одной транзакции видят одни и те же данные; они не видят изменений, внесённых и зафиксированных другими транзакциями после начала их текущей транзакции.
