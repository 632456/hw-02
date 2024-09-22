# Домашнее задание к занятию "`Disaster recovery и Keepalived`" - `Исаков Михаил`

### Задание 1
- Дана [схема](1/hsrp_advanced.pkt) для Cisco Packet Tracer, рассматриваемая в лекции.
- На данной схеме уже настроено отслеживание интерфейсов маршрутизаторов Gi0/1 (для нулевой группы)
- Необходимо аналогично настроить отслеживание состояния интерфейсов Gi0/0 (для первой группы).
- Для проверки корректности настройки, разорвите один из кабелей между одним из маршрутизаторов и Switch0 и запустите ping между PC0 и Server0.
- На проверку отправьте получившуюся схему в формате pkt и скриншот, где виден процесс настройки маршрутизатора.

------


### Решение 

![z](https://github.com/632456/hw-02/blob/main/Disaster/1_1.jpg)
![z](https://github.com/632456/hw-02/blob/main/Disaster/1_2.jpg)

#### Схема pkt: [shema.pkt](https://github.com/632456/hw-02/blob/main/Disaster/shema.pkt)



### Задание 2
- Запустите две виртуальные машины Linux, установите и настройте сервис Keepalived как в лекции, используя пример конфигурационного [файла](1/keepalived-simple.conf).
- Настройте любой веб-сервер (например, nginx или simple python server) на двух виртуальных машинах
- Напишите Bash-скрипт, который будет проверять доступность порта данного веб-сервера и существование файла index.html в root-директории данного веб-сервера.
- Настройте Keepalived так, чтобы он запускал данный скрипт каждые 3 секунды и переносил виртуальный IP на другой сервер, если bash-скрипт завершался с кодом, отличным от нуля (то есть порт веб-сервера был недоступен или отсутствовал index.html). Используйте для этого секцию vrrp_script
- На проверку отправьте получившейся bash-скрипт и конфигурационный файл keepalived, а также скриншот с демонстрацией переезда плавающего ip на другой сервер в случае недоступности порта или файла index.html


------

### Решение 

![z](https://github.com/632456/hw-02/blob/main/Disaster/2_1.png)
![z](https://github.com/632456/hw-02/blob/main/Disaster/2_2.png)
![z](https://github.com/632456/hw-02/blob/main/Disaster/2_3.png)
![z](https://github.com/632456/hw-02/blob/main/Disaster/2_4.png)
![z](https://github.com/632456/hw-02/blob/main/Disaster/2_5.png)

#### bash-скрипт: [check.sh](https://github.com/632456/hw-02/blob/main/Disaster/check.sh)
#### keepalived-backup.conf: [keepalived-backup.conf](https://github.com/632456/hw-02/blob/main/Disaster/keepalived-backup.conf)
#### keepalived-master.conf: [keepalived-master.conf](https://github.com/632456/hw-02/blob/main/Disaster/keepalived-master.conf)




