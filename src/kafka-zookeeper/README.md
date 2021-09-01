ELRR Kafka Cluster 
==============
This repo is a clone of the Kafka cluster used to route xAPI within the ADL's TLA reference implementation.  

It contains the container definitions and scripts for standing up up a 3/3 SASL-authenticated Kafka cluster with Docker. Images are slightly modified versions of the Confluent Docker images for both Zookeeper and Kafka, with the only adjustments being to relay Compose variables to the Kafka/Zookeeper config files.  Admittedly, this makes the image build time a bit long as I have to install `gettext`, so a cleaner / faster solution would be welcomed.

## What's in the box
Everything necessary to stand up the Kafka cluster, including:
- `docker-compose.yml`: The Compose file we use to stand up:
  - 3 Kafka Brokers
  - 3 Zookeeper nodes
- **Configuration files:**
  - `.env`: Change the `KAFKA_HOST` to an accessible IP / domain name running this.
  - `tla-topics.txt`: Text file with each TLA topic.  
- **Scripts to maintain the cluster:**
  - `tla-topics.txt`: Text file with the Kafka topics.
  - `export-topics.sh`: Creates the Kafka topics (this is necessary as we disable auto-topics).
  - `list-topics.sh`: Lists all topics available in the cluster.


## Intended use
Intended use of this code is that a user could reference the steps below, including the associated scripts within this repository, to build a kafka-zookeeper cluster using docker containers in a private subnet. The configuration scripts within this environment the codes to make this happen. Developers may leverage their own configuration scripts.

## Capabilities and limitations
The code in this repository leverages Docker images for running majority of the core applications. While the code has been tested on AWS EC2 instances, the code should be able to run on any major cloud platform with parameter tweaks, but has not been verified.
Note that connection to the cluster will be from the bastion host since the databases are in private subnets
Configure AWS security groups to allow appropriate ports and ips.

In the future, the development team will explore running the ELRR suite using Docker Compose and/or on Kubernetes.

### TL;DR Initial Setup
Download private key and ssh into private instance
Setup ssh connection between bastion host and database servers and verify connections
```console
ssh -i ~/.ssh/<key-name.pem> <hostname>@<address>
```

# Install, start and enable firewalld on ubuntu
```console
sudo apt-get install firewalld -y
sudo systemctl start firewalld.service
sudo systemctl enable firewalld.service
sudo systemctl status firewalld.service
```

# Disable RAM Swap
sudo sysctl vm.swappiness=1

Make it persistent
echo 'vm.swappiness=1' | sudo tee --append /etc/sysctl.conf


# Allow connections and test connection from bastion host
```console
sudo firewall-cmd --zone=public --add-port=ssh/tcp --permanent;
sudo firewall-cmd --zone=public --add-port=<kafka-port-1>/tcp --permanent;
sudo firewall-cmd --zone=public --add-port=<kafka-port-2>/tcp --permanent;
sudo firewall-cmd --zone=public --add-port=<kafka-port-3>/tcp --permanent;
sudo firewall-cmd --zone=public --add-port=<Kafka-Advertised-Listeners-Port>/tcp --permanent;
sudo firewall-cmd --reload;
sudo firewall-cmd --state
```
# Security Group Incoming Rules
| IP version  | Type | Protocol | Port range | Source | Description |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| IPv4  | Custom TCP  | TCP  | kafka-port-1  | Internal CIDR  | Allow connection from VPC CIDR for Kafka port  |
| IPv4  | Custom TCP  | TCP  | kafka-port-2  | Internal CIDR  | Allow connection from VPC CIDR for Kafka port  |
| IPv4  | Custom TCP  | TCP  | kafka-port-3  | Internal CIDR  | Allow connection from VPC CIDR for Kafka port  |
| IPv4  | Custom TCP  | TCP  | zookeeper-port-1  | Internal CIDR  | Allow connection from VPC CIDR for Zookeeper port  |
| IPv4  | Custom TCP  | TCP  | zookeeper-port-2  | Internal CIDR  | Allow connection from VPC CIDR for Zookeeper port  |
| IPv4  | Custom TCP  | TCP  | zookeeper-port-3  | Internal CIDR  | Allow connection from VPC CIDR for Zookeeper port  |
| IPv4  | Custom TCP  | TCP  | Kafka-Advertised-Listeners-Port | Internal CIDR  | Allow connection from VPC CIDR for Kafka Advertised listener port  |
| IPv4  | All traffic  | All  | All  | Server Private IP  | Allow all traffic from kafka zookeeper private ip  |

# Install tree tool
```console
sudo apt-get update
sudo apt-get install tree -y
```

## Generate local ssh keys on your OS
Establish your ssh connection with git and add id_rsa.pub to git settings or copy an existing one
1. `Create keys from the cli: ssh-keygen -t rsa -b 4096 -C "name@adress.com"`
1. `Copy the id_rsa.pub into git ssh settings`

## Pull repo from github
1. `git clone git@github.com:US-ELRR/elrr-infrastructure.git`
1. ` cd elrr-infrastructure/src/kafka-zookeeper/`
1. Modify `.env` and set `KAFKA_HOST` to the resolvable network path for your cluster - this can be a domain name or an IP.
1. `sudo ./install-reqs.sh`

Post installaion steps:
```console
sudo usermod -aG docker $USER
sudo setfacl -m user:$USER:rw /var/run/docker.sock
sudo chmod +x /usr/local/bin/docker-compose
sudo systemctl enable docker
sudo systemctl status docker
```
Continue installation...
1. `sudo docker-compose up -d --build`
1. `sudo docker logs -f kafka_1`, then wait until the text stops
1. `sudo ./export-topics.sh`
1. `docker exec kafka_1 kafka-topics --list --zookeeper localhost:12181`
1. `docker exec kafka_2 kafka-topics --list --zookeeper localhost:12181`
1. `docker exec kafka_3 kafka-topics --list --zookeeper localhost:12181`


## Topics
Each topic on the cluster exists for a reason, but we also have topics for testing purposes.  By default, there are 3 topics you can use to test out messages and demo producers / consumers: `test-1`, `test-2`, and `test-3`.  

### `learner-xapi`
All statements sent to the LRS will be pushed into this topic by our LRS proxy Kafka producers.  

### `system-xapi`
Historically used to emit logs sent as xAPI statements, but not currently used.

### `resolve-pending`
Topic reserved for statements that were deemed MOM relevant but require additional resolution.

### `resolved-xapi`
Transactional statements.  These statements have been either resolved or confirmed relevant to the current TLA enclave.

### `authority-xapi`
Authoritative statements.  These statements have influenced a competency decision -- usually assertions.
