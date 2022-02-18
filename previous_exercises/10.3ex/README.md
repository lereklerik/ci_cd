# Домашнее задание к занятию "10.03. Grafana"

## Задание 1. Используя директорию [help](./help) внутри данного домашнего задания - запустите связку prometheus-grafana. 
## Зайдите в веб-интерфейс графана, используя авторизационные данные, указанные в манифесте docker-compose.
## Подключите поднятый вами prometheus как источник данных.

![01](pictures/01.png)

![02](pictures/02.png)

## Задание 2. Изучите самостоятельно ресурсы: - [promql-for-humans](https://timber.io/blog/promql-for-humans/#cpu-usage-by-instance) - [understanding prometheus cpu metrics](https://www.robustperception.io/understanding-machine-cpu-usage)
## Создайте Dashboard и в ней создайте следующие Panels:

### Утилизация CPU для nodeexporter (в процентах, 100-idle)

![05](pictures/05.png)

* Для каждого CPU свой номер:

```shell
rate(node_cpu_seconds_total{job="node",cpu="0",mode="idle"}[1m])*100
```
### CPULA 1/5/15

![04](pictures/04.png)

* Для каждого CPU свой номер:

```shell
100 - ((rate(node_cpu_seconds_total{cpu="0",job="node",mode="idle"}[1m])) * 100)
```
### Количество свободной оперативной памяти

![06](pictures/06.png)

```shell
100 * (1 - ((avg_over_time(node_memory_MemFree_bytes[5m]) + avg_over_time(node_memory_Cached_bytes[5m]) + avg_over_time(node_memory_Buffers_bytes[5m])) / avg_over_time(node_memory_MemTotal_bytes[5m])))
```

### Количество места на файловой системе

![07](pictures/07.png)

```shell
(node_filesystem_free_bytes{fstype=~"ext4|xfs"} / node_filesystem_size_bytes{fstype=~"ext4|xfs"})*100
```

![03](pictures/03.png)

## Задание 3. Создайте для каждой Dashboard подходящее правило alert

![09](pictures/09.png)

![10](pictures/10.png)

![11](pictures/11.png)

![12](pictures/12.png)

![13](pictures/13.png)

![14](pictures/14.png)

![15](pictures/15.png)

![16](pictures/16.png)

## Задание 4. Сохраните ваш Dashboard.

* [Файлик](dashboard.json) в репозитории
