Sat, 20 Feb 2021 19:35:37 +0300
* вытащил из контейрена grafana плагин image render в отдельный контейнер
  (теперь оба контейнера используют стандартные image с hub.docker.com)

Wed, 10 Feb 2021 15:25:07 +0300
* Поставил дополнительную сетевую карту, повесил на нее внешинй IP
* Закрыл снаружи ненужные порты (поменял привязку (bind) сервисов к IP)
* Перенес web и DNS на внешний IP.

Fri, 22 Jan 2021 14:25:06 +0300
* добавил блокировку сканирования порта ssh

Mon, 11 Jan 2021 13:18:15 +0300
* создал докер контейнер графаны на основе ubuntu и обновил описание

Wed Dec 30 21:00:00 MSK 2020
* переразбил диск, добавил ssd в качестве кеша для фаловой системы на `/home`
  (по сути, все данные помещаются на ssd и практически всегда работа будет
  со скоростью ssd, при этом размер диска - размер HDD)
* перенес influx и grafana в контейнеры docker на хостовой машине
  (теперь виртуализация их никак не тормозит).

Описание работы с [docker](./about_docker.md) и конкретные [команды для работы
с контейнерами](./IoT.md).

Wed, 16 Dec 2020 17:34:47 +0300
* настроил автозапуск виртуалок на хосте (машине).
* остановил запуск vboxWebService, который также не смог
  создать Pid файл и в результате не работает. Он не нужен.
* удалил mosquitto из виртуалки.

Wed, 15 Dec 2020
* обновил пакеты внутри виртуалки и на машине
* исправил запуск mqtt клиента, который перестал работать
* исправил запуск mosquitto, который не смог создать Pid file

Mon, 14 Dec 2020 18:33:11 +0300

* вынес файлы influxbd (/var/lib/influxdb) на отдельную шару vboxfs
* перезапустил вартуалку на 4-х CPU (работала на одном)

Thu, 10 Dec 2020 18:31:36 +0300
* сделал задержку в 200ms на запись журнала в influxdb 
  (был sync после каждой записи в файл журнала)   
  (wal-fsync-delay = "200ms" в конфиге influx)
* рестартанул influx (reload он не умеет)
* рестартанул скрипт на питоне (он упал при рестарте influx)
---
сделал автозапуск скриптов MQTT внутри виртуалки
* создал unit file для systemd /lib/systemd/system/mqtt-client.service
* systemctl enable mqtt-client
* systemctl start mqtt-client

influx теперь пишет данные в журнал пачками (ждет 200ms и
записывает оптом). Нагрузка на диск уменьшилась.
Т.е. при внезапном падении машины данные за <=200 ms потеряются
(думаю вам это не важно).