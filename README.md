# Домашнее задание к занятию "`Резервное копирование баз данных`" - `Никулин Михаил Сергеевич`



---

### Задание 1

`1.1 Необходимо восстанавливать данные в полном объеме за предыдущий день.`

`Можно использовать Дифференциальное резервное копирование. Оно быстрее, чем полное, но медленнее инкрементного. В нашем случае
этот вариант выгоднее, т.к. копии не нужно делать часто, а восстановление у дифференциального быстрее за счет того, что использует
только 2 файла: полную резервную копию и последнюю дифференциальную копию, которая содержит в себе все изменения с момента последнего 
полного копирования. Если деятельность будут останавливать для профилактики раз в день, тогда можно применить холодное 
резервное копирование в это время, в противном случае - горячее`

`1.2 Необходимо восстанавливать данные за час до предполагаемой поломки.`

`Инкрементное резервное копирование. Согласно плану требуется частое и быстрое создание резервных копий без возможности остановки деятельности.
Это приводит к выбору именно Инкрементного типа резервного копирования, т.к. оно самое быстрое из всех, за счет того, что при 
каждом копировании происходит сохранение только изменений с последнего резервирования. При больших размерах базы данных 
это критично.`

`1.3 Возможен ли кейс, когда при поломке базы происходило моментальное переключение на работающую/починеную БД?`

`Репликация по принципу master-slave`


---

### Задание 2

`Используйте специальный формат дампа pg_dump. Если при сборке PostgreSQL была подключена библиотека zlib, дамп в 
специальном формате будет записываться в файл в сжатом виде. В таком формате размер файла дампа будет близок к размеру, 
полученному с применением gzip, но он лучше тем, что позволяет восстанавливать таблицы выборочно. Следующая команда 
выгружает базу данных в специальном формате:`

```
pg_dump -Fc имя_базы > имя_файла
```

`Дамп в специальном формате не является скриптом для psql и должен восстанавливаться с помощью команды pg_restore, например:`

```
pg_restore -d имя_базы имя_файла
```
---

`Скрипт для автоматизации процесса резервного копирования postgresql_dump.sh`

```
#!/bin/sh
PATH=/etc:/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin

PGPASSWORD=password
export PGPASSWORD
pathB=/backup
dbUser=dbuser
database=db

find $pathB \( -name "*-1[^5].*" -o -name "*-[023]?.*" \) -ctime +61 -delete
pg_dump -U $dbUser $database | gzip > $pathB/pgsql_$(date "+%Y-%m-%d").sql.gz

unset PGPASSWORD
```

`Данный скрипт сначала удалит все резервные копии, старше 61 дня, но оставит от 15-о числа как длительный архив. После 
при помощи утилиты pg_dump будет выполнено подключение и резервирование базы db. Пароль экспортируется в системную 
переменную на момент выполнения задачи.`

`Для запуска резервного копирования по расписанию, сохраняем скрипт в файл, например, /scripts/postgresql_dump.sh и 
создаем задание в планировщике:`

```
crontab -e
5 0 * * * /scripts/postgresql_dump.sh
```
`Скрипт будет запускаться каждый день в 03.00`

---

### Задание 3

`3.1`

`Согласно официальной документации для создания инкрементной копии, используется команда:`

```
--incremental-base=history:last_backup
```

`При этом, если в аргументах заменить last_backup на last_full_backup, тогда получится дифференцированное резервное копирование`

`Также популярна утилита XtraBackup`

```
xtrabackup --backup --target-dir=/root/backupdb/inc1 --incremental-basedir=/root/backupdb/full
```
---
`3.1*`

* Репликация позволяет переключиться на резервную ВМ и продолжить работу через 5-20 минут.
* Сосредоточена на аварийном восстановлении рабочей виртуальной машины и сервиса на ней.
* Используется для критически важных сервисов, которые всегда должны быть в рабочем состоянии.
* Имеет более эффективные показатели RTO и RPO, что обусловлено частой синхронизацией, а также отсутствием сжатия и шифрования.