# IMPORTANT NOTE

Reworked from the great repo by yukulehe/gazpar2mqtt, now that GRDF is again without Captcha.
Main differences are in the login method, now using virtual browser (old method still does not work), and allowing to export to HA Long Term Statistics

### Important (27/May/2024): 
- working to collect data in SQLite and MQTT but under development, various functions may disappear or be modified.
- no verification if the now 2-year old code from yukulehe/gazpar2mqtt is still valid **in its entirety**
  - not tested (and maybe to be omitted): influx
- only tested via docker image and docker-compose, see [INSTALLATION](https://github.com/vingerha/gazpar_2_mqtt?tab=readme-ov-file#installation-and-usage)
- This repo does not support add-ons for HAOS, I donot run that and have not (yet) the knowledge on how to accomplish this
  - I did ask [@AlexBelgium](https://github.com/alexbelgium/alexbelgium/commits?author=alexbelgium) sif he would consider this repo to be part of his library....

For now: **USE AT YOUR OWN DISCRETION**

# gazpar2mqtt
Python script to fetch GRDF's website data and publish data to a mqtt broker

![Gazpar logo](res/gazpar.png)
<img src="res/mqtt.png?raw=true" width="250" height="250">

## Changelogs :
- 0.2.0 :
  - add export to HA Long Term Statistic
  - fixes/tuning/cleaning
- 0.1.0 :
  - Basis from yukulehe/gazpar2mqtt
  - add login via selenium
  Still contains functionality from yukulehe/gazpar2mqtt but not tested
  - Grafana dashboard template
  - Cost calculation from prices file

## Roadmap :

- Home assistant custom entity card (low prio)

# Requirements

## GRDF Gazpar API

Verify you have gazpar data available on [GRDF Portal](https://monespace.grdf.fr/monespace/connexion)
Remember, kWh provided is conversion factor dependant. Please verify it's coherent with your provider bills.

Gazpar2mqtt request the API and retrieve 4 groups of data :

### Account informations

It corresponds to the customer profile, the list of PCE (Point de Comptage et d'Estimation) and its attributes (address, state, activation date)  

### Informative measures

GRDF provides informative measures at day level. The tool returns the last measure and several calculated indicators. 

### Published measures

GRDF provides published measures. It corresponds to consumptions measured by GRDF and transmitted to your gas supplier. Consequently, it should correspond to the consumption that the supplier invoices to the consumer.

### Thresholds

Thresholds (Seuil) can be set on GRDF website for current and future months.
![image](https://user-images.githubusercontent.com/31646663/147261711-e74beca2-aa2c-49f9-b994-2dc3f013eaa2.png)
![image](https://user-images.githubusercontent.com/31646663/147261741-5a45c17c-f621-4e1e-86e0-964eef81b435.png)

The script can retrieve those informations and publish a warn when the threshold is close to being reached.

## Prices

Gazpar2mqtt can ingest your own energy prices to estimate your costs. Download and complete the sample file prices.csv using the following format :

- PCE : your PCE id
- startDate : starting date of the price
- endDate : end date of the price (put a far far end date at the last row of the file, ie : 2999-12-31)
- kwhPrice : the price per kWh (don't forget to apply the TVA to this cost, and to add the french energy tax per kwh)
- fixPrice : the fix price per day (it corresponds for example to the price of your subscription with your gas provider, don't forget to apply the TVA)

``` 
pce;startDate;endDate;kwhPrice;fixPrice
XXXX;2019-11-28;2020-12-27;0.03713;0.25
XXXX;2020-12-28;2021-01-25;0.03472;0.30
XXXX;2021-01-26;2099-12-31;0.03733;0.35
``` 
Then precise in parameters the path to the file prices.csv.

## MQTT broker

A MQTT broker is required. Please check its configuration (hostname, port, remote access allowed, username & password if needed).


## Parameters

Currently, parameters can be provided by command's arguments or by the OS's environment variables.

Mandatory :

| Variable | Description |
| --- | --- |
| **GRDF_USERNAME** | Your GRDF login (ex : myemail@email.com) |
| **GRDF_PASSWORD** | Your GRDF password |
| **MQTT_HOST** | Hostname or ip adress of the MQTT broker |

Optional :

| Variable                | Description                                           | Default value        |
|-------------------------|-------------------------------------------------------|----------------------|
| **SCHEDULE_TIME**       | Time for refreshing data everyday                     | None (format : 14:30) |
| **MQTT_PORT**           | Port of the MQTT broker                               | 1883                 |
| **MQTT_TOPIC**          | Topic used as prefix                                  | gazpar               |
| **MQTT_CLIENTID**       | Client id to be used for connexion to the MQTT broker | gazou                |
| **MQTT_USERNAME**       | Username to be used for connexion to the MQTT brokerr |                      |
| **MQTT_PASSWORD**       | Password to be used for connexion to the MQTT broker  |                      |
| **MQTT_QOS**            | QOS for message publishing (0, 1 or 2)                | 1                    |
| **MQTT_RETAIN**         | Retain flag                                           | False                |
| **MQTT_SSL**            | Enable MQTT SSL connexion                             | False                |
| **STANDALONE_MODE**     | Enable standalone publication mode                    | True                 |
| **HASS_DISCOVERY**      | Enable Home assistant dicovery mode                   | False                |
| **HASS_PREFIX**         | Home assistant topic prefix                           | homeassistant        |
| **HASS_DEVICE_NAME**    | Home assistant device name                            | gazpar               |
| **HASS_LTS**            | enable export to HA long Term Stats                   | False                |
| **HASS_LTS_TOKEN**      | HA Long Lived Access token from host below            | ""                   |
| **HASS_LTS_URI**        | Home assistant uri for LTS                            |/api/services/recorder/import_statistics|
| **HASS_LTS_HOST**       | Home assistant server/host                            | http://192.168.x.y:8123|
| **THRESOLD_PERCENTAGE** | Percentage of the thresold to be reached              | 80                   |
| **PRICE_PATH**          | Path to price.csv file                                | /data                |
| **PRICE_KWH_DEFAULT**   | Energy price in € per kWh                             | 0.04                 |
| **PRICE_FIX_DEFAULT**   | Fix price in € per day                                | 0.0                  |
| **INFLUXDB_ENABLE**     | Activate export to Influxdb v2                        | False                |
| **INFLUXDB_HOST**       | Host of influxdb                                      |                      |
| **INFLUXDB_PORT**       | Port of influxdb                                      | 8086                 |
| **INFLUXDB_ORG**        | Influxdb organization                                 |                      |
| **INFLUXDB_BUCKET**     | Influxdb bucket                                       |                      |
| **INFLUXDB_TOKEN**      | Influxdb token with read access to the bucket         |                      |
| **INFLUXDB_HORIZON**    | Number of days in the past to be write to Influxdb    | 0 (= all data)       |
| **DB_INIT**             | Force reinitialization of the database                | False                |
| **DB_PATH**             | Database path                                         | /data                |
| **DEBUG**               | Enable debug mode                                     | False                |


# Installation and Usage

## Docker
- create a folder ([yourdocker_dir]/gazpar_2_mqtt) to map to the container volume
- in that folder, create a folder app ([yourdocker_dir]/gazpar_2_mqtt/app)
- run docker command below, the very first time it will copy all necessary files into [yourdocker_dir]/gazpar_2_mqtt/app
- it will then CRASH as it does not have the credentials
- modify the param.py to match grdf and mqtt ip/port and (if needed) mqtt credentials
- restart the container
[Docker location](https://hub.docker.com/repository/docker/vingerha/gazpar_2_mqtt/general)

Example of docker run command, assuming param.py having the credentials:

``` 
docker run --name gazpar_2_mqtt -v [yourdocker_dir]/gazpar_2_mqtt/app:/app -v [yourdocker_dir/gazpar_2_mqtt:/data --tty vingerha/gazpar_2_mqtt:latest
```

## Docker compose

Even easier.... just create a folder and place the docker-compose.yml, example:
```
/home/docker/gazpar_2_mqt
me@ha-nuc:~/docker/gazpar_2_mqtt_2$ ls -la
total 112
drwxrwxrwx 1 root root   4096 Apr 28 17:47 .
drwxrwxrwx 1 root root   4096 Apr 25 18:23 ..
-rwxrwxrwx 1 root root   1909 Apr 28 17:42 docker-compose.yml
```
Edit the docker-compose.yml to match your setup, see [parameter](https://github.com/vingerha/gazpar_2_mqtt?tab=readme-ov-file#parameters)

Unles you are pretty sure what you are doing, it is recommend to do this in steps, start with the mandatory ones first and any connection credentials to MQTT

Then, execute the docker compose run applicable for your OS
e.q. on ubuntu this is
```
docker-compose run --rm gazpar2mqtt
```
After that it will have created the db and you should have data in MQTT
```
me@ha-nuc:~/docker/gazpar_2_mqtt_2$ ls -la
total 112
drwxrwxrwx 1 root root   4096 Apr 28 17:47 .
drwxrwxrwx 1 root root   4096 Apr 25 18:23 ..
-rwxrwxrwx 1 root root   1909 Apr 28 17:42 docker-compose.yml
-rwxrwxrwx 1 root root 102400 Apr 27 16:25 gazpar2mqtt.db
```
From here on you can extend the docker-compose.yml e.g. to HA Long Term Statistics

## Manual without container

to be added in near future (requires a.o. chromium to be pre-installed)

## Standalone mode

Default mode, gazpar2mqtt is autonomous and is not dependent of any third-party tool.
You can replace the default topic prefix *gazpar* (see mqtt broker requirements chapter)

### Measures :

Last measures :

| Topic                        | Description |
|------------------------------| --- |
| gazpar/PCE/last/index        | Gas index in m3 of the last measure |
| gazpar/PCE/last/date              | Date of the last measure |
| gazpar/PCE/last/energy            | Gas consumption in kWh of the last measure |
| gazpar/PCE/last/gas               | Gas consumption in m3 of the last measure  |
| gazpar/PCE/last/conversion_factor | Conversion factor in kWh/m3 of the last measure  |

Published  measures :

| Topic                                  | Description                                               |
|----------------------------------------|-----------------------------------------------------------|
| gazpar/PCE/published/index             | Gas index in m3 of the last published measure             |
| gazpar/PCE/published/start_date        | Start date of the period of the last published measure    |
| gazpar/PCE/published/end_date          | End date of the period of the last published measure                                   |
| gazpar/PCE/published/energy            | Gas consumption in kWh of the last published measure      |
| gazpar/PCE/published/gas               | Gas consumption in m3 of the last published measure       |
| gazpar/PCE/published/conversion_factor | Conversion factor in kWh/m3 of the last published measure |


Calculated calendar measures :

| Topic | Description |
| --- | --- |
| gazpar/PCE/histo/current_year_gas | Gas consumption in m3 of current year |
| gazpar/PCE/histo/previous_year_gas | Gas consumption in m3 of previous year |
| gazpar/PCE/histo/current_month_gas | Gas consumption in m3 of current month |
| gazpar/PCE/histo/previous_month_gas | Gas consumption in m3 of previous month |
| gazpar/PCE/histo/current_month_previous_year_gas | Gas consumption in m3 of current month, 1 year ago |
| gazpar/PCE/histo/current_week_gas | Gas consumption in m3 of current week |
| gazpar/PCE/histo/previous_week_gas | Gas consumption in m3 of previous week |
| gazpar/PCE/histo/current_week_previous_year | Gas consumption in m3 of current week, 1 year ago |
| gazpar/PCE/histo/day-1_gas | Gas consumption in m3, 1 day ago |
| gazpar/PCE/histo/day-2_gas | Gas consumption in m3, 2 day ago |
| gazpar/PCE/histo/day-3_gas | Gas consumption in m3, 3 day ago |
| gazpar/PCE/histo/day-4_gas | Gas consumption in m3, 4 day ago |
| gazpar/PCE/histo/day-5_gas | Gas consumption in m3, 5 day ago |
| gazpar/PCE/histo/day-6_gas | Gas consumption in m3, 6 day ago |
| gazpar/PCE/histo/day-7_gas | Gas consumption in m3, 7 day ago |

Calculated rolling measures :

| Topic | Description |
| --- | --- |
| gazpar/PCE/histo/rolling_year_gas | Gas consumption in m3 for 1 rolling year |
| gazpar/PCE/histo/rolling_year_last_year_gas | Gas consumption in m3 for 1 rolling year, 1 year ago |
| gazpar/PCE/histo/rolling_month_gas | Gas consumption in m3 for 1 rolling month |
| gazpar/PCE/histo/rolling_month_last_month_gas | Gas consumption in m3 for 1 rolling month, 1 month ago |
| gazpar/PCE/histo/rolling_month_last_year_gas | Gas consumption in m3 for 1 rolling month, 1 year ago |
| gazpar/PCE/histo/rolling_month_last_2_year_gas | Gas consumption in m3 for 1 rolling month, 2 years ago |
| gazpar/PCE/histo/rolling_week_gas | Gas consumption in m3 for 1 rolling week |
| gazpar/PCE/histo/rolling_week_last_week_gas | Gas consumption in m3 for 1 rolling week, 1 week ago |
| gazpar/PCE/histo/rolling_week_last_year_gas | Gas consumption in m3 for 1 rolling week, 1 year ago |
| gazpar/PCE/histo/rolling_week_last_2_year_gas | Gas consumption in m3 for 1 rolling week, 2 years ago |

Calculated thresold measures :

| Topic | Description |
| --- | --- |
| gazpar/PCE/thresold/current_month_thresold | Thresold in kWh of current month |
| gazpar/PCE/thresold/current_month_thresold_percentage | Percentage of energy consumption and thresold of current month |
| gazpar/PCE/thresold/current_month_thresold_problem | Warning when energy consumption is higher than 80% of the thresold of current month |
| gazpar/PCE/thresold/previous_month_thresold | Thresold in kWh of previous month |
| gazpar/PCE/thresold/previous_month_thresold_percentage | Percentage of energy consumption and thresold of previous month |
| gazpar/PCE/thresold/previous_month_thresold_problem | Warning when energy consumption is higher than 80% of the thresold of previous month |

Note : thresold percentage can be editable in environment variable

### Status :

| Topic | Description |
| --- | --- |
| gazpar/PCE/status/date | Last execution date time of gazpar2mqtt |
| gazpar/PCE/status/value | Last execution status of  gazpar2mqtt (ON or OFF) |


## Home Assistant discovery mode

![HA_logo](https://user-images.githubusercontent.com/31646663/141127001-6f868a1a-1820-45bc-9f3b-c3ee2a4d2d06.png)

Gazpar2mqtt provides Home Assistant compatible Mqtt devices. The discovery function enable to use MQTT devices with a minimal configuration effort.
Have a look to [Home Assistant Mqtt discovery documentation](https://www.home-assistant.io/docs/mqtt/discovery/).

### Device :
| Device name | Device ID | Model | Manufacturer |
| --- | --- | --- | --- |
| gazpar | gazpar_PCE | PCE | GRDF |

Note : you can replace the default device name *gazpar* by editing the related parameter.

### List of entities :

Last informative measure entities :

| Entity name | Component | Device class | Description |
| --- | --- | --- | --- |
| gazpar_PCE_index | Sensor | Gas | Gas index in m3 of the last measure |
| gazpar_PCE_gas | Sensor | Gas | Gas consumption in m3 of the last measure |
| gazpar_PCE_energy | Sensor | Energy | Gas consumption in kWh of the last measure |
| gazpar_PCE_consumption_date | Sensor | Date | Date of the last measure |
| gazpar_PCE_connectivity | Binary sensor | Connectivity | Binary sensor which indicates if the last gazpar statement succeeded (ON) or failed (OFF) |

Last published measure entities :

| Entity name                                 | Component | Device class | Description                                                                               |
|---------------------------------------------| --- | --- |-------------------------------------------------------------------------------------------|
| gazpar_PCE_published_index                  | Sensor | Gas | Gas index in m3 of the last published measure                                             |
| gazpar_PCE_published_gas                    | Sensor | Gas | Gas consumption in m3 of the last published measure                                       |
| gazpar_PCE_published_energy                 | Sensor | Energy | Gas consumption in kWh of the last published measure                                      |
| gazpar_PCE_published_consumption_start_date | Sensor | Date | Start date of the last published measure                                                  |
| gazpar_PCE_published_consumption_end_date   | Sensor | Date | End date of the last published measure                                                    |

Calendar measure entities :

| Entity name | Component | Device class | Description |
| --- | --- | --- | --- |
| gazpar_PCE_current_year_gas | Sensor | Gas | Gas consumption in m3 of current year |
| gazpar_PCE_previous_year_gas | Sensor | Gas | Gas consumption in m3 of previous year |
| gazpar_PCE_previous_2_year_gas | Sensor | Gas | Gas consumption in m3 of previous 2 years |
| gazpar_PCE_current_month_gas | Sensor | Gas | Gas consumption in m3 of current month |
| gazpar_PCE_previous_month_gas | Sensor | Gas | Gas consumption in m3 of previous month |
| gazpar_PCE_current_month_last_year_gas | Sensor | Gas | Gas consumption in m3 of current month, 1 year ago |
| gazpar_PCE_current_week_gas | Sensor | Gas | Gas consumption in m3 of current week |
| gazpar_PCE_previous_week_gas | Sensor | Gas | Gas consumption in m3 of current week |
| gazpar_PCE_current_week_last_year_gas | Sensor | Gas | Gas consumption in m3 of current week, 1 year ago |
| gazpar_PCE_current_day_1_gas | Sensor | Gas | Gas consumption in m3 of day -1 |
| gazpar_PCE_current_day_2_gas | Sensor | Gas | Gas consumption in m3 of day -2 |
| gazpar_PCE_current_day_3_gas | Sensor | Gas | Gas consumption in m3 of day -3 |
| gazpar_PCE_current_day_4_gas | Sensor | Gas | Gas consumption in m3 of day -4 |
| gazpar_PCE_current_day_5_gas | Sensor | Gas | Gas consumption in m3 of day -5 |
| gazpar_PCE_current_day_6_gas | Sensor | Gas | Gas consumption in m3 of day -6 |
| gazpar_PCE_current_day_7_gas | Sensor | Gas | Gas consumption in m3 of day -7 |


Rolling measure entities :

| Entity name | Component | Device class | Description |
| --- | --- | --- | --- |
| gazpar_PCE_rolling_year_gas | Sensor | Gas | Gas consumption in m3 for 1 rolling year |
| gazpar_PCE_rolling_year_last_year_gas | Sensor | Gas | Gas consumption in m3 for 1 rolling year, 1 year ago |
| gazpar_PCE_rolling_rolling_month_gas | Sensor | Gas | Gas consumption in m3 for 1 rolling month |
| gazpar_PCE_rolling_month_last_month_gas | Sensor | Gas | Gas consumption in m3 for 1 rolling month, 1 month ago |
| gazpar_PCE_rolling_month_last_year_gas | Sensor | Gas | Gas consumption in m3 for 1 rolling month, 1 year ago |
| gazpar_PCE_rolling_month_last_2_year_gas | Sensor | Gas | Gas consumption in m3 for 1 rolling month, 2 years ago |
| gazpar_PCE_rolling_rolling_week_gas | Sensor | Gas | Gas consumption in m3 for 1 rolling week |
| gazpar_PCE_rolling_week_last_week_gas | Sensor | Gas | Gas consumption in m3 for 1 rolling week, 1 week ago |
| gazpar_PCE_rolling_week_last_year_gas | Sensor | Gas | Gas consumption in m3 for 1 rolling week, 1 year ago |
| gazpar_PCE_rolling_week_last_2_year_gas | Sensor | Gas | Gas consumption in m3 for 1 rolling week, 2 years ago |

Calculated thresold measures :

| Entity name | Component | Device class | Description |
| --- | --- | --- | --- |
| gazpar_PCE_current_month_thresold | Sensor | Energy | Thresold in kWh of current month |
| gazpar_PCE_current_month_thresold | Sensor | None | Percentage of energy consumption and thresold of current month  |
| gazpar_PCE_current_month_thresold | Binary sensor | Problem | Warning when energy consumption is higher than 80% of the thresold of current month  |
| gazpar_PCE_previous_month_thresold | Sensor | Energy | Thresold in kWh of previous month |
| gazpar_PCE_previous_month_thresold | Sensor | None | Percentage of energy consumption and thresold of previous month  |
| gazpar_PCE_previous_month_thresold | Binary sensor | Problem | Warning when energy consumption is higher than 80% of the thresold of previous month  |

Note : thresold percentage can be editable in environment variable. 

### Add-on
For Hass.io users, gazpar2mqtt is also available as an add-on provided by [alexbelgium](https://github.com/alexbelgium) (thanks you to him). Please visit the dedicated [repository](https://github.com/alexbelgium/hassio-addons/tree/master/gazpar2mqtt).


## InfluxDb & Grafana

You can activate export of data to your InfluxDb v2 database. InfluxDb v1 is currently not supported.

Example of a Grafana dashboard using InfluxDb v2 and gazpar2mqtt measures :
<img src="res/grafana_sample.png?raw=true" width="70%">

To integrate this dashboard into Grafana, copy/paste content of file [grafana_sample.json](https://github.com/yukulehe/gazpar2mqtt/blob/main/sample/grafana_sample.json).
