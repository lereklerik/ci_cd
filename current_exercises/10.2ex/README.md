# Домашнее задание к занятию "10.02. Системы мониторинга"

## 1. Опишите основные плюсы и минусы pull и push систем мониторинга.

--------------------------------------------------------------------------------------------------------------------------------

    * Push-модель – когда сервер мониторинга ожидает подключений от агентов для получения метрик;
    * Pull-модель – когда сервер мониторинга сам подключается к агентам мониторинга и забирает данные;

* Push-модель:
    * Плюсы:
        - Упрощение репликации данных в разные системы мониторинга или их резервные копии;
        - Более гибкая настройка отправки пакетов данных с метриками;
        - Удобна для использования в динамически создаваемых машинах (например из докер-контейнеров), так как в противном случае Система мониторинга должна будет узнавать о новых хостах для их опроса;
        - UDP является менее затратным способом передачи данных, вследствии чего может вырасти производительность сбора метрик.
    * Минусы:
        - Риск утечки данных при их передачи по UDP;
        - Если принимающая сторона будет недоступна, возможно потерять данные по метрикам.

* Pull-модель:
    * Плюсы:
        - Легче контролировать подлинность данных;
        - Контроль над метриками с единой точки, возможность конеккта по SSL к агентам;
        - Более высокий уровень контроля за источниками метрик.
    * Минусы:
        - При сетевом сбое можно потерять все данные, накопленные за время простоя системы;
        - Нужно динамически собирать статистику о наличии машин при использовании динамического окружения, т.е. нужен дополнительный оркестратор.

--------------------------------------------------------------------------------------------------------------------------------

## 2. Какие из ниже перечисленных систем относятся к push модели, а какие к pull? А может есть гибридные?
      - Prometheus 
      - TICK
      - Zabbix
      - VictoriaMetrics
      - Nagios

* **Prometheus** - _pull_-система. Сбор метрик в Prometheus осуществляется с помощью механизма pull. Имеется также возможность сбора метрик с помощью механизма push (для этого используется специальный компонент pushgateway, который устанавливается отдельно). 
* **TICK** - _push_-система. Набор компонентов:
    * Telegraf собирает данные временного ряда из разных источников. Telegraf не ограничивается только InfluxDB как конечной точкой для передачи данных. Также Telegraf можно настроить для работы в соответствии с Pull-моделью
    * InfluxDB хранит данные временного ряда.
    * Chronograf визуализирует данные временных рядов.
    * Kapacitor отвечает за оповещения и обнаружение аномалий в данных временных рядов.
* **Zabbix** - работает в соответствии с _push_ и _pull_ моделью. Механизм дуализма pull и push моделей в Zabbix заключается в настраиваемых “Активных и Пассивных” проверках.
    * Активные проверки - указываются данные для непрерывного наблюдения, которые перенаправляются агентом серверу. 
    * Пассивные проверки - набор данных, которые собираются и хранятся на агенте, но отправляются на сервер только по запросу.
* **VictoriaMetrics** - _push_-система. Быстрая и масштабируемая СУБД для хранения и обработки данных в форме временного ряда (запись образует время и набор соответствующих этому времени значений, например, полученных через периодический опрос состояния датчиков или сбор метрик).
* **Nagios** - _pull_-система. Nagios работает в качестве демона (фонового процесса) на выделенном сервере, периодически отправляя ICMP запросы на хост мониторинга. Полученная информация обрабатывается на сервере и отображается администратору в рамках WEB – интерфейса.

--------------------------------------------------------------------------------------------------------------------------------

## 3. Склонируйте себе [репозиторий](https://github.com/influxdata/sandbox/tree/master) и запустите TICK-стэк, используя технологии docker и docker-compose.

    В виде решения на это упражнение приведите выводы команд с вашего компьютера (виртуальной машины):

        - curl http://localhost:8086/ping
        - curl http://localhost:8888
        - curl http://localhost:9092/kapacitor/v1/ping

    А также скриншот веб-интерфейса ПО chronograf (`http://localhost:8888`). 

    P.S.: если при запуске некоторые контейнеры будут падать с ошибкой - проставьте им режим `Z`, например
    `./data:/var/lib:Z`

------------------

```shell
netology@netology:~/Projects$ curl http://localhost:8086/ping -v
*   Trying 127.0.0.1:8086...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 8086 (#0)
> GET /ping HTTP/1.1
> Host: localhost:8086
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 204 No Content
< Content-Type: application/json
< Request-Id: 6a3c5beb-7c7f-11ec-8050-0242ac120002
< X-Influxdb-Build: OSS
< X-Influxdb-Version: 1.8.10
< X-Request-Id: 6a3c5beb-7c7f-11ec-8050-0242ac120002
< Date: Sun, 23 Jan 2022 19:05:22 GMT
< 
* Connection #0 to host localhost left intact

#######################################################################

netology@netology:~/Projects$ curl http://localhost:8888 -v
*   Trying 127.0.0.1:8888...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 8888 (#0)
> GET / HTTP/1.1
> Host: localhost:8888
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Accept-Ranges: bytes
< Cache-Control: public, max-age=3600
< Content-Length: 336
< Content-Security-Policy: script-src 'self'; object-src 'self'
< Content-Type: text/html; charset=utf-8
< Etag: "336820331"
< Last-Modified: Fri, 08 Oct 2021 20:33:01 GMT
< Vary: Accept-Encoding
< X-Chronograf-Version: 1.9.1
< X-Content-Type-Options: nosniff
< X-Frame-Options: SAMEORIGIN
< X-Xss-Protection: 1; mode=block
< Date: Sun, 23 Jan 2022 19:05:55 GMT
< 
* Connection #0 to host localhost left intact
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="Content-type" content="text/html; charset=utf-8">
      <title>Chronograf</title>
      <link rel="icon shortcut" href="/favicon.fa749080.ico">
      <link rel="stylesheet" href="/src.3dbae016.css">
  </head>
  <body> 
    <div id="react-root" data-basepath=""></div> 
    <script src="/src.fab22342.js"></script> 
  </body>
</html>

#######################################################################

netology@netology:~/Projects$ curl http://localhost:9092/kapacitor/v1/ping -v
*   Trying 127.0.0.1:9092...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 9092 (#0)
> GET /kapacitor/v1/ping HTTP/1.1
> Host: localhost:9092
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 204 No Content
< Content-Type: application/json; charset=utf-8
< Request-Id: 9bb4e318-7c7f-11ec-805a-000000000000
< X-Kapacitor-Version: 1.6.2
< Date: Sun, 23 Jan 2022 19:06:45 GMT
< 
* Connection #0 to host localhost left intact

```

![01](01.png)

![02](02.png)

---------------------------------------------------------------------------------------------------------------------------

## 4. Перейдите в веб-интерфейс Chronograf (`http://localhost:8888`) и откройте вкладку `Data explorer`.

    - Нажмите на кнопку `Add a query`
    - Изучите вывод интерфейса и выберите БД `telegraf.autogen`
    - В `measurments` выберите mem->host->telegraf_container_id , а в `fields` выберите used_percent. Внизу появится график утилизации оперативной памяти в контейнере telegraf.
    - Вверху вы можете увидеть запрос, аналогичный SQL-синтаксису. 
    Поэкспериментируйте с запросом, попробуйте изменить группировку и интервал наблюдений.

Для выполнения задания приведите скриншот с отображением метрик утилизации места на диске 
(disk->host->telegraf_container_id) из веб-интерфейса.

* Пришлось добавлять данные в конфигурацию:
```shell
[[inputs.disk]]
  ## By default stats will be gathered for all mount points.
  ## Set mount_points will restrict the stats to only the specified mount points.
  # mount_points = ["/"]

  ## Ignore mount points by filesystem type.
  ignore_fs = ["tmpfs", "devtmpfs", "devfs", "iso9660", "overlay", "aufs", "squashfs"]
```
![03](03.png)

------------------------------------------------------------------------------------------------------

## 5. Изучите список [telegraf inputs](https://github.com/influxdata/telegraf/tree/master/plugins/inputs). 
Добавьте в конфигурацию telegraf следующий плагин - [docker](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/docker):
```shell
[[inputs.docker]]
  endpoint = "unix:///var/run/docker.sock"
```

* Пришлось скорректировать `docker-compose.yml` в части `telegraf` следующим образом:

```yaml
  telegraf:
    # Full tag list: https://hub.docker.com/r/library/telegraf/tags/
    build:
      context: ./images/telegraf/
      dockerfile: ./${TYPE}/Dockerfile
      args:
        TELEGRAF_TAG: ${TELEGRAF_TAG}
    image: "telegraf"
    privileged: true
    environment:
      HOSTNAME: "telegraf-getting-started"
      HOST_PROC: /rootfs/proc
      HOST_SYS: /rootfs/sys
      HOST_ETC: /rootfs/etc
    # Telegraf requires network access to InfluxDB
    links:
      - influxdb
    
    restart: unless-stopped
    user: telegraf:134
    volumes:
      # Mount for telegraf configuration
      - ./telegraf/:/etc/telegraf/:ro
      # Mount for Docker API access
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /sys:/rootfs/sys:ro
      - /proc:/rootfs/proc:ro
      - /etc:/rootfs/etc:ro
      # - /sys/fs/cgroup:/sys/fs/cgroup:ro
    ports:
      - "8092:8092/udp"
      - "8094:8094"
      - "8125:8125/udp"
    depends_on:
      - influxdb
```

* Конфигурация `telegraf`:

```shell
[agent]
  interval = "5s"
  round_interval = true
  metric_batch_size = 1000
  metric_buffer_limit = 10000
  collection_jitter = "0s"
  flush_interval = "5s"
  flush_jitter = "0s"
  precision = ""
  debug = false
  quiet = false
  logfile = ""
  hostname = "$HOSTNAME"
  omit_hostname = false

[[outputs.influxdb]]
  urls = ["http://influxdb:8086"]
  database = "telegraf"
  username = ""
  password = ""
  retention_policy = ""
  write_consistency = "any"
  timeout = "5s"
 
[[inputs.cpu]]
[[inputs.system]]
[[inputs.influxdb]]
  urls = ["http://influxdb:8086/debug/vars"]
[[inputs.syslog]]
#   ## Specify an ip or hostname with port - eg., tcp://localhost:6514, tcp://10.0.0.1:6514
#   ## Protocol, address and port to host the syslog receiver.
#   ## If no host is specified, then localhost is used.
#   ## If no port is specified, 6514 is used (RFC5425#section-4.1).
  server = "tcp://localhost:6514"
[[inputs.disk]]
  ignore_fs = ["tmpfs", "devtmpfs", "devfs", "iso9660", "overlay", "aufs", "squashfs"]
[[inputs.diskio]]
[[inputs.kernel]]
[[inputs.mem]]
[[inputs.processes]]
[[inputs.swap]]

# Read metrics about docker containers
[[inputs.docker]]
  endpoint = "unix:///var/run/docker.sock"
  timeout = "5s"
  perdevice = true
  # ## Whether to report for each container total blkio and network stats or not
  total = false
  # ## docker labels to include and exclude as tags.  Globs accepted.
  # ## Note that an empty array for both will include all labels as tags
  docker_label_include = []
  docker_label_exclude = []
# [[inputs.net]]
  # interfaces = ["eth0"]
```
![04](04.png)

![05](05.png)