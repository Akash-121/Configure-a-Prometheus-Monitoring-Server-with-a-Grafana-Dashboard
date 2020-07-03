# Configure-a-Prometheus-Monitoring-Server-with-a-Grafana-Dashboard

`Configure a Prometheus Monitoring Server with a Grafana Dashboard`

Ref: https://www.scaleway.com/en/docs/configure-prometheus-monitoring-with-grafana/

Prometheus is a flexible monitoring solution that is in development since 2012. The software stores all its data in a time series database and offers a multi-dimensional data-model and a powerful query language to generate reports of the monitored resources.

There are five steps to use Prometheus with Grafana:

# Preparing your Environment
# Downloading and Installing Node Exporter
# Downloading and Installing Prometheus
# Configuring Prometheus
# Downloading and Installing Grafana
# Preparing your Environment
Here, we use an instance running on Ubuntu Xenial (16.04).

1 . To run Prometheus safely on our server, we have to create a user for Prometheus and Node Exporter without the possibility to log in. To achieve this, we use the parameter --no-create-home which skips the creation of a home directory and disable the shell with --shell /usr/sbin/nologin.

>sudo useradd --no-create-home --shell /usr/sbin/nologin prometheus

>sudo useradd --no-create-home --shell /bin/false node_exporter

2 . Create the folders required to store the binaries of Prometheus and its configuration files:
>sudo mkdir /etc/prometheus

>sudo mkdir /var/lib/prometheus

3 . Set the ownership of these directories to our prometheus user, to make sure that Prometheus can access to these folders:

>sudo chown prometheus:prometheus /etc/prometheus

>sudo chown prometheus:prometheus /var/lib/prometheus

`Downloading and Installing Node Exporter`

As your Prometheus is only capable of collecting metrics, we want to extend its capabilities by adding Node Exporter, a tool that collects information about the system including CPU, disk, and memory usage and exposes them for scraping.

1 . Download the latest version of Node Exporter:

>wget https://github.com/prometheus/node_exporter/releases/download/v0.16.0/node_exporter-0.16.0.linux-amd64.tar.gz

2 . Unpack the downloaded archive. This will create a directory node_exporter-0.16.0.linux-amd64, containing the executable, a readme and license file:

>tar xvf node_exporter-0.16.0.linux-amd64.tar.gz

3 . Copy the binary file into the directory /usr/local/bin and set the ownership to the user you have created in step previously:

>sudo cp node_exporter-0.16.0.linux-amd64/node_exporter /usr/local/bin

>sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter

4 . Remove the leftover files of Node Exporter, as they are not needed any longer:

>rm -rf node_exporter-0.16.0.linux-amd64.tar.gz node_exporter-0.16.0.linux-amd64

5 . To run Node Exporter automatically on each boot, a Systemd service file is required. Create the following file by opening it in Nano:

>sudo nano /etc/systemd/system/node_exporter.service

6 . Copy the following information in the service file, save it and exit Nano:
```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```
7 . Collectors are used to gather information about the system. By default a set of collectors is activated. You can see the details about the set in the README-file. If you want to use a specific set of collectors, you can define them in the ExecStart section of the service. Collectors are enabled by providing a--collector.<name> flag. Collectors that are enabled by default can be disabled by providing a --no-collector.<name> flag.
  
8 . Reload Systemd to use the newly defined service:

>sudo systemctl daemon-reload

9 . Run Node Exporter by typing the following command:

>sudo systemctl start node_exporter

10 . Verify that the software has been started successfully:

>sudo systemctl status node_exporter

You will see an output like this, showing you the status active (running) as well as the main PID of the application:
```
● node_exporter.service - Node Exporter
   Loaded: loaded (/etc/systemd/system/node_exporter.service; disabled; vendor preset: enabled)
   Active: active (running) since Mon 2018-06-25 11:47:06 UTC; 4s ago
 Main PID: 1719 (node_exporter)
   CGroup: /system.slice/node_exporter.service
           └─1719 /usr/local/bin/node_exporter
```
11 . If everything is working, enable Node Exporter to be started on each boot of the server:

>sudo systemctl enable node_exporter

`Downloading and Installing Prometheus`

1 . Download and Unpack Prometheus latest release of Prometheus. As exemplified, the version is 2.2.1:

>sudo apt-get update && apt-get upgrade

>wget https://github.com/prometheus/prometheus/releases/download/v2.2.1/prometheus-2.2.1.linux-amd64.tar.gz

>tar xfz prometheus-*.tar.gz

>cd prometheus-*

The following two binaries are in the directory:

Prometheus - Prometheus main binary file

promtool

The following two folders (which contain the web interface, configuration files examples and the license) are in the directory:
consoles

console_libraries

2 . Copy the binary files into the /usr/local/bin/directory:

>sudo cp ./prometheus /usr/local/bin/

>sudo cp ./promtool /usr/local/bin/

3 . Set the ownership of these files to the prometheus user previously created:

>sudo chown prometheus:prometheus /usr/local/bin/prometheus

>sudo chown prometheus:prometheus /usr/local/bin/promtool

4 . Copy the consoles and console_libraries directories to /etc/prometheus:

>sudo cp -r ./consoles /etc/prometheus

>sudo cp -r ./console_libraries /etc/prometheus

5 . Set the ownership of the two folders, as well as of all files that they contain, to our prometheus user:

>sudo chown -R prometheus:prometheus /etc/prometheus/consoles

>sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries

6 . In our home folder, remove the source files that are not needed anymore:

>cd .. && rm -rf prometheus-*

`Configuring Prometheus`

Prior to using Prometheus, it needs basic configuring. Thus, we need to create a configuration file named prometheus.yml

`Note: The configuration file of Prometheus is written in YAML which strictly forbids to use tabs. If your file is incorrectly formatted, Prometheus will not start. Be careful when you edit it.`

1 . Open the file prometheus.yml in a text editor:

>sudo nano /etc/prometheus/prometheus.yml

Prometheus’ configuration file is divided into three parts: global, rule_files, and scrape_configs.

In the global part we can find the general configuration of Prometheus: scrape_interval defines how often Prometheus scrapes targets, evaluation_interval controls how often the software will evaluate rules. Rules are used to create new time series and for the generation of alerts.
The rule_files block contains information of the location of any rules we want the Prometheus server to load.
The last block of the configuration file is named scape_configs and contains the information which resources Prometheus monitors.

Our file should look like this example:
```
global:
  scrape_interval:     15s
  evaluation_interval: 15s

rule_files:
  # - "first.rules"
  # - "second.rules"

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']
```
The global scrape_interval is set to 15 seconds which is enough for most use cases.
We do not have any rule_files yet, so the lines are commented out and start with a #.

In the scrape_configs part we have defined our first exporter. It is Prometheus that monitors itself. As we want to have more precise information about the state of our Prometheus server we reduced the scrape_interval to 5 seconds for this job. The parameters static_configsand targets determine where the exporters are running. In our case it is the same server, so we use localhost and the port 9090.

As Prometheus scrapes only exporters that are defined in the scrape_configs part of the configuration file, we have to add Node Exporter to the file, as we did for Prometheus itself.

We add the following part below the configuration for scrapping Prometheus:
```
- job_name: 'node_exporter'
  scrape_interval: 5s
  static_configs:
    - targets: ['localhost:9100']
```
Overwrite the global scrape interval again and set it to 5 seconds. As we are scarping the data from the same server as Prometheus is running on, we can use localhost with the default port of Node Exporter: 9100.

If you want to scrape data from a remote host, you have to replace localhost with the IP address of the remote server.

For all information about the configuration of Prometheus, you may check the configuration documentation.

2 . Set the ownership of the file to our Prometheus user:

>sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml

Our Prometheus server is ready to run for the first time.


`Running Prometheus`

1 . Start Prometheus directly from the command line with the following command, which executes the binary file as our Prometheus user:

>sudo -u prometheus /usr/local/bin/prometheus --config.file /etc/prometheus/prometheus.yml --storage.tsdb.path /var/lib/prometheus/ --web.console.templates=/etc/prometheus/consoles --web.console.libraries=/etc/prometheus/console_libraries

The server starts displaying multiple status messages and the information that the server has started:
```
level=info ts=2018-04-12T11:56:53.084000977Z caller=main.go:220 msg="Starting Prometheus" version="(version=2.2.1, branch=HEAD, revision=bc6058c81272a8d938c05e75607371284236aadc)"
level=info ts=2018-04-12T11:56:53.084463975Z caller=main.go:221 build_context="(go=go1.10, user=root@149e5b3f0829, date=20180314-14:15:45)"
level=info ts=2018-04-12T11:56:53.084632256Z caller=main.go:222 host_details="(Linux 4.4.127-mainline-rev1 #1 SMP Sun Apr 8 10:38:32 UTC 2018 x86_64 scw-041406 (none))"
level=info ts=2018-04-12T11:56:53.084797692Z caller=main.go:223 fd_limits="(soft=1024, hard=65536)"
level=info ts=2018-04-12T11:56:53.09190775Z caller=web.go:382 component=web msg="Start listening for connections" address=0.0.0.0:9090
level=info ts=2018-04-12T11:56:53.091908126Z caller=main.go:504 msg="Starting TSDB ..."
level=info ts=2018-04-12T11:56:53.102833743Z caller=main.go:514 msg="TSDB started"
level=info ts=2018-04-12T11:56:53.103343144Z caller=main.go:588 msg="Loading configuration file" filename=/etc/prometheus/prometheus.yml
level=info ts=2018-04-12T11:56:53.104047346Z caller=main.go:491 msg="Server is ready to receive web requests."
```
2 . Open your browser and type http://IP.OF.YOUR.SERVER:9090 to access the Prometheus interface. If everything is working, we end the task by pressing on CTRL + C on our keyboard.

Note: If you get an error message when you start the server, double check your configuration file for possible YAML syntax errors. The error message will tell you what to check.

3 . The server is working now, but it cannot yet be launched automatically at boot. To achieve this, we have to create a new systemd configuration file that will tell your OS which services should it launch automatically during the boot process.

>sudo nano /etc/systemd/system/prometheus.service

The service file tells systemd to run Prometheus as prometheus and specifies the path of the configuration files.

4 . Copy the following information in the file and save it, then exit the editor:
```
[Unit]
  Description=Prometheus Monitoring
  Wants=network-online.target
  After=network-online.target

[Service]
  User=prometheus
  Group=prometheus
  Type=simple
  ExecStart=/usr/local/bin/prometheus \
  --config.file /etc/prometheus/prometheus.yml \
  --storage.tsdb.path /var/lib/prometheus/ \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries
  ExecReload=/bin/kill -HUP $MAINPID

[Install]
  WantedBy=multi-user.target
  ```
5 . To use the new service, reload systemd:

>sudo systemctl daemon-reload

We enable the service so that it will be loaded automatically during boot:

>sudo systemctl enable prometheus

6 . Start Prometheus:

>sudo systemctl start prometheus

Your Prometheus server is ready to be used.

We have now installed Prometheus to monitor your instance.


`Prometheus Web Interface`

Prometheus provides a basic web server running on http://your.server.ip:9000 that provide access to the data collected by the software.
We can verify the status of our Prometheus server from the interface:

Moreover, do some queries in the data that has been collected.

The interface is very lightweight, and the Prometheus team recommend to use a tool like Grafana if you want to do more than testing and debugging the installation.


`Installing Grafana`

1 . Install Grafana on our instance which queries our Prometheus server.

>wget https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana_5.0.4_amd64.deb

>sudo apt-get install -y adduser libfontconfig

>sudo dpkg -i grafana_5.0.4_amd64.deb

2 . Enable the automatic start of Grafana by systemd:

>sudo systemctl daemon-reload && sudo systemctl enable grafana-server && sudo systemctl start grafana-server

Grafana is running now, and we can connect to it at http://your.server.ip:3000. The default user and password is admin / admin.

Now you have to create a Prometheus data source
:
Click on the Grafana logo to open the sidebar.

Click on “Data Sources” in the sidebar.

Choose “Add New”.

Select “Prometheus” as the data source.

Set the Prometheus server URL (in our case: http://localhost:9090/)

Click “Add” to test the connection and to save the new data source.

Your settings should look like this:

You are now ready to create your first dashboard from the information collected by Prometheus. You can also import some dashboards from a collection of shared dashboards

Here is an example of a Dashboard that uses the CPU usage of our node and presents it in Grafana:

In this tutorial, we were able to configure a Prometheus server with two data collectors that are scraped by our Prometheus server which provides the data to build Dashboards with Grafana. Don’t hesitate to consult the official documentation of Prometheus and Grafana.

We can install or pull container exporter to export running containers metrics on docker. There is an alternative exporter cadvisor which has rich UI to monitor containers status and also export metrics to grafana.

We can pull prometheus to see the status of the container or service and visualize metrics in grafana with grafana build in shared dashboards.

Now we will work with alert manager.

Install Alertmanager to Alert based on Metrics from Prometheus

Ref: https://sysadmins.co.za/install-alertmanager-to-alert-based-on-metrics-from-prometheus/

So we are pushing our time series metrics into prometheus, and now we would like to alarm based on certain metric dimensions. That's where alertmanager fits in. We can setup targets and rules, once rules for our targets does not match, we can alarm to destinations suchs as slack.

`Install Alertmanager`

Create the user for alertmanager:

>$ sudo useradd --no-create-home --shell /bin/false alertmanager

Download alertmanager and extract:

>$ wget https://github.com/prometheus/alertmanager/releases/download/v0.17.0/alertmanager-0.17.0.linux-amd64.tar.gz

>$ sudo tar -xvf alertmanager-0.17.0.linux-amd64.tar.gz

Move alertmanager and amtool birnaries in place:

>$ sudo cp alertmanager-0.17.0.linux-amd64/alertmanager /usr/local/bin/

>$ sudo cp alertmanager-0.17.0.linux-amd64/amtool /usr/local/bin/

Ensure that the correct permissions are in place:

>$ sudo chown alertmanager:alertmanager /usr/local/bin/alertmanager

>$ sudo chown alertmanager:alertmanager /usr/local/bin/amtool

`Cleanup:`

>$ sudo rm -rf alertmanager-0.17.0*

`Configure the Alarm definition:`

Create an alarm definition that describes that defines when to notify when an endpoint goes down:

>$ sudo vim /etc/prometheus/alert.rules.yml

And our alert definition:

Ensure that the permission is set:

>$ sudo chown prometheus:prometheus /etc/prometheus/alert.rules.yml

Use the promtool to validate that the alert is correctly configured:

>$ sudo promtool check rules /etc/prometheus/alert.rules.yml
```
Checking /etc/prometheus/alert.rules.yml
  SUCCESS: 1 rules found
  ```
If everything is good, restart prometheus:

>$ sudo systemctl restart prometheus


```Configure Alertmanager:```

Create the alertmanager directory and configure the global alertmanager configuration:

>$ sudo mkdir /etc/alertmanager

>$ sudo vim /etc/alertmanager/alertmanager.yml

Now we will create a slack channel and add an app named incoming webhook and go to its setting. We will create a webhook for this specific channel.
Provide the global config and ensure to populate your personal information.

Ensure the permissions are in place:

>$ sudo chown alertmanager:alertmanager -R /etc/alertmanager

Create the alertmanager systemd unit file:

>$ sudo  vim /etc/systemd/system/alertmanager.service

And supply the unit file configuration. Note that I am exposing port 9093 directly as Im not using a reverse proxy.
```
[Unit]
Description=Alertmanager
Wants=network-online.target
After=network-online.target

[Service]
User=alertmanager
Group=alertmanager
Type=simple
WorkingDirectory=/etc/alertmanager/
ExecStart=/usr/local/bin/alertmanager --config.file=/etc/alertmanager/alertmanager.yml --web.external-url http://0.0.0.0:9093

[Install]
WantedBy=multi-user.target
```
Now we need to inform prometheus that we will send alerts to alertmanager to it's exposed port:

>$ sudo vim /etc/prometheus/prometheus.yml

And supply the alertmanager configuration for prometheus:

So when we get alerted, our alert will include a link to our alert. We need to provide the base url of that alert. That get's done in our alertmanager systemd unit file: /etc/systemd/system/alertmanager.service under --web.external-url passing the alertmanager base ip address:

Then we need to do the same with the prometheus systemd unit file: /etc/systemd/system/prometheus.service under --web.external-url passing the prometheus base ip address:

Since we have edited the systemd unit files, we need to reload the systemd daemon:

>$ sudo systemctl daemon-reload

Then restart prometheus and alertmanager:

>$ sudo systemctl restart prometheus

>$ sudo systemctl restart alertmanager

Inspect the status of alertmanager and prometheus:

>$ sudo systemctl status alertmanager

>$ sudo systemctl status prometheus

If everything seems good, enable alertmanager on boot:

>$ sudo systemctl enable alertmanager


`Access Alertmanager`

Access alertmanager on your endpoint on port 9093

From our previous tutorial we started a local web service on port 8080 that is being monitored by prometheus. Let's stop that service to test out the alert.
And check the notification via slack:

When you start the service again and head over to the prometheus ui under alerts, you will see that the service recovered:

`Install Prometheus Alert manager Plugin`

Install the Prometheus Alertmanager Plugin in Grafana. Head to the instance where grafana is installed and install the plugin:

>$ sudo grafana-cli plugins install camptocamp-prometheus-alertmanager-datasource

Once the plugin is installed, restart grafana:

>$ sudo service grafana-server restart

Install the dasboard grafana.com/dashboards/8010. Create a new datasource, select the prometheus-alertmanager datasource, configure and save.
Add a new dasboard, select import and provide the ID 8010, select the prometheus-alertmanager datasource and save. 
