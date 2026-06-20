# Домашнее задание к занятию «Репликация и масштабирование. Часть 1» - `Смирнов Максим`

### Задание 1

На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия.




### Задание 2

`Ссылка на .cfg`

https://github.com/turboturtle-90/Homework_clustering_and_load_balancing/blob/593b469a59cef8dcb0deedeb24426b0453281f6e/haproxy.cfg-assignment2 

`Балансировка на 7 уровне по roubdrobin с весовыми множителями и обработкой только при адресации к домену example.com`
![2-weighted-level7.jpg](https://github.com/turboturtle-90/Homework_clustering_and_load_balancing/blob/593b469a59cef8dcb0deedeb24426b0453281f6e/2-weighted-level7.jpg)


`Текст команд для конфигурирования master-slave системы на postgres приведен ниже :`

sql команды (выполняеются после вызова postgres командой строки sudo -i -u postgres psql )

```                                                         
CREATE DATABASE test_master;

CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD '123456';
SHOW data_directory;
```

Запрос пути где живет файл конфигурации
```                                                         
sudo -i -u postgres psql -c "SHOW data_directory;"
```

В файл кофигурации дописать если еще не дописаны права на подключение пользователя replication локально
```                                                         
host    replication     replicator      127.0.0.1/32            scram-sha-256
```

В настройках master активируем репликацию
```                                                         
sudo nano /etc/postgresql/16/main/postgresql.conf
wal_level = replica
max_wal_senders = 10
hot_standby = on
```


Перезагрузка сервера
```                                                         
sudo systemctl restart postgresql
```

Создаем новую пустую папку, где будут жить данные нашей реплики (создавать в директории полученной sql-командой SHOW data_directory)
```                                                         
sudo mkdir -p /var/lib/postgresql/16/replica
sudo chown postgres:postgres /var/lib/postgresql/16/replica
sudo chmod 700 /var/lib/postgresql/16/replica
```

Копируем файл конфигурации из master папки в slave папку
``` 
sudo cp /etc/postgresql/16/main/postgresql.conf /var/lib/postgresql/16/replica/postgresql.conf
sudo chown postgres:postgres /var/lib/postgresql/16/replica/postgresql.conf
```

Редактируем файл - меняем порт, т.к. мастер и слейв слушают разные порты, также меняем настройки чтобы избежать конфликтов
```
sudo nano /var/lib/postgresql/16/replica/postgresql.conf
port = 5433
data_directory = '/var/lib/postgresql/16/replica'
sudo nano /var/lib/postgresql/16/replica/postgresql.conf
ssl = off
#external_pid_file = '/var/run/postgresql/16-main.pid'
```
Добавляем папку conf.d
sudo mkdir -p /var/lib/postgresql/16/replica/conf.d
sudo chown postgres:postgres /var/lib/postgresql/16/replica/conf.d

Запускаем. Если что то не так читаем логи 
```
sudo -i -u postgres /usr/lib/postgresql/16/bin/pg_ctl -D /var/lib/postgresql/16/replica -l /var/lib/postgresql/16/replica/logfile start
sudo tail -n 20 /var/lib/postgresql/16/replica/logfile
```



---
