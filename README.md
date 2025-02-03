# loki
https://sbcode.net/grafana/install-loki-service/

----

Download and Install Loki Binary

We will install the Loki binary as a service on our existing Grafana server.

To check the latest version of Grafana Loki, visit the Loki releases page. https://github.com/grafana/loki/releases/

cd /usr/local/bin

curl -O -L "https://github.com/grafana/loki/releases/download/v3.2.0/loki-linux-amd64.zip"

A new Ubuntu is unlikely to have unzip pre-installed, so install it using the command,

apt install unzip

Now unzip the downloaded file,

unzip "loki-linux-amd64.zip"

And allow execute permission on the Loki binary

chmod a+x "loki-linux-amd64"

Create the Loki config

Download the Loki config that matches the version.

wget https://raw.githubusercontent.com/grafana/loki/v3.2.0/cmd/loki/loki-local-config.yaml

Configure Loki to run as a service

Now we will configure Loki to run as a service so that it stays running in the background.

Create a user specifically for the Loki service

useradd --system loki

Create a file called loki.service

nano /etc/systemd/system/loki.service

Add the script and save

[Unit]
Description=Loki service
After=network.target

[Service]
Type=simple
User=loki
ExecStart=/usr/local/bin/loki-linux-amd64 -config.file /usr/local/bin/loki-local-config.yaml

[Install]
WantedBy=multi-user.target

Now start and check the service is running.

service loki start
service loki status

We can now leave the new Loki service running.

If you ever need to stop the new Loki service, then type

service loki stop
service loki status

Note it may take a minute to stop.

Warning

If you reboot your server, the Loki Service may not restart automatically.

You can set the Loki service to auto restart after reboot by entering,

systemctl enable loki.service

Troubleshooting
Permission Denied, Internal Server Error

If you get any of these errors

    Loki: Internal Server Error. 500. open /tmp/loki/index/index_2697: permission denied
    "failed to flush user" "open /tmp/loki/chunks/...etc : permission denied"
    Loki: Internal Server Error. 500. Internal Server Error

You should check the owner of the folders configured in the storage_config section of the the loki-local-config.yaml to match the name of the user configured in the loki.service script above.

My user is loki, and the folders begins with /tmp/loki so I recursively set the owner.

chown -R loki:loki /tmp/loki

You may need to restart the Loki service and checking again its status.

If when connecting to Loki using the Grafana data source configuration, you see the error Loki: Bad Gateway. 502. Bad Gateway, this will happen if the Loki service is not running, your have entered the wrong URL, or ports are blocked by a firewall. The default Loki install uses both ports 3100 for HTTP and 9096 for gRPC.
Data source connected, but no labels received

When connecting to your Loki data source for the first time, you may see the error,

    Data source connected, but no labels received. Verify that Loki and Promtail is configured properly.

Note how it says, Data source connected at the beginning of the warning. Recent versions of Loki, despite having a successful connection, will still indicate that you have no labels because you don't have Promtail sending any data to it yet. Promtail is set up and discussed in the next lesson.
Unable to connect with Loki. Please check the server logs for more details

To view the Grafana server logs,

tail /var/log/grafana/grafana.log

To filter by errors,

tail /var/log/grafana/grafana.log | grep level=error
