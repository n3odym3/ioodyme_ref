# Installation et setup de NodeRed/Zigbee2MQTT Inlfuxdb et Grafana

## NodeRed (ou utiliser un container tout fait)

### Installation de NodeJS
```
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt install -y nodejs
```
### Installation de NodeRed
Par défaut sur le port **1880**
```
sudo npm install -g --unsafe-perm node-red
```
Rajouter le module 
```
node-red-contrib-influxdb
```

## Zigbee2MQTT 
https://www.zigbee2mqtt.io/

## Grafana (ou utiliser un container tout fait)
Par défaut sur le port **3000**
```
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
sudo apt update
sudo apt install grafana
sudo systemctl start grafana-server
```
Ajouter DB
- Config > Data source (ou Add data source menu d'origine)
- Name : NOM de la BD
- Cocher Basic Auth
- Remplir Basic Auth Details
- Remplir InfluxDB Details avec NOM de la BD et user login
- Save and TEST

## InfluxDB (ou utiliser un container tout fait)
Par défaut sur le port **8086**
```
sudo apt install influxdb
sudo apt install influxdb-client
```
Editer influxdb.conf
```
sudo nano /etc/influxdb/influxdb.conf
```
Le fichier conf devrait ressembler à ça au minimum

```
[meta]                                                                                            
  dir = "/var/lib/influxdb/meta"                                                                  
                                                                                                  
[data]                                                                                            
  dir = "/var/lib/influxdb/data"                                                                  
  engine = "tsm1"                                                                                 
  wal-dir = "/var/lib/influxdb/wal"                                                               
                                                                                                  
[http]                                                                                            
enabled = true                                                                                    
bind-adress = ":8086"                                                                             
                                                                                                  
[retention]                                                                                       
eanble = true                                                                                     
check-interval = "30m"                                                                            
                                                                                                  
[continuous_queries]                                                                              
enabled = true                                                                                    
run-interval = "1m" 
```
### Cheat sheet

Création d'un user et connection
```
CREATE USER "user" WITH PASSWORD 'password' WITH ALL PRIVILEGES
SHOW USERS
influx -username nom -password motdepasse
```
Création d'une DB
```
CREATE DATABASE NOM
SHOW DATABASES
```
Accès à des données
```
USE nomDB
SHOW FIELD KEYS
SELECT * FROM nommeasurement LIMIT 10
```

Retention policies et continuous queries
```
CREATE RETENTION POLICY "1week" ON "DB01" DURATION 1w REPLICATION 1 DEFAULT
CREATE RETENTION POLICY "30days" ON "DB01" DURATION 30d REPLICATION 1
CREATE RETENTION POLICY "1year" ON "DB01" DURATION 365d REPLICATION 1

CREATE CONTINUOUS QUERY "mean-1h" ON "DB01" RESAMPLE EVERY 1h FOR 24h BEGIN SELECT mean("Temp") AS "Temp", mean("Hum") AS "Hum" INTO "30days"."DHT22-ESP-1h" FROM "DHT22-ESP" GROUP BY time(1h) END
CREATE CONTINUOUS QUERY "mean1h" ON "DB01" BEGIN SELECT mean("Temp") AS "Temp1h",mean("Hum") AS "Hum1h" INTO "30days"."DHT22-ESP-1h" FROM "DHT22-ESP" GROUP BY time(1h) END
CREATE CONTINUOUS QUERY "mean1d" ON "DB01" BEGIN SELECT mean("Temp") AS "Temp1d",mean("Hum") AS "Hum1d" INTO "1year"."DHT22-ESP-1d" FROM "DHT22-ESP-1h" GROUP BY time(1d) END
```




