FROM influxdb:1.8

COPY ./influxdb.conf /etc/influxdb/influxdb.conf
USER influxdb

#docker run -it --rm --mount type=bind,source="/home/san/",target=/home/san/ san:v1 /run.sh
# docker run --rm -p 8086:8086 -v $(pwd)/influxdb:/var/lib/influxdb influxdb:1.8
