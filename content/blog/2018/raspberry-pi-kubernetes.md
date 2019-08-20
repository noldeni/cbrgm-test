---
title: "My Raspberry Pi Kubernetes Cluster (with Ansible)"
date: 2018-01-28T13:38:48+01:00
draft: false
tags: [ "Development", "Programming", "Kubernetes", "Cluster", "Raspberry", "Ansible", "Traefik", "Weave"]
categories:
  - "Development"
  - "Orchestration"
  - "Cloud Computing"
description: "Kubernetes was 2017's buzzwords in container orchestration and still is. Therefore, the Raspberry Pi is the perfect opportunity to gain experience in orchestration with Kubernetes!"
---

Kubernetes was 2017's buzzwords in container orchestration and still is. The only problem is that you can't try out Kubernetes so easily, because some resources are needed. There are already some projects like Minikube, which deal with this problem and create a local playground, but such a cluster is something more tangible with real hardware in my opinion. Therefore, the Raspberry Pi is the perfect opportunity to gain experience in orchestration with Kubernetes and was reason enough for me to deal with the topic!

![raspberrypi](/img/blog/2018/raspberry-pi-kubernetes-cluster/raspicluster.gif)

## Why did I set up this cluster?

On several blogs you can find articles of people who built their own clusters at home with the help of the ARM-powered minicomputer. First and foremost, I wanted to gain experience in working with the orchestration of docker containers with Kubernetes. At university we have the possibility to work with Kubernetes in the computer science cloud, but personally I would like to go one step further and set up a cluster from scratch.

At the same time, I wanted to acquire knowledge about Ansible in order to simplify software deployment and administration in the cluster. I also wanted to build a small experimental environment at home where I could test my microservices and various projects.

There are several possibilities to rent the necessary resources online via cloud, but honestly: what's cooler than having your own cluster on real hardware at home on your desk?

## My shopping list

![raspberrypi](/img/blog/2018/raspberry-pi-kubernetes-cluster/shoppinglist.jpg)

As lazy as I am, I placed a large order with Amazon Prime. The total cost of the own Kubernetes Cluster amounts to approx. 295,00 EUR. I already owned a Raspberry Pi at home. Here is the complete list of my installed components:

Cluster Hardware:

-   1x [Stackable Raspberry Pi 3 Case][ec24f821]
-   5x [Stackable Raspberry Pi 3 Intermediate Plate][dd6aa079]
-   1x [Anker PowerPort 6, 6-Port Charging Station][65467ea6]
-   2x [Anker PowerLine Micro USB Cable \[4-Pack\] 0.3m Charging Cable][2aae36ea]
-   6x [0.3m CAT.6 Ethernet Gigabit Lan Network Cable  (RJ45) 1Gbps 250Mhz][875d4f22]
-   5x [SanDisk Ultra 32GB microSDHC, 98 MB/Sek., Class 10, U1, A1, FFP][d5a6a932]
-   5x [Raspberry Pi 3 Model B ARM-Cortex-A53 4x 1,2GHz, 1GB RAM][80216893]


Network Components:

-   1x [NETGEAR WNR2000-200PES N300 WLAN Router (300Mbit/s, 4x LAN-Ports, WPA)][39aaf7e8]
-   1x [NETGEAR GS308-100PES Gigabit Metal Case Unmanaged Switch (8-Port)][95a8ba47]


[dd6aa079]: https://www.amazon.de/gp/product/B00NB1WQZW/ref=as_li_tl?ie=UTF8&tag=cbrgm-21&camp=1638&creative=6742&linkCode=as2&creativeASIN=B00NB1WQZW&linkId=fefa690ca8089d989c4d735d39520be3 "Intermediate Plate"

[80216893]: https://www.amazon.de/gp/product/B01CD5VC92/ref=as_li_tl?ie=UTF8&tag=cbrgm-21&camp=1638&creative=6742&linkCode=as2&creativeASIN=B01CD5VC92&linkId=9f14e1d288c7e6f785cdcff4892cdc26 "Raspberry Pi 3"

[d5a6a932]: https://www.amazon.de/gp/product/B073S8LQSL/ref=as_li_tl?ie=UTF8&tag=cbrgm-21&camp=1638&creative=6742&linkCode=as2&creativeASIN=B073S8LQSL&linkId=7cd57275de7e47da32db132cdac909d7 "Micro SDHC Card"

[875d4f22]: https://www.amazon.de/gp/product/B077PTGDC3/ref=as_li_tl?ie=UTF8&tag=cbrgm-21&camp=1638&creative=6742&linkCode=as2&creativeASIN=B077PTGDC3&linkId=0d9efae6b3a96f1039ef2544cbfdf438 "Netzwork Cable"

[2aae36ea]: https://www.amazon.de/gp/product/B016BEVNK4/ref=as_li_tl?ie=UTF8&tag=cbrgm-21&camp=1638&creative=6742&linkCode=as2&creativeASIN=B016BEVNK4&linkId=bf3f46ea33a55319578faf18f4dd1e42 "Power Micro Charging Cable"

[65467ea6]: https://www.amazon.de/gp/product/B00PTLSH9G/ref=as_li_tl?ie=UTF8&tag=cbrgm-21&camp=1638&creative=6742&linkCode=as2&creativeASIN=B00PTLSH9G&linkId=6a753d9d95c5aaf235b2467ea5eea9fd "Anker 6 Port"

[ec24f821]: https://www.amazon.de/gp/product/B00NB1WPEE/ref=as_li_tl?ie=UTF8&tag=cbrgm-21&camp=1638&creative=6742&linkCode=as2&creativeASIN=B00NB1WPEE&linkId=87e1766259ef3ab7b691043c6f11426d "Case"

[95a8ba47]: https://www.amazon.de/gp/product/B00A33BYRC/ref=as_li_tl?ie=UTF8&tag=cbrgm-21&camp=1638&creative=6742&linkCode=as2&creativeASIN=B00A33BYRC&linkId=8caddc448abb1bc5507d39324f3074b7 "Switch"

[39aaf7e8]: https://www.amazon.de/gp/product/B00S9DKS02/ref=as_li_tl?ie=UTF8&tag=cbrgm-21&camp=1638&creative=6742&linkCode=as2&creativeASIN=B00S9DKS02&linkId=0f75bf9a3ce864bd76de3a2895903f84 "Router"

There are many different ways to build up the cluster. You can also use less Pis (e. g. 3 or more) or just use your home networks router instead of a dedicated router for the cluster. However, as I would like to assign my own IP address range to my cluster (Yes, I know you don't need a router for that alone, a simple DHCP Server would do the job as well) and want to transport everything quickly from A to B, I decided to buy and extra router for the project. I also didn't want to reconfigure my laptop to transfer packets from the private network to the internet via NAT. Of course, you can also use wireless as the Raspberry Pi3 has a WLAN module. This eliminates the switch and cables.

If you want to buy other Ethernet cables, make sure they are flat! On the other hand, it can be difficult to get a clear cable management! I also recommend using SD cards with a memory size of 32GB or more, otherwise you will have to budget if you want to use multiple docker images and the like. Don't save at the wrong end, fast memory is worth it!

## Setting up the Cluster!

I would like to briefly explain how I set up my cluster. There are certainly better and faster approaches and even completely finished Ansible Playbooks and installation scripts to configure the cluster without much effort, but I have gained a lot of experience through my independent configuration.

By the way, I'm not the first one to describe how to set up a Raspberry Pi Kuernetes cluster. If you find stumbling blocks in my instructions or if it is too complicated, I recommend the following articles:

-   [Hypriots Setup for a Raspi K8n cluster][741c9703]
-   [Article written by Roland Huß about his k8n cluster setup][c8651354]
-   [Alex Elli's Serverless Kubernetes Home Article][61d759d9]
-   [Scott Hanselmanns How to Build a Kubernetes Cluster with ARM Raspberry Pi][79e1425d]
- [Grizzly Koala Bear's Article about Kubernetes and Weave Net][c6234213]

All these articles describe other setups and installation procedures. Before I started configuring my cluster, I spent a lot of time dealing with the articles. **Thanks to the authors, you did a great job!**

[c6234213]: https://grizzlykoalabear.com/2017/kubernetes-on-pi/ "Grizzlx Koala"

[c8651354]: https://ro14nd.de/kubernetes-on-raspberry-pi3 "Roland Huss"

[61d759d9]: https://blog.alexellis.io/serverless-kubernetes-on-raspberry-pi/ "AlexEllis"

[79e1425d]: https://www.hanselman.com/blog/HowToBuildAKubernetesClusterWithARMRaspberryPiThenRunNETCoreOnOpenFaas.aspx "Scott"

[741c9703]: https://blog.hypriot.com/post/setup-kubernetes-raspberry-pi-cluster/ "Hypriot"

![raspberrypi](/img/blog/2018/raspberry-pi-kubernetes-cluster/raspi_pic1.jpg)

### Installing the Raspbian Images on the SD Cards

Different images are available for your own Kubernet cluster. HypriotOS, for example, is very well suited for use with docker containers and Kubernetes, but I will use the official Raspbian image here.

You can download the image from the official website. For flashing the image I recommend [Hypriots flash tool][bd1c9f0d], which makes the flash process incredibly easy. You can install it using the following commands:

[bd1c9f0d]: https://github.com/hypriot/flash "Flash Tool"

```bash
curl -O https://raw.githubusercontent.com/hypriot/flash/master/$(uname -s)/flash
chmod +x flash
sudo mv flash /usr/local/bin/flash
```

Download the latest [Raspbian Stretch Lite][c8854057] (without Desktop) Image and store it on your local device.

  [c8854057]: https://www.raspberrypi.org/downloads/raspbian/ "Raspbian"

```bash
curl -L https://downloads.raspberrypi.org/raspbian_lite_latest -o raspbian-lite.zip
```

Now insert the SD card into your laptop and you can write the image to the storage medium. Flash it using `flash raspbian-lite.zip`. During the flash process, the script asks for a device name. With the Sandisk SDHC cards it is normally `mmcblk0`, so make sure that you choose the correct name during the flash process, otherwise you risk overwriting your host system.

The output looks like this (If everything went well ... :-)):

```bash
/usr/bin/unzip                                           
Uncompressing raspbian-lite.zip ...           
Archive:  raspbian-lite.zip                                
inflating:/tmp/2017-11-29-raspbian-stretch-lite.img                                        
Use /tmp/2017-11-29-raspbian-stretch-lite.img                                          
NAME          SIZE TYPE MOUNTPOINT                    
mmcblk0      29,7G disk                       
└─mmcblk0p1  29,7G part                                         
Please pick your device: mmcblk0                                            
Is /dev/mmcblk0 correct? y                   
Unmounting /dev/mmcblk0 ...   
Flashing /tmp/2017-11-29-raspbian-stretch-lite.img to /dev/mmcblk0...                                                      
No pv command found, so no progress available.  
Press CTRL+T when you want to see the current info of dd command.                                                              
1772+0 Datensätze ein         
1772+0 Datensätze aus                         
1858076672 Bytes (1,9 GB, 1,7 GiB) kopiert, 130,392 s, 14,2 MB/s                                                                                                                                                     
Waiting for device...                                        
/dev/mmcblk0:re-reading partition table              
Mounting Disk   
Mounting /dev/mmcblk0p1 to customize...
Unmounting /dev/mmcblk0 ...
Finished.
```

**Pro-Tip**: Activate SSH by putting a file called `ssh` (without extension) on the boot partition of the SD card, so that it is automatically activated! Otherwise you have to enable SSH access via the `raspi-config` command on your Raspberry Pi.

### The Network

The network structure is relatively simple. The individual Raspberry Pis are all wired to the switch, just like the router. My router automatically distributes the IP addresses to the individual nodes in my cluster. I have used a private class B network for this purpose and assigned the following IP addresses statically (so that they don't change their network address constantly when the DHCP lease expires)

Please don't be confused by hostnames. Since Kubernetes is aimed at the term helmsman, I have adapted my nodes in the cluster to the annecdote.

My master is therefore the flagship of my fleet, while the individual nodes represent my normal ships ;-).

-   172.16.1.1 Router
-   172.16.1.2 Laptop
-   172.16.1.10 Masternode (flagship)
-   172.16.1.11 node1 (ship1)
-   172.16.1.12 node2 (ship2)
-   172.16.1.13 node3 (ship3)
-   172.16.1.14 node4 (ship4)

![raspberrypi](/img/blog/2018/raspberry-pi-kubernetes-cluster/raspi_pic2.jpg)

### Setting up the Raspberry Pis!

First of all, flash all SD cards with the Raspbian Lite Image and then insert the cards into the individual devices. From this step on, we could also configure all the devices manually, instead we will use Ansible to help and use a "playbook" to transfer a standard configuration on all the raspberries. That saves us a lot of work and nerves!

This saves us a lot of work and nerves! After all, we are computer scientists (or something like that) and by nature lazy!

In this article I will not go into more detail about Ansible and the many features it offers. That would go beyond the scope of this. In case you don't know Ansible: Ansible is an open source automation tool for orchestration and general configuration and administration of computers. It combines software distribution, ad hoc command execution and configuration management. It manages network computers via SSH, among other things, and does not require any additional software on the system to be managed. Modules use JSON for output and can be written in any programming language. Also the system uses YAML to formulate reusable descriptions of systems.

If you want to learn more about Ansible and how to use it, I recommend the tutorial on the official [Ansible][10420b0a] website.

[10420b0a]: https://docs.ansible.com/ansible/latest/intro_getting_started.html "Ansible"

### Setting up the master node

Connect to your master node via ssh using standard user `pi` and password `raspberry`. Before we can use our configuration via Ansible, we need to set up a few basic things.

#### Change keyboard layout

By default, the English keyboard input is set on the Raspbian Image. If you like, you can change it as follows:

```bash
sudo nano /etc/default/keyboard
XKBLAYOUT=”gb” # find where it says, change to your language code
```

If you are connected via ssh, this step is not necessary because your own keyboard layout is used. However, if the Raspberry Pi is not headless, this step can be useful.

#### Pull the latest software updates, install Git and SSHPass

Next, you should update the installed packages. We also need Git to clone the repository containing the [Ansible Playbooks][95f3366a]. sshpass is needed for Ansible so that the tool can connect to the individual hosts.

[95f3366a]: https://github.com/cbrgm/ansible-pi3cluster-playbooks "Ansible Playbooks"

```bash
# Pull latest updates
sudo apt-get update -y && sudo apt-get upgrade -y
sudo apt-get install git, sshpass
```

Then clone my repository to the Pi user's home directory.

```bash
git clone https://github.com/cbrgm/ansible-pi3cluster-playbooks.git
```

### Install Ansible on master node!

#### Ansible setup

On the masternode we will now install Ansible. Unfortunately, the Raspberry Pi doesn't have up-to-date package sources for Ansible, so we have to compile Ansible ourselves to get the latest version.

However, compiling requires a lot of dependencies like TexLive, which we don't want to have on our master. I recommend to compile the source code on a device other than the Raspberry Pi. Detailed instructions can be found [here][8cbdafb7].

For all those who want to be comfortable: The author of the linked article "arnesorium" has created a debian package of version 2.2. At this point in time 2.4 is the latest version of Ansible, but for our purpose 2.2 also does its service.

In the folder "tools" in my repository there is an installation script that you can start and automatically install Ansible 2.2 on your master.

```bash
./tools/installing-ansible-2-2.sh
```

After that you should be able to use the Ansible commands via the command line.

[8cbdafb7]: https://arnesonium.com/2016/11/installing-ansible-2-2-0-on-a-raspberry-pi/ "arnesonium"

#### Create an Inventory file

Next we have to create a hosts file. In the Hosts file, all our devices are listed in the cluster. You can change the name of the devices as you like, but be aware: The Ansible Playbook will automatically set the hostname of the corresponding Raspberry Pis to the name given in the hostfile.

I also recommend numbering when selecting the names. You can also take fancy names, but it is important for practice purposes that you can quickly get an overview of your cluster.

Here is an example of my host file, which I also saved in the repository under `hosts.example`. Change it as you like. Then copy the file to the path `/etc/ansible/hosts` (The default path for Ansibles hosts lookup)

```bash
[cluster]
flagship ansible_host=172.16.1.10
ship1 ansible_host=172.16.1.11
ship2 ansible_host=172.16.1.12
ship3 ansible_host=172.16.1.13
ship4 ansible_host=172.16.1.14

[master]
flagship ansible_host=172.16.1.10

[nodes]
ship1 ansible_host=172.16.1.11
ship2 ansible_host=172.16.1.12
ship3 ansible_host=172.16.1.13
ship4 ansible_host=172.16.1.14
```

### Basic cluster configuration using an Ansible Playbook

Now that Ansible is installed and the host file is in place, we can start with the setup of the master and the nodes. I have prepared two playbooks for this purpose, which you can find in the repository. Both will roughly do the following things:

**Setup-Playbook**

-   Configure the basic system (Pull updates, set hostname, Disable swapfile, ...)
-   Add Docker and Kubernetes Repositories to packet sources
    -   Install latest Docker version
    -   Install latest Kubernetes version
    -   Install Weave as a Container Network Interface (See below)

**User-Playbook**

-   Create a new power user (sudo group) as standard user for cluster management
-   Add ssh public key of master's user to all node's users `authorized_keys`

#### Deploy the basic configuration to the cluster

To roll out the basic configuration in the cluster, use the following command:

```bash
ansible-playbook setup.yml --extra-vars "hosts=cluster" -u pi --ask-pass --ask-become-pass
```

Afterwards, each individual device in the service is briefly restarted. So don't be surprised if the SSH connection will fail after about a minute. After a few seconds you can connect to the master again via SSH.

#### Setup Superuser for Cluster Management

Now that we have rolled out the basic configuration on all devices in the cluster, we can create a new user for cluster management. Alternatively, you can continue to use the default user of the Raspberry Pi, but I do not recommend this for security reasons.

On the **master** execute the following command:

```bash
ansible-playbook user.yml --extra-vars "username=<username> password=<password> hosts=master" -u pi --ask-pass --ask-become-pass
```

Afterwards you can log in with the newly created user. All further steps will be done with this user. Clone the git repository again or copy the folder from the home directory of the pi user into your new users home directory.

On the master **logged in with your new super user** execute the following command:

```bash
ansible-playbook user.yml --extra-vars "username=<username> password=<password> copy-ssh=true hosts=nodes" -u <master_username> --ask-pass --ask-become-pass
```

On all nodes in our cluster the new user has been created and the public SSH key of our master's user has been copied to the node's users `authorized_keys`, which means that we can now connect via SSH from our master without entering a password.

#### Delete Standard Pi user (Optional for security reasons)

The default user of the Raspberry Pis is no longer needed and can be deleted from all nodes and the master. However, this step is not mandatory if you are the only one who will play around with the cluster.

## Initialize the Kubernetes cluster

On the master we will now initialize the Kubernetes Cluster. For this you can execute the following command:

```bash
kubeadm init
```

Then the join command appears, with which we can integrate the nodes into our Kubernetes cluster. **Be sure to copy this command!**

Then copy the cluster configuration with the following commands:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
echo "export KUBECONFIG=${HOME}/.kube/config" >> ~/.bashrc
source ~/.bashrc
```

### Installing Weave as a Container Network Interface (CNI)

Weave Net creates a virtual network that connects Docker containers deployed across multiple hosts. To application containers, the network established by Weave resembles a giant Ethernet switch, where all containers are connected and can easily access services from one another.

Weave is just one of many possibilities for container networking with Kubernetes, other options are for example Flannel, which works wonderfully on ARM systems. Other ones are for example Calico, romana or Canal.

However, I chose Weave because I had some problems with Flannel. To install Weave in Kubernetes as a plugin, run the following command

```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

For more information about Weave, click on the link to the [Weave Net][33817e91] documentation.

  [33817e91]: https://www.weave.works/docs/net/latest/install/installing-weave/ "Weave Net"

#### What is a CNI? And why do I need that?

Container Network Interface (CNI), is a library definition and a set of tools, under the umbrella of the Cloud Native Computing Foundation project. Kubernetes uses CNI as an interface between network providers and Kubernetes networking and exposes a simple set of interfaces for adding and removing a container from a network.

There is much more to tell about it, but I'll confine myself to the short explanation above.

### Let each node join your cluster

Now let the individual nodes join the cluster. Use the Join command that you received from the console after initializing the cluster and hopefully wrote it down! Execute the command on each node, then use `kubectl get nodes` to check if everything worked. My output looks like this:

```bash
NAME       STATUS    ROLES     AGE       VERSION
flagship   Ready     master    1d        v1.9.2
ship1      Ready     <none>    1d        v1.9.2
ship2      Ready     <none>    1d        v1.9.2
ship3      Ready     <none>    1d        v1.9.2
ship4      Ready     <none>    1d        v1.9.2
```

### Installing Traefik as a Load Balancer

[Traefik][96cc654e] is a Docker-aware reverse proxy that includes its own monitoring dashboard.

  [96cc654e]: https://docs.traefik.io/user-guide/kubernetes/ "Traefik"

```bash
kubectl apply -f https://raw.githubusercontent.com/containous/traefik/master/examples/k8s/traefik-rbac.yaml

kubectl apply -f https://raw.githubusercontent.com/containous/traefik/master/examples/k8s/traefik-ds.yaml
```

Check that Traefik is running using `kubectl --namespace=kube-system get pods`

You should see that after submitting the DaemonSet to Kubernetes it has launched a Pod, and it is now running. It might take a few moments for Kubernetes to pull the Træfik image and start the container, so don't be stressed!

### Deploying a test application

As a test we will deploy a simple nginx web container provided by the Hypriot guys.

```bash
kubectl run hypriot --image=hypriot/rpi-busybox-httpd --replicas=2 --port=80
kubectl expose deployment hypriot --port 80
```

You can check your Pods endpoints by using `kubectl get endpoints hypriot`

```
NAME         ENDPOINTS                   AGE
hypriot      10.42.0.1:80,10.44.0.1:80   1d
```

You can also `curl` the cluster ip and receive the websites content:

```
sailor@flagship:~ $ curl 10.42.0.1
<html>
<head><title>Pi armed with Docker by Hypriot</title>
  <body style="width: 100%; background-color: black;">
    <div id="main" style="margin: 100px auto 0 auto; width: 800px;">
      <img src="pi_armed_with_docker.jpg" alt="pi armed with docker" style="width: 800px">
    </div>
  </body>
</html>
```

Finally create an Ingress to make the pods port accessible from the outside.

```bash
cat > hypriot-ingress.yaml <<EOF
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: hypriot
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: hypriot
          servicePort: 80
EOF
kubectl apply -f hypriot-ingress.yaml
```

Now browse to one of your nodes IP adresses and you should see the website in your browser! configurations! You finally set up your Kubernetes Cluster

![raspberrypi](/img/blog/2018/raspberry-pi-kubernetes-cluster/application.png)

As a test you can try to shut one of the nodes down, you will see the website is still available!

### Reset the experiment!

If you would like to undo the steps from above, you can run the following commands to remove the ingress, service, and deployment:

```bash
kubectl delete ingress hypriot
kubectl delete service hypriot
kubectl delete deployment hypriot
```
## Conclusion and further reading

All in all I am very happy with my setup. I must admit, it took me a while to configure everything and gain a foothold! Nevertheless, I was able to learn a lot about Ansible and now I can finally say: I not only used Kubernetes, I even got it set up.

I am looking forward to testing out my microservice projects from university and private in my small mini cluster on the desk. There will certainly be some more articles about the cluster in the future and I will be delivering status updates from time to time. I'm looking forward to it and I hope you enjoyed reading and maybe even rebuilding it!

cheers  
Chris
