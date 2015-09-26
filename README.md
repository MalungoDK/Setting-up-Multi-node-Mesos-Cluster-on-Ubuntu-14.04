# Setting-up-Multi-node-Mesos-Cluster-on-Ubuntu-14.04

I have been working on my final project at the University, and as part of the project, I had to setup a multi-node Mesos cluster to be used as a shared computing infrastructure for distributed applications.
I followed many tutorial online, however, most of them are not well detailed when it comes to setting up the Mesos cluster on a network with a proxy server which was my case. Therefore, I had to add some commands in order to get the infrastructure up and running.

After many months on struggling, I finally managed to get it right. In my infrastructure, I used one master and multiple slaves for the purpose of my project,and here is how.

**Confingure the Master**

**1. Add Mesosphere Archive key**

Because I was working on the network with proxy server, I had to put my proxy details when importing the key. Replace username, password, proxy IP address, and port with the correct values.

sudo  apt-key adv --keyserver-options http-proxy="http://username:password@proxyIPaddress:port" --keyserver hkp://keyserver.ubuntu.com:80 --recv E56151BF

**2. Add Mesosphere Ubuntu repository**

DISTRO=$(lsb_release -is | tr '[:upper:]' '[:lower:]')

CODENAME=$(lsb_release -cs)

echo "deb http://repos.mesosphere.io/${DISTRO} ${CODENAME} main" | sudo tee /etc/apt/sources.list.d/mesosphere.list

**3. Download the package list**

sudo apt-get -y update

**4. Install Mesos and Marathon**
  ****
sudo apt-get -y install mesos marathon

**Note:** It is not a must to reboot the system after installing Mesos and Marathon. However, In my case, I had to reboot the system because the services did not start immediately.

sudo reboot

**Additional Master configuration**

The Mesosphere packaged installed includes an init script that starts the Mesos master, slave and Zookeeper automatically. But I only wanted the master to manage the cluster and not to offer its resources, So I stopped the slave service.

**1. Stop the Slave service**

sudo service mesos-slave stop

**2. Disable the slave service**

echo manual | sudo tee /etc/init/mesos-slave.override

**3. Configure the MasterIP address**

echo XX.XX.XX.XX | sudo tee /etc/mesos-master/ip

Note: replace the xx.xx.xx.xx with the master IP address

**4. Configure the host name**

In order to avoid issues when it comes to resolving names, I decided to use the master IP address as the host name. However, by default the master uses will use the system host name. Again replace xx.xx.xx.xx with the master IP address.

echo XX.XX.XX.XX | sudo tee /etc/mesos-master/hostname

**5. Specify the master zookeeper URL which the Mesos master and marathon will register with.**

echo zk://XX.XX.XX.XX:2181/mesos | sudo tee /etc/mesos/zk

**Note:** replace xx.xx.xx.xx with the master IP address

**6. Set the Cluster name**

echo ClusterName | sudo tee /etc/mesos-master/cluster

**Note:** replace clusterName with your own name.

**7. Restart the services**

sudo service zookeeper restart

sudo service mesos-master restart

sudo service marathon restart

When I got to this point, I was able to access the Mesos UI on port 5050 and Marathon on port 8080 by entering the following URL on my web browser.

**Mesos UI**

http://XX.XX.XX.XX:5050

**Marathon**

http://XX.XX.XX.XX:8080

**Note:** replace xx.xx.xx.xx with the master IP address 

**Configure the Slave**

This process was done on all the slaves in my cluster.

**1. Add Mesosphere Archive key**

Because I was working on the network with proxy server, I had to put my proxy details when importing the key. Replace username, password, proxy IP address, and port with the correct values.

sudo  apt-key adv --keyserver-options http-proxy="http://username:password@proxyIPaddress:port" --keyserver hkp://keyserver.ubuntu.com:80 --recv E56151BF

**2. Add Mesosphere Ubuntu repository**

DISTRO=$(lsb_release -is | tr '[:upper:]' '[:lower:]')

CODENAME=$(lsb_release -cs)

echo "deb http://repos.mesosphere.io/${DISTRO} ${CODENAME} main" | sudo tee /etc/apt/sources.list.d/mesosphere.list

**3. Download the package list**

sudo apt-get -y update

**4. Install Mesos**

sudo apt-get -y install mesos

**Note:** It is not a must to reboot the system after installing Mesos and Marathon. However, In my case, I had to reboot the system because the services did not start immediately.

sudo reboot

**Additional Slave Configuration**

**1. Stop the master service**

 sudo service mesos-master stop

 echo manual | sudo tee /etc/init/mesos-master.override

**2. Stop Zookeeper**

sudo service zookeeper stop

echo manual | sudo tee /etc/init/zookeeper.override

**3. Configure the Slave IP address**

echo XX.XX.XX.XX | sudo tee /etc/mesos-slave/ip

Note: replace the xx.xx.xx.xx with the slave IP address

**4. Configure the host name**

In order to avoid issues when it comes to resolving names, I decided to use the slave IP address as the host name. However, by default the slave uses will use the system host name. Again replace xx.xx.xx.xx with the master IP address.

echo XX.XX.XX.XX | sudo tee /etc/mesos-slave/hostname

**5. Connect the slave to the Zookeeper**

echo zk://XX.XX.XX.XX:2181/mesos | sudo tee /etc/mesos/zk

**Note:** replace xx.xx.xx.xx with the Master IP address

**6. Restart the slave service**

sudo service mesos-slave restart

**Note:** I had problems after restarting the slave service, when I run **netstat -ntv ** I could see that there was no process running on port 5051. However, the problem was resolved by rebooting the system.

sudo reboot

After rebooting the system, the slave appeared on the Mesos master UI.

