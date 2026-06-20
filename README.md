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


---
