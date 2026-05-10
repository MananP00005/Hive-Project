# Graylog configuration guide 

- Create a VM instance on AWS / GCE (any  other cloud computing platform) with allocation of static _IP address_ and specified hardware requirements.

- __*Very Important Note*__ (Your machine should allow SSH traffic on specified ports like 9000, 9200) OR else you can configure the machine such that it allows all traffic on all ports if you want to change the port configs.

- You can modify the traffic policies as per security requirement in your organization !!!

 
# **__Graylog Installation and Configuration__**

With our backend storage in place, we now need a tool in place that will be responsible for sending our logs to the Wazuh-Indexer (Read More). Graylog is the perfect tool for this. Graylog will receive logs from our Wazuh-Manager, network devices, or any service that has a syslog forwarding option (many 3rd parties are beginning to offer this, but I always recommend pulling data via API if possible :) )..

Perhaps we want to ingest AWS logs, firewall logs, and endpoint logs. Within Graylog we can instruct our node to accept data via inputs. This opens a port that Graylog listens on to receive our logs. Even better is that inputs of various types are supported, such as:
* AWS Cloudtrail
* Beats
* Gelf
* Syslog
* Raw/Plaintext
* And MUCH MORE

### __[Reference (Important !!!!!)](https://go2docs.graylog.org/5-2/home.htm)__



#  **__Features of Graylog__**
##  1. Index Management
Indexes are the building blocks to how Elastic/OpenSearch store data. Poor performance and disk capacity limitations can occur if you are not ensuring healthy index retention periods. For example, one index may hold data from 6 months ago. If I do not delete that index, that data will still reside on disk which could impact the ability to write current date/future logs.

![](../images/graylog-3.png)

Thus, we need a mechanism in place to roll off old data that we no longer care about storing to make room for new data. Graylog allows us to do exactly that!

##  2. Log Normalization
Normalizing our data is a must. We need to ensure common data fields that we receive from our logs (no matter the source) are universally mapped so we can create dashboard and alerting standards that apply to all log types. Your future self will love you!

For example, our firewall writes the source IP that triggered a connection to source_ip_ipv4 , and our Sysmon events that we collect from our endpoints stores the source IP within the data_win_eventdata_sourceIp field. Because these fields contain the same metadata (source IP), it is beneficial for us to write these values to a standard field such as src_ip . Now our SOC team can quickly search for a source IP address within one query, rather than two. Dashboards and alerting are now faster to configure because we are only having to recreate the same dashboards/alerting once rather than having to create new ones as new log sources begin to be ingested.

##  3. API Enrichment
Automating our ingested logs with threat feed analysis is a crucial piece to any SIEM stack. We can use Graylog to interact with MISP, Virustotal, etc. to detect any known malicious IPs, domains, file hashes, etc. that are appearing within our collected logs.

* API data
![](../images/graylog-4.png)
 
* API pipeline to do the task automatically
![](../images/graylog-5.png)


##  4. No Data Loss
Unfortunately, failures to our backend storage can happen. Thankfully Graylog can detect when the backend is in an unhealthy state and write logs to disk until the backend is back online. We now have time to get the backend back in a healthy state without fear of losing logs!


### Operating systems

 - Ubuntu 20.04 LTS
 - Debian 11
 - RHEL 8
 - Fedora 35

 ---


#  **__Step-by-Step Installation__**
Let’s now install Graylog onto our Debian 11 machine!

### __**VERY IMPORTANT : HERE I HAVE USED DEBIAN BASED "OS"  IF YOU ARE ON OTHER "OS" THEN PLEASE REFER THE LINK GIVEN BELOW**__

## __[Installation Reference](https://go2docs.graylog.org/5-0/downloading_and_installing_graylog/installing_graylog.html?tocpath=Downloading%20and%20Installing%20Graylog%7CInstalling%20Graylog%7C_____0)__

# STEPS

---
## INSTALL "MONGODB"
Graylog 5.0 is compatible with MongoDB 5.x-6.x.

MongoDB recommends you disable Transparent Huge Pages: Disable Transparent Huge Pages (THP). (To install MongoDB on Debian, the official MongoDB documentation provides a helpful tutorial.)

The official MongoDB repository provides the most up-to-date version and is the recommended way of installing MongoDB:

#### 1. Install the cryptographic libraries required for the repository keys.
```
$  sudo apt-get install gnupg
```

#### 2. Import the key.
```
$  wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
```

#### 3. Add the Debian repo to the APT list.(For Debian 11)
```
$  echo "deb http://repo.mongodb.org/apt/debian bullseye/mongodb-org/6.0 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
```

#### 4. Update repository package.
```
$  sudo apt-get update
```

#### 5. Install the latest stable version of MongoDB.
```
$  sudo apt-get install -y mongodb-org
```

#### 6. The final step is to enable MongoDB during the operating system’s start up.
```
$  sudo systemctl daemon-reload
$  sudo systemctl enable mongod.service
$  sudo systemctl restart mongod.service
$  sudo systemctl --type=service --state=active | grep mongod
```
---

## INSTALL "OPENSEARCH"
If you are using OpenSearch as your data node, then follow the steps below to install OpenSearch 2.5.

The recommended method of installation is to follow the user documentation provided by the OpenSearch service.

#### 1. Import the public GPG key. This key is used to verify that the APT repository is signed.
```
$  curl -o- https://artifacts.opensearch.org/publickeys/opensearch.pgp | sudo apt-key add -
```

#### 2. Create an APT repository for OpenSearch.
```
$  echo "deb https://artifacts.opensearch.org/releases/bundle/opensearch/2.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/opensearch-2.x.list
```

#### 3. Verify that the repository was created successfully.
```
$  sudo apt-get update
```

#### 4. With the repository information added, list all available versions of OpenSearch:
```
$  sudo apt list -a opensearch
```

#### 5. Choose the version of OpenSearch you want to install. (Unless otherwise indicated, the latest available version of OpenSearch is installed.)
```
$  sudo apt-get install opensearch
```

#### 6. Begin by opening the yml file.
```
$  sudo nano /etc/opensearch/opensearch.yml
```

#### 7. Update the following fields for a minimum unsecured running state (single node).
```
cluster.name: graylog
node.name: ${HOSTNAME}
path.data: /var/lib/opensearch
path.logs: /var/log/opensearch
discovery.type: single-node
network.host: 0.0.0.0
action.auto_create_index: false
plugins.security.disabled: true
indices.query.bool.max_clause_count: 32768
```

#### 8. Enable JVM options.
```
$  sudo nano /etc/opensearch/jvm.options
```

#### 9. Now, update the Xms and Xmx settings with half of the installed system memory, like shown in the example below.
```
## JVM configuration
################################################################
## IMPORTANT: JVM heap size
################################################################
##
## You should always set the min and max JVM heap
## size to the same value. For example, to set
## the heap to 4 GB, set:
##
## -Xms4g
## -Xmx4g
##
## See https://opensearch.org/docs/opensearch/install/important-settings/
## for more information
##
################################################################
# Xms represents the initial size of total heap space
# Xmx represents the maximum size of total heap space
-Xms1g
-Xmx1g
```

#### 10. Finally, enable the system service.
```
$  sudo systemctl daemon-reload
$  sudo systemctl enable opensearch.service
$  sudo systemctl start opensearch.service
```

## search https://YOUR_SERVER_IP:9200 & you will get the results similar to the following.

![](../images/graylog-1.png)
---

## INSTALL "GRAYLOG"

#### 1. Now install the Graylog repository configuration and Graylog Open itself with the following commands.
```
$  wget https://packages.graylog2.org/repo/packages/graylog-5.0-repository_latest.deb
$  sudo dpkg -i graylog-5.0-repository_latest.deb
$  sudo apt-get update && sudo apt-get install graylog-server
```

#### 2. Add RootCA to Keystore if using HTTPS for Wazuh-Indexer
```
$  mkdir /etc/graylog/server/certs
$  cp -a /usr/lib/jvm/java-11-openjdk-amd64/lib/security/cacerts /etc/graylog/server/certs/cacerts
$  keytool -importcert -keystore /etc/graylog/server/certs/cacerts -storepass changeit -alias root_ca -file /etc/graylog/server/certs/rootCA.crt
```

#### 3. Change Default Java Options
```
$  nano /etc/default/graylog-server
```

#### Add the below:
```
GRAYLOG_SERVER_JAVA_OPTS="$GRAYLOG_SERVER_JAVA_OPTS -Dlog4j2.formatMsgNoLookups=true -Djavax.net.ssl.trustStore=/etc/graylog/server/certs/cacerts -Djavax.net.ssl.trustStorePassword=changeit"
```

### Read the instructions within the configurations file and edit as needed, located at /etc/graylog/server/server.conf.

#### 4. To create your password_secret, run the following command.
```
< /dev/urandom tr -dc A-Z-a-z-0-9 | head -c${1:-96};echo;
```

#### 5. To generate a root_password_sha2.
```
echo -n "Enter Password: " && head -1 </dev/stdin | tr -d '\n' | sha256sum | cut -d" " -f1
```

#### 6. Configure the Connection to your Wazuh-Indexer:
```
$ nano /etc/graylog/server/server.conf
```

#### Add the below:
```
elasticsearch_hosts = https://user:pass@wazuh-indexerhostname:9200
```

#### 7. The last step is to enable Graylog during the operating system’s start up and verify it is running.
```
$  sudo systemctl daemon-reload
$  sudo systemctl enable graylog-server
$  sudo systemctl start graylog-server
$  sudo systemctl --type=service --state=active | grep graylog
```
## Note : 
- To be able to connect to Graylog, you should set http_bind_address to the public host name or a public IP address for the machine with which you can connect.

- Then just search the URL => http://YOUR_SERVER_IP:9000 and you will be able to see the graylog webpage.

![](../images/graylog-2.png)
---

#__After you completed these step now we have to connect Graylog with wazuh server__
## __[Connect Graylog and Wazuh](../Graylog/graylog-wazuh-configuration.md)__

