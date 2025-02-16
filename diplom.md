#  Дипломная работа по профессии "Системный администратор"-Исаков Михаил

Содержание
==========
  # Выполнение дипломной работы  
### Terraform
- [Инфраструктура](#terra)
    - [Сеть](#net)
    - [Группы безопасности](#group)
    - [Load Balancer](#balancer)
    - [Резервное копирование](#snapshot)
### Ansible
- [Установка и настройка ansible](#cfg)
- [NGINX](#web)
- [Мониторинг](#zabbix)
- Логи
    -  [Установка Elasticsearch](#elastic)
    -  [Установка Kibana](#kibana)
    -  [Установка Filebeat](#filebeat)
 
 **ссылки на ресурсы**  
[Сайт](http://158.160.135.0/)  
[Kibana](http://158.160.75.172:5601/)  
[Zabbix](http://158.160.67.178/zabbix)  
---------

## Задача
Ключевая задача — разработать отказоустойчивую инфраструктуру для сайта, включающую мониторинг, сбор логов и резервное копирование основных данных. Инфраструктура должна размещаться в [Yandex Cloud](https://cloud.yandex.com/) и отвечать минимальным стандартам безопасности: запрещается выкладывать токен от облака в git. Используйте [инструкцию](https://cloud.yandex.ru/docs/tutorials/infrastructure-management/terraform-quickstart#get-credentials).

**Перед началом работы над дипломным заданием изучите [Инструкция по экономии облачных ресурсов](https://github.com/netology-code/devops-materials/blob/master/cloudwork.MD).**

## Инфраструктура
Для развёртки инфраструктуры используйте Terraform и Ansible.  

Не используйте для ansible inventory ip-адреса! Вместо этого используйте fqdn имена виртуальных машин в зоне ".ru-central1.internal". Пример: example.ru-central1.internal  - для этого достаточно при создании ВМ указать name=example, hostname=examle !! 

Важно: используйте по-возможности **минимальные конфигурации ВМ**:2 ядра 20% Intel ice lake, 2-4Гб памяти, 10hdd, прерываемая. 

**Так как прерываемая ВМ проработает не больше 24ч, перед сдачей работы на проверку дипломному руководителю сделайте ваши ВМ постоянно работающими.**

Ознакомьтесь со всеми пунктами из этой секции, не беритесь сразу выполнять задание, не дочитав до конца. Пункты взаимосвязаны и могут влиять друг на друга.

### Сайт
Создайте две ВМ в разных зонах, установите на них сервер nginx, если его там нет. ОС и содержимое ВМ должно быть идентичным, это будут наши веб-сервера.

Используйте набор статичных файлов для сайта. Можно переиспользовать сайт из домашнего задания.

Виртуальные машины не должны обладать внешним Ip-адресом, те находится во внутренней сети. Доступ к ВМ по ssh через бастион-сервер. Доступ к web-порту ВМ через балансировщик yandex cloud.

Настройка балансировщика:

1. Создайте [Target Group](https://cloud.yandex.com/docs/application-load-balancer/concepts/target-group), включите в неё две созданных ВМ.

2. Создайте [Backend Group](https://cloud.yandex.com/docs/application-load-balancer/concepts/backend-group), настройте backends на target group, ранее созданную. Настройте healthcheck на корень (/) и порт 80, протокол HTTP.

3. Создайте [HTTP router](https://cloud.yandex.com/docs/application-load-balancer/concepts/http-router). Путь укажите — /, backend group — созданную ранее.

4. Создайте [Application load balancer](https://cloud.yandex.com/en/docs/application-load-balancer/) для распределения трафика на веб-сервера, созданные ранее. Укажите HTTP router, созданный ранее, задайте listener тип auto, порт 80.

Протестируйте сайт
`curl -v <публичный IP балансера>:80` 

### Мониторинг
Создайте ВМ, разверните на ней Zabbix. На каждую ВМ установите Zabbix Agent, настройте агенты на отправление метрик в Zabbix. 

Настройте дешборды с отображением метрик, минимальный набор — по принципу USE (Utilization, Saturation, Errors) для CPU, RAM, диски, сеть, http запросов к веб-серверам. Добавьте необходимые tresholds на соответствующие графики.

### Логи
Cоздайте ВМ, разверните на ней Elasticsearch. Установите filebeat в ВМ к веб-серверам, настройте на отправку access.log, error.log nginx в Elasticsearch.

Создайте ВМ, разверните на ней Kibana, сконфигурируйте соединение с Elasticsearch.

### Сеть
Разверните один VPC. Сервера web, Elasticsearch поместите в приватные подсети. Сервера Zabbix, Kibana, application load balancer определите в публичную подсеть.

Настройте [Security Groups](https://cloud.yandex.com/docs/vpc/concepts/security-groups) соответствующих сервисов на входящий трафик только к нужным портам.

Настройте ВМ с публичным адресом, в которой будет открыт только один порт — ssh.  Эта вм будет реализовывать концепцию  [bastion host]( https://cloud.yandex.ru/docs/tutorials/routing/bastion) . Синоним "bastion host" - "Jump host". Подключение  ansible к серверам web и Elasticsearch через данный bastion host можно сделать с помощью  [ProxyCommand](https://docs.ansible.com/ansible/latest/network/user_guide/network_debug_troubleshooting.html#network-delegate-to-vs-proxycommand) . Допускается установка и запуск ansible непосредственно на bastion host.(Этот вариант легче в настройке)

Исходящий доступ в интернет для ВМ внутреннего контура через [NAT-шлюз](https://yandex.cloud/ru/docs/vpc/operations/create-nat-gateway).

### Резервное копирование
Создайте snapshot дисков всех ВМ. Ограничьте время жизни snaphot в неделю. Сами snaphot настройте на ежедневное копирование.

---

# Выполнение дипломной работы  

## Terraform

### <a id="terra">Инфраструктура</a>  

Поднимаем инфраструктуру в Yandex Cloud используя **terraform**  

Проверяем правильность конфигурации командой
`terraform plan` сверяем что все верно и запускаем процесс поднятия инфраструктуры  командой `terraform apply`  
в конце выполнения получаем данные output получение которые мы прописывали в файле **outputs.tf**  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/1.PNG)

после завершения работы **terraform** проверяем в web консоли YC созданную инфраструктуру.  Сервера WEB-1 и WEB-2 созданы в разных зонах.

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/2.PNG)

### <a id="net">Сеть</a>  

 **VPC и subnet** 

создаем 1 VPC с публичными и внутренними подсетями и таблицей маршрутизации для доступа к интернету ВМ находящихся внутри сети за Бастионом который будет выступать в роли интернет-шлюза.  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/3.PNG) 

### <a id="group">Группы безопасности</a>  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/4.PNG)

**Группы безопасности_LoadBalancer**  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/5.PNG)

**Группы безопасности_internal** с разрешением любого трафика между ВМ кому присвоена данная группы безопасности

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/6.PNG)

**Группы безопасности_bastion** c с открытием только 22 порта для работы SSH  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/7.PNG)

**Группы безопасности_kibana** c открытым портом 5601 для доступа c интернета к Fronted Kibana  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/8.PNG)

**Группы безопасности_zabbix** с открытым портом 80 и 10051 для доступа с интернета к Fronted Zabbix и работы Zabbix agent  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/9.PNG)

### <a id="balancer">Load Balancer</a>  

**Создаем Target Group**  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/10.PNG)

**Создаем Backend Group**  

Настраиваем backends на target group, ранее созданную, healthcheck на корень (/) и порт 80, протокол HTTP.  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/11.PNG)
![z](https://github.com/632456/hw-02/blob/main/diplom-scr/12.PNG)

**Создаем HTTP router**  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/13.PNG)

**Создаем Application load balancer**  

для распределения трафика на веб-сервера созданные ранее. Указываем HTTP router, созданный ранее, задаем listener тип auto, порт 80.  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/14.PNG)

**карта балансировки**  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/15.PNG)

### <a id="snapshot">Резервное копирование</a>  

**snapshot**  

создаем в terraform блок с расписанием snapshots  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/16.PNG)

**проверяем на следующий день что снимки создались по расписанию**  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/17.PNG)

## Ansible  

### <a id="cfg">Установка и настройка ansible</a>  

устанавливаем ***ansible*** на локальном хосте где работали с ***terraform***  и настраиваем его на работу через ***bastion***  

**файл конфигурации**

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/18.PNG) 

**файл inventory**

создаем файл hosts.ini c использованием FQDN имен серверов вместо ip  
(т.к. DNS имя для ВМ bastion не регистрировалось глобально, то я просто использовал iр для доступа из интернета)  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/20.PNG)

**проверяем доступность ВМ используя модуль ping**  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/19.PNG)

### <a id="web">Установка NGINX и загрузка сайта</a>  

**ставим nginx**  
запускаем playbook установки Nginx с созданием web страницы  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/21.PNG)

**проверяем доступность сайта в браузере по публичному ip адресу Load Balancer**  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/22.PNG)

**делаем запрос curl -v 158.160.135.0:80**  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/23.PNG)

**проверяем работу Load Balancer в web консоли YC**  

видим что меняется ip backend, значит балансировщик работает.  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/24.PNG)

### <a id="zabbix">Мониторинг</a>  

**установка Zabbix сервера**  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/25.PNG)

**проверяем доступность frontend zabbix сервера**  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/26.PNG)

**установка Zabbix агентов на web сервера**  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/27.PNG)

**добавляем хосты используя FQDN имена в zabbix сервер и настраиваем дашборды**  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/30.PNG)

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/31.PNG)

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/32.PNG)

## Установка стека ELK для сбора логов  

### <a id="elastic">Установка Elasticsearch</a>  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/33.PNG)


### <a id="kibana">Установка Kibana</a>  

**устанавливаем Kibana**  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/35.PNG)

### <a id="filebeat">Установка Filebeat</a>  

**Устанавливаем Filebeat на web сервера**  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/39.PNG)

**Проверяем в Kibana что Filebeat доставляет логи в Elasticsearch**  

![z](https://github.com/632456/hw-02/blob/main/diplom-scr/44.PNG)


---
