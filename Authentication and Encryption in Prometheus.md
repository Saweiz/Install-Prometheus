# <center>Authentication/Encryption in Prometheus</center>

<br>

### Note: 
We are using two nodes: `node01` and `node02`, with user `'nodeusr'` configured on each.

<br>

## 1: Configure Node Exporter service (on all nodes)

This is to be done on all nodes. In our case, it's `node01` and `node02`. 

#### SSH to node01:
```bash
ssh root@node01
```

#### Create directory and config file:
```bash
mkdir /etc/node_exporter
touch /etc/node_exporter/config.yml
    
chmod 700 /etc/node_exporter
chmod 600 /etc/node_exporter/config.yml 
chown -R nodeusr:nodeusr /etc/node_exporter
```
#### Edit node_exporter service:
```bash
vi /etc/systemd/system/node_exporter.service 
```

Change the line `ExecStart=/usr/local/bin/node_exporter` to
```
ExecStart=/usr/local/bin/node_exporter --web.config=/etc/node_exporter/config.yml
```

#### Reload Node exporter service:
```bash
sudo systemctl daemon-reload 
systemctl restart node_exporter
```

## 2: Configure hash password on Nodes (all nodes)

This is to be done on all nodes. In our case, it's `node01` and `node02`. 

#### SSH to node01:
```bash
ssh root@node01
```

#### Install package:
```bash
apt update
apt install apache2-utils -y
```


#### Generate password hash:
```bash
htpasswd -nBC 10 "" | tr -d ':\n'; echo
```
Enter password you want to generate the hash for. We have entered `secret-password`. Save this password as it will be used to configure node credentials on Prometheus server.


#### Edit config file for node_exporter:
```bash
vi /etc/node_exporter/config.yml 
```
Enter following configuration with previously generated `hash password`
```yaml
basic_auth_users:
  prometheus: <hashed-password>
```


#### Reload service:
```bash
systemctl restart node_exporter
```

#### Check if Authorization is working (accessing without credentials):
```bash
curl http://node01:9100/metrics
```
The output would be `Unauthorized`, if authorization is enabled succcessfully.


#### Access Nodes usng credentials:
```bash
curl -u prometheus:secret-password http://node01:9100/metrics
curl -u prometheus:secret-password http://node02:9100/metrics
```

## 3. Configure Prometheus server to use authentication

This is to be done on `Prometheus server`.

#### Edit the Prometheus configuration file:

```bash
vi /etc/prometheus/prometheus.yml
```

Under `- job_name: "nodes"` add below lines:
```bash
basic_auth:
  username: prometheus
  password: secret-password
```

#### Restart prometheus service:
```bash
systemctl restart prometheus
```

## 4. Add TLS Encryption on Nodes (all nodes)

This is to be done on all nodes. In our case, it's `node01` and `node02`. 

#### SSH to node01:
```bash
ssh root@node01
```

#### Generate the certificate and key:
```bash
openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout node_exporter.key -out node_exporter.crt -subj "/C=US/ST=California/L=Oakland/O=MyOrg/CN=localhost" -addext "subjectAltName = DNS:localhost"
```


#### Move the crt and key file under `/etc/node_exporter/` directory:
```bash
mv node_exporter.crt node_exporter.key /etc/node_exporter/
```


#### Change ownership:
```bash
chown nodeusr:nodeusr /etc/node_exporter/node_exporter.key
chown nodeusr:nodeusr /etc/node_exporter/node_exporter.crt
```

#### Edit `/etc/node_exporter/config.yml` file:
```bash
vi /etc/node_exporter/config.yml
```

Add below lines in this file:
```yaml
tls_server_config:
  cert_file: node_exporter.crt
  key_file: node_exporter.key
```


#### Restart Node Exporter service:
```bash
systemctl restart node_exporter
```

You can verify your changes using curl command:
```bash
curl -u prometheus:secret-password -k https://node01:9100/metrics
```

## 5. Add TLS certificate on Prometheus server
Do this on Prometheus server:

#### Copy the certificate from node01 to Prometheus server
```bash
scp root@node01:/etc/node_exporter/node_exporter.crt /etc/prometheus/node_exporter.crt
```


#### Change certificate file ownership:
```bash
chown prometheus.prometheus /etc/prometheus/node_exporter.crt
```


#### Edit /etc/prometheus/prometheus.yml file

```bash
vi /etc/prometheus/prometheus.yml 
```


#### Add below given lines under - job_name: "nodes"

```yaml
    scheme: https
    tls_config:
      ca_file: /etc/prometheus/node_exporter.crt
      insecure_skip_verify: true
```


#### Restart prometheus service
```bash
systemctl restart prometheus
```

Now check the Prmetheus UI Website. Both nodes will be `Up` and accessible.