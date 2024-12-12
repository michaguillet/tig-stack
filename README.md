# TIG stack (Telegraf/InfluxDB/Grafana)
[Telegraf](https://www.influxdata.com/time-series-platform/telegraf/) is a plugin-driven server agent for collecting and reporting metrics.  
[InfluxDB](https://www.influxdata.com/time-series-platform/influxdb/) handle massive amounts of time-stamped information.  
[Grafana](https://grafana.com/) is an open platform for beautiful analytics and monitoring.  

![System dashboard](./docs/system_dashboard.png?raw=true "System dashboard")
![Docker dashboard](./docs/docker_dashboard.png?raw=true "Docker dashboard")

## Requirements
As docker images, TIG stack needs:

* docker v18.* at least
* docker-compose v1.2* at least

To be installed on your machine.

## How to use it?
`.env` to the root directory exposes environment variables:

* **TELEGRAF_HOST** - agent hostname
* **INFLUXDB_HOST** - database hostname
* **INFLUXDB_PORT** - database port
* **INFLUXDB_DATABASE** - database name
* **INFLUXDB_ADMIN_USER** - admin user
* **INFLUXDB_ADMIN_PASSWORD** - admin password
* **GRAFANA_PORT** - monitoring port
* **GRAFANA_USER** - monitoring user
* **GRAFANA_PASSWORD** - monitoring password
* **GRAFANA_PLUGINS_ENABLED** - enable monitoring plugins
* **GRAFANA_PLUGINS** - monitoring plugins list (fetch all available plugins if empty)
* **MQTT_BROKER** - IP of the MQTT Broker
* **MQTT_TOPICS** - Subscribed topics
* **MQTT_USER** - User to authenticate
* **MQTT_PASSWORD** - Password to authenticate

Modify it according to your needs and build your custom TIG stack:

```bash
docker compose up -d
```

### Known issues
* `docker compose` command fails for non-root user
    1. Create the `docker` group if not exists:

    ```bash
    sudo groupadd docker
    ``` 

    2. Add your user to the `docker` group:

    ```bash
    sudo usermod -aG docker $USER
    ```

    3. Reboot your machine

Then access graphana at `http://localhost:3000` (or replace the default port with your **GRAFANA_PORT**'s value).


## TIG-Stack Learnings

### Install Mosquitto Clients in Container

```bash
apt install -y mosquitto-clients
```

### Publish MQTT Messages to the `sensor` Topic

Use the following commands to publish control and sensor data to the `sensor` topic:

```bash
mosquitto_pub -h mosquitto_broker -p 1883 -t "sensor" -m "controls,motor=1 state=\"red\""

mosquitto_pub -h mosquitto_broker -p 1883 -t "sensor" -m "temperature,sensor=1,location=livingroom value=34.2"
mosquitto_pub -h mosquitto_broker -p 1883 -t "sensor" -m "temperature,sensor=1,location=office value=34.2"
mosquitto_pub -h mosquitto_broker -p 1883 -t "sensor" -m "temperature,sensor=1,location=office value=24.2"
mosquitto_pub -h mosquitto_broker -p 1883 -t "sensor" -m "temperature,sensor=1,location=office value=15.2"
mosquitto_pub -h mosquitto_broker -p 1883 -t "sensor" -m "temperature,sensor=1,location=office value=23.2"
```

### Steps to Visualize Data in TIG-Stack

1. **Start the Services**:
   ```bash
   docker compose up -d
   ```

2. **Publish MQTT Messages**:
   Send data to the `plantfarming123` topic as shown above. The messages will be consumed by Telegraf and inserted into InfluxDB.

3. **Interact with InfluxDB**:
   Access the InfluxDB container and query the database:
   ```bash
   docker exec -it <influxdb_container_name> influx
   ```

4. **Grafana Demo Query**:
   In Grafana, you can use the following query to retrieve data:
   ```sql
   SELECT value FROM temperature WHERE "location" = 'livingroom';
   ```

5. **Grafana Login**:
   ```
   Username: admin
   Password: admin
   ```

6. **Telegraf MQTT Configuration**:
   In the `telegraf.conf.template` file, search for the `mqtt_consumer` section to adjust the MQTT settings.
