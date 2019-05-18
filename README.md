# MasterMindChallenge
IaaS Kubernetes DevOps for DigitalOnUs Challenge 

# Getting Started with Minikube

## Goals

1. write and object/driver in KUBERNETES that can launch an unique instance with a PostgreeSQL database with the next config:
	a. User: postgres
	b. Password: Aw3s0m3
	c. Databasename: WORKSHOP

2. Once the PostgreSQL DB is working, ensure that each time the PostgreSQL POD is down, an email will be sent to alert this

3. Bonus Points: automate the PODto create preconfigured DBs and Tables 

4. My personal note: I'll be running this on AWS EC2 instances with an Ubuntu 18.04 LTS, so this Readme will contain aditional troubleshooting of 
   what to do in order to get this Challengue done in an AWS instance wherever you find the $$$ it notes an aditional troubleshooting
   IMPORTANT in EC2 you will need minimum a t2.medium as minikube needs an instances with 2 CPU's to work, Yes this will cost you 0.046 USD/hour on US-EAST-2

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

(In case you already tryied starting minikube before the command above but failed use: minikube delete -p minikube and run the command above again)
	 

#NOW THE CHALLENGUE

  PART 1: write and object/driver in KUBERNETES that can launch an unique instance with a PostgreeSQL database with the next config:
	a. User: postgres
	b. Password: Aw3s0m3
	c. Databasename: WORKSHOP
