# MasterMind DevOps Challenge

## Goals

1. write and object/driver in KUBERNETES that can launch an unique instance with a PostgreeSQL database with the next config:
	a. User: postgres
	b. Password: Aw3s0m3
	c. Databasename: WORKSHOP

2. Once the PostgreSQL DB is working, ensure that each time the PostgreSQL POD is down, an email will be sent to alert this

3. Bonus Points: automate the PODto create preconfigured DBs and Tables 

4. My personal note: I'll be running this on AWS EC2 instances with an Ubuntu 18.04 LTS, so this Readme will contain aditional troubleshooting of 
   what to do in order to get this Challengue done in an AWS instance wherever you find the $$$ it notes an aditional troubleshooting
   $$$ IMPORTANT in EC2 you will need minimum a t2.medium as minikube needs an instances with 2 CPU's to work, Yes this will cost you 	 0.046 USD/hour on US-EAST-2

5. Also this is the first time I try using kubernetes, I made several tries to create the message alerts with Searchlight, Datadog and alertmanager from Promotheus

### Setup Prerequisities

#### Docker:

* Download links for Docker for Windows: https://www.docker.com/docker-windows or https://hub.docker.com/
* Download links for Docker for Linux: https://docs.docker.com/install/linux/docker-ce/ubuntu/
* Download links for Docker for Mac: https://docs.docker.com/docker-for-mac/install/

Command to test: `docker version` 
2nd command to test: docker run hello-world

#### Virtualbox

$$$ If you are inside an AWS instance runing linux or Ubuntu 18.04 you wont need a VirtualMachine, I've used another method
	--vm-driver=none, se more info at Setup Minikube

* Download link for Windows and MAC: https://www.virtualbox.org/

For Linux via apt:
	wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
	wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | sudo apt-key add -
	sudo add-apt-repository "deb http://download.virtualbox.org/virtualbox/debian bionic contrib"
	sudo apt update
	sudo apt install virtualbox-6.0

Command to test: `virtualbox`

### Setup Kubectl:
* Download link: https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-binary-via-curl

For Linux via curl:
	curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
	chmod +x ./kubectl
	sudo mv ./kubectl /usr/local/bin/kubectl

Command to test: `kubectl version`


### Setup Minikube:
* General Download Instructions: https://kubernetes.io/docs/tasks/tools/install-minikube/
* Download link: https://github.com/kubernetes/minikube/releases
	curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube \
	&& sudo mv minikube /usr/local/bin/

Command to test: `minikube version`

$$$ If you are inside an AWS instance running linux or Ubuntu 18.04 you should use --vm-driver=none to rune minikube in the host and not in a VM as you dont have access to configure BIOS on AWS:

	sudo -i
	minikube start --vm-driver=none

(In case you already tryied starting minikube before the command above but failed use:
	minikube delete -p minikube
	and run the command minikube start --vm-driver=none)

## NOW THE CHALLENGUE

### PART 1:
	write and object/driver in KUBERNETES that can launch an unique instance with a PostgreeSQL database with the next config:
	a. User: postgres
	b. Password: Aw3s0m3
	c. Databasename: WORKSHOP

First Create a yaml file that allows you to launch the instance with the preconfigured setting

********postgres.yaml********
---
 apiVersion: v1
 kind: Pod
 metadata:
  name: mastermind
  labels:
    type: local
    app: postgres

 spec:
  containers:
   - name: postgresql 
     image: postgres
     env:
      - name: POSTGRES_USER
        value: "postgres"
      - name: POSTGRES_PASSWORD
        value: "Aw3s0m3"
      - name: POSTGRES_DB
        value: WORKSHOP

   - name: adminer
     image: adminer
     ports:
      - containerPort: 8080 

******************************

	Once Created use in sudo -i mode 	 
		sudo -i
		kubectl create -f name.yml

Log In to a Shell in your new pod and validate that the WORKSHOP DB is active

	kubectl exec -it mastermind -c postgresql -- /bin/bash
	psql WORKSHOP postgres

You will now see "WORKSHOP=#" in the prompt meanning that you logged in succesfully to your Database, it will not ask you for a password as when the pod was created the password was configured as an enviromental Variable

You can validate in any log table containing the USENAME column that you username assigned to your Postgres DB is correct, let's try using the table pg_stat_activity that your username is correct as it should be shown in the column usename with the following query

	SELECT pid, usename, wait_event_type, wait_event FROM pg_stat_activity WHERE wait_event is NOT NULL;
 pid | usename  | wait_event_type |     wait_event
-----+----------+-----------------+---------------------
  65 | postgres | Activity        | LogicalLauncherMain
  63 |          | Activity        | AutoVacuumMain
  61 |          | Activity        | BgWriterMain
  60 |          | Activity        | CheckpointerMain
  62 |          | Activity        | WalWriterMain
(5 rows)

If correct you should see something like above and the usename postgres showing up

### PART 2:
	Searchlight Will help us in this part as we just need to make a few configurations to get email alerts
	
	firts we'll install searchlight in kubernetes by runing next commands	
		curl -fsSL https://raw.githubusercontent.com/appscode/searchlight/8.0.0-rc.0/hack/deploy/searchlight.sh | bash

	Validate the installation with
		kubectl get pods -n kube-system | grep searchlight-operator
retuns:	searchlight-operator-55b6f78d94-nr7v9   3/3     Running            0          88s
	
	first we need to create files for a secret containing the configuration files to send our email via SMTP protocol via gmail
		echo -n 'smtp.gmail.com' > SMTP_HOST
		echo -n '587' > SMTP_PORT
		echo -n 'true' > SMTP_INSECURE_SKIP_VERIFY
		echo -n 'your_fullemail@gmail.com' > SMTP_USERNAME
		echo -n 'your-complex-pwd' > SMTP_PASSWORD
		echo -n 'yourfullemail@gmail.com' > SMTP_FROM

	Then to create the secret run next command
			kubectl create secret generic notifier-config -n default \
		    --from-file=./SMTP_HOST \
		    --from-file=./SMTP_PORT \
		    --from-file=./SMTP_INSECURE_SKIP_VERIFY \
		    --from-file=./SMTP_USERNAME \
		    --from-file=./SMTP_PASSWORD \
		    --from-file=./SMTP_FROM

	secret "notifier-config" created *this will be the reply from system


	second we'll crete the Searchlight PodAlert with a Yaml file and run the following command
		kubectl create -f podalertmail.yaml

	The file cointains the following code below and will run a command inside the masterming (postgres pod) to validate if the pod is available, if not it will return a Critical state and then email a notification
	********podalertmail.yaml*********
---
apiVersion: monitoring.appscode.com/v1alpha1
kind: PodAlert
metadata:
  name: pod-alert-mail
  namespace: default
spec:
  check: pod-exec
  podName: mastermind
  vars:  
    argv: ls -l -a /usr
  checkInterval: 30s
  alertInterval: 2m
  notifierSecretName: notifier-config
  receivers:
  - notifier: SMTP
    state: Critical
    to: ["jimenecig@gmail.com"]

	******************************************

### PART 3:
	Add in the yaml file preconfigured tables and databases

	If we need to add preconfigured tables into our database or configure new databases we can take again our first yaml script and add args inside the container, this args will be executed in the same order we add them so, let's looks at the yaml below

	kubectl create -f postgreswithtables.yaml
	********postgreswithtables.yaml********
---
 apiVersion: v1
 kind: Pod
 metadata:
  name: mastermind-plus-tables-dbs
  labels:
    type: local
    app: postgres

 spec:
  containers:
   - name: postgresql 
     image: postgres
     env:
      - name: POSTGRES_USER
        value: "postgres"
      - name: POSTGRES_PASSWORD
        value: "Aw3s0m3"
      - name: POSTGRES_DB
        value: WORKSHOP
     argv:
      - kubectl exec -it mastermind -c postgresql -- /bin/bash
	  - psql
      - CREATE DATABASE dbmastermind1
      - CREATE DATABASE dbmastermind2
      - \c dbmastermind1
      - CREATE TABLE dbmmoneteam (	team_id serial PRIMARY KEY, teamlead varchar (50) NOT NULL,	teamcolor varchar (25) NOT NULL, location varchar(25) check (location in ('north', 'south', 'west', 'east', 'northeast', southeast', 'southwest', 'northwest')), INS_TMP date); 
      - \c dbmastermind2
      - CREATE TABLE dbmmtwoteam (	team_id serial PRIMARY KEY, teamlead varchar (50) NOT NULL,	teamcolor varchar (25) NOT NULL, location varchar(25) check (location in ('north', 'south', 'west', 'east', 'northeast', southeast', 'southwest', 'northwest')), INS_TMP date);
   - name: adminer
     image: adminer
     ports:
      - containerPort: 8080 

******************************

