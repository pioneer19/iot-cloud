### Работа с контейнерами докер для проекта IoT
* [influxdb](#%D1%81%D0%BE%D0%B7%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5-%D0%BF%D0%B0%D0%BA%D0%B5%D1%82%D0%B0-influxdb)
* [grafana](#%D1%81%D0%BE%D0%B7%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5-%D0%BF%D0%B0%D0%BA%D0%B5%D1%82%D0%B0-grafana)
* [mqtt-client](#%D1%81%D0%BE%D0%B7%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5-%D0%BF%D0%B0%D0%BA%D0%B5%D1%82%D0%B0-mqtt-client)
#### Создание пакета influxdb
Пакет создается на основе стандартного пакета influx. `Dockerfile` для него находится 
в `pod_influxdb`. Все делается на машине, где будет работать приложение.
Возможно, команды нужно будет поправить в зависимости от поменявшегося окружения
(на момент написания я давал именно эти команды).
```
# создание пакета докер в локальном репозитории, делается в папке с Dockerfile
docker build -t iot_influxdb:1.8 .
# создание тома для постоянного хранения данных
docker volume create iot_influxdb

# копирование существующих данных в том influx
cp -a /home/vbox-ubuntu-data/influxdb .
# run any container (ubuntu, influxdb, ... any) with mounted volume and host folder
docker run --rm -it --mount type=bind,source="$(pwd)/influxdb",target=/host_data -v iot_influxdb:/var/lib/influxdb influxdb:1.8 /bin/bash
# copy data in started docker container from host folder to volume
cp -a /host_data/* /var/lib/influxdb/
chown -R 999:999   /var/lib/influxdb/*
# данные скопированы, контейнер можно остановить
exit
```
Запуск контейнера в докере я делаю из `systemd`. Файл `docker-influxdb.service`.
```
# примерная команда, которую выполнит systemd для запуска
docker run --rm -p 8086:8086 -v iot_influxdb:/var/lib/influxdb iot_influxdb:1.8
```
Докер из контейнера пробросит порт 8086 (сеть в докере своя, но к influx нужно
как-то подключиться, поэтому пробрасывается порт). В нашем случае этот порт
будет доступен по IP машины.

#### Создание пакета grafana
Образ контейнера grafana создавать не нужно, он используется стандартный с hub.docker.com.
Нужно лишь скопировать файл запуска для systemd `pod_grafana/docker-grafana.service`.
Также нужно будет скопировать файл запуска плагина графаны image render
`pod_grafana_image_renderer/docker-grafana-image-render.service`.

```
docker volume create iot_grafana

# copy original data in some folder
scp -r root@old-grafana-host.lan:/var/lib/grafana .
# run any container (ubuntu, influxdb, ... any) with mounted volume and host folder
docker run --rm -it --mount type=bind,source="$(pwd)/grafana",target=/host_data -v iot_grafana:/var/lib/grafana influxdb:1.8 /bin/bash
# copy data in started docker container from host folder to volume
cp -a /host_data/* /var/lib/grafana/ 
chown -R 472:0   /var/lib/grafana
# exit from docker
exit 
```
Контейнер графаны и плагина image render запускается из systemd.

#### Создание пакета mqtt-client
MQTT-client самый простой контейнер, в него не нужно пробрасывать порты
и ему не нужно хранилище данных - он все получает по сети.

Репозиторий для пакета mqtt-client отдельный (https://github.com/BalalaykaJazz/influx).
Для создания образа docker нужно будет вручную откуда-то взять файл с паролями
`settings.json` и скопировать его поверх того, что в репозитории.
```
# создание пакета докер в локальном репозитории, делается в папке с Dockerfile
docker build -t iot_mqtt_client .
```
Далее если systemd файл для запуска (docker-mqtt_client.service) еще не
установлен в систему его нужно скопировать из того же репозитория.
Запускается контейнер из `systemd`.
```
# только один раз нужно разрешить запуск контейнера автоматом при загрузке
# sudo systemctl enable docker-mqtt_client.service
# запуск контейнера вручную
sudo systemctl start docker-mqtt_client.service
```
