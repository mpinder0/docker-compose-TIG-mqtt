# Overview
This project forms the server (broker) and storage components of a data historian system for distributed sensors.
Various devices with sensors (e.g ESP32) can publish to the MQTT server, all values then persisted in Influx DB and Grafana used to visualise and monitor data received.

 - MQTT broker (in Docker)
 - TIG stack (all in Docker)
 - Running on a raspberry pi, headless setup

## Usage
Once set up, use an MQTT client (I tested using [MQTT fx](https://mqttfx.jensd.de/)) to send values to your server, port 1883.
Telegraf will then store the received values in influx db in the "mqtt_consumer" table, tagged with the topic path.
Grafana can then be used to create dashboards to visualise the data later.

# Setup
## Raspberry Pi

Creating SD card:
- Downloaded Raspberry Pi OS latest [here](https://www.raspberrypi.org/downloads/raspberry-pi-os/)
- On Ubuntu used "Startup Disk Creator" to write img to micro SD card
- For headless, in boot partition create:
    - empty "SSH" file - to enable SSH
    - wpa_supplicant.conf [more details here](https://www.raspberrypi.org/documentation/configuration/wireless/headless.md)

Connect via SSH then:
- change password with "passwd" command
- for VNC if required
    - [apt install realvnc-vnc-server](https://www.raspberrypi.org/documentation/remote-access/vnc/)
    - raspi_config tool Resolution > Highest (was required for me before VNC would start working when headless)
- apt install docker docker-compose


## Docker compose

- [docker compose file](docker-compose.yml) including the following images
    - mqtt (eclipse-mosquitto)
    - influxdb (influxdb)
    - grafana (grafana/grafana)
    - telegraf (telegraf)
- create folders on the server for docker images to use:
    - sudo mkdir -p /srv/docker/influxdb/data
    - sudo mkdir -p /srv/docker/grafana/data
    - sudo chown 472:472 /srv/docker/grafana/data
- useful docker compose commands - `sudo docker-compose`
    - `up` (-d for as daemon)
    - `down`
    - `ps` for container status'

### MQTT & Influx DB
Nothing to configure

### Telegraf
See [telegraf.conf](telegraf.conf) with MQTT consumer as input, Influx DB as output:
- connects to MQTT broker and subscribes to topics of interest (e.g. "home/#", # is wildcard)
- parse values as float
- target database name configured (e.g. "telegraf")

### Grafana
- In browser go to "localhost:3000"
- default user/password is admin/admin - you'll be prompted to change it on first login
- add Influx DB as data source (http://influxdb:8086), database "telegraf"
- create a dashboard to view your data! table "mqtt_consumer", topic is the mqtt topic you want to graph.

# Helpful queries
Show all mqtt topics recorded
`curl -G 'http://localhost:8086/query?pretty=true' --data-urlencode "db=telegraf" --data-urlencode "q=SHOW TAG VALUES WITH KEY = \"topic\""`


