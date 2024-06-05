# jenkins_with_anisible_in_VM

## DevOps CICD pipeline with Ansible [Jenkins freestyle] 

_This project will show how we can use Ansible in Jenkins Pipeline to Automate on multiple servers. [For beginner]_

> Master's VM Specification: <br /> Cores : 2 <br /> RAM : 4Gb <br /> OS : Rocky Linux 9<br /><br />
> Nodes's VM Specification: <br /> Cores : 1 <br /> RAM : 2Gb <br /> OS : Rocky Linux 9

In This Project we need at least two VMs one will act as Master which we will run Jenkins and Ansible. And other will act as Node, it will run Docker containers. [you can use more if you like]

Cause we are using VM and using it for Study we can Turn off the `Firewalld and Selinux service`.<br />
`# For all VMs`

    sudo systemctl stop firewalld && sudo systemctl disable firewalld     # to turn off the firewall
    sudo vi /etc/selinux/config
        SELINUX=disabled                                                  # we need to disable the selinux
    sudo setenforce 0
    echo 0 > /sys/fs/selinux/enforce                                      # so we dont have to restart the system 

our Master VM will communicate with Nodes VM. To make communication passwordless we have to generate a key by using SSH service.

Collect all IP addresses of Node VMs 
> you can see you VM's IP address:<br />
> hostname -I | awk '{print $1}' or ip a

    sudo ssh-keygen
    ssh-copy-id -i /root/.ssh/id_rsa.pub root@'IP_address_of_Node':/root/.ssh/
    
    # to conformation of passwordless communicatioin
    ssh root@'IP_address_of_Node'

### **Step 1 :** Installing Ansible, Jenkins, Docker and Git on Master VM

`**This Installation only be done on Master VM.**`

**Ansible Installation :**

    sudo dnf install epel-release python3 pip
    sudo dnf install ansible
    sudo pip install ansible

We have to edit the hosts file of Ansible to keep track of all nodes.

    sudo vi /etc/ansible/hosts

> update it on last line of the file.<br />
> [`server_name`]<br />
> #add only on Nodes where the docker contaners will run<br />
> `VM's IP address`

you can also use provided hosts file for reference.

Now we have to edit ansible.cfg file 

    sudo vi /etc/ansible/ansible.cfg

Append this on last line of the file: 

    [defaults]
    
    #inventory = /root/anitest.yml
    host_key_checking = False
    #deprecation_warning = False
    #remote_user = root

Thats it for Ansible Installation!!!

**Jenkins Installation :** 

    sudo mkdir /jenkins
    sudo wget https://updates.jenkins.io/download/war/2.459/jenkins.war -P /jenkins # [must be latest]

You need Java openjdk to run the jenkins file. [must be latest]

    sudo yum -y install java-21-openjdk-demo.x86_64

Now run the file using command 

    sudo java -jar /jenkins/jenkins.war

When the above command execution complites it will show you CODE for Jenkins copy it and save it in text file. 
<br /><br />![jenkins-code](/../main/Pics/jenkins-code.png) 

Now open the VM's Browser and search :-
> http://localhost:8080 <br /> #Else you can search it on your Device on which the VM is running and search <br /> http://'VM's IP address':8080

It will ask you for the CODE that we just copyed, paste it.
<br /><br />![jenkins-askscode](/../main/Pics/jenkins-askscode.png)<br /><br />

After that it will for Plugin for Jenkins, Just select suggested plugins it will start the installation. Then you will be ask for signup on Jenkins.<br />
Next you will be ask for Login which you just signup for Jenkins. It will take you to Blank Dashboard.

<br /><br />![jenkins-dashboard](/../main/Pics/jenkins-dashboard.png)

> [!Caution]
> Now, open new terminal and don't close or stop the terminal where the Jenkins.war file is running.

**Docker Installation :**

    sudo dnf check-update    #update the system it need 
    sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    sudo dnf install docker-ce docker-ce-cli containerd.io

To start the docker service, 

    sudo systemctl start docker
    sudo systemctl status docker
    sudo systemctl enable docker    #make sure the service is 'active' after system is restarted/reboot 

You need to have Docker Account to Create your own docker images:-

> [!Tip]
> How to create Docker Account link : https://docs.docker.com/docker-id/

How to login in VM using terminal:

    sudo docker login --username "your_email_address"    #Enter the password of docker account

**Git Installation :**

    sudo yum install git 
    sudo git config --global user.email "email_address"
    sudo git config --global user.name "username"

### **Step 2 :** Create a repositorie in GitHub

> how to create new repository
> https://docs.github.com/en/repositories/creating-and-managing-repositories/quickstart-for-repositories

We have to Create two files in that Repo. name 'Dockerfile' and 'index.html', For example you can use files which is avilable in this Repo.<br />
'Dockerfile' is need for building the images and 'index.html' be our website file.

We need this Repo. in our VM, make directory in '/' name 'git_files' 

    sudo mkdir /git_files
    cd /git_files
    git clone 'URL of your git repo.'

### **Step 3 :**  Installing Docker on Nodes VM

Using Ansible Playbook we can install Docekr on the Node. Playbook is avilable in this Repo.<br /> 
Just copy all `yml` files here > /etc/ansible/ 

    # Running the playbook
    sudo ansible-playbook docker-install.yml

### **Step 4 :**  Creating Freestyle Pipeline

Goto Jenkins DashBoard in Browser, Click on 'New item' to create First step of pipeline.

> `I run this pipeline as root user thats why i didn't use 'sudo' command, if you are not root user add 'sudo' in commands for running ansible playbook`

<br />Select Freestyle Project, Name the project 'git_pull'. <br />Then do the same as shown in below figures:-

<br /><br />![jenkins-git_pull-1](/../main/Pics/jenkins-git_pull-1.png)
<br /><br />![jenkins-git_pull-2](/../main/Pics/jenkins-git_pull-2.png)
<br /><br />![jenkins-git_pull-3](/../main/Pics/jenkins-git_pull-3.png)

Save it and Go back to DashBoard and click on New item for Second step of pipline.
<br />Select Freestyle Project, Name the project 'docker_build'. <br />Then do the same as shown in below figures:-

<br /><br />![jenkins-docker_build-1](/../main/Pics/jenkins-docker_build-1.png)
<br /><br />![jenkins-docker_build-2](/../main/Pics/jenkins-docker_build-2.png)

Save it and Go back to DashBoard and click on New item for Third and Final step of pipline.
<br />Select Freestyle Project, Name the project 'docker_deploy'. <br />Then do the same as shown in below figures:-

<br /><br />![jenkins-docker_deploy-1](/../main/Pics/jenkins-docker_deploy-1.png)
<br /><br />![jenkins-docker_deploy-2](/../main/Pics/jenkins-docker_deploy-2.png)

### **Step 5 :** Run the Pipeline!!!

Final step... goto dashboard and click on play button of git_pull. 

<br /><br />![jenkins-dashboard](/../main/Pics/jenkins-dashboard-1.png)

And see the Build Executor Status if its complite or not.

> [!NOTE]
> you might need to refresh the page to check the every step status. [green or red]

After Build is complite goto VM's Browser and search

> http://localhost:6400

If you see your web page the pipeline was created Susseccfully!!!

it might look like this if you use the provided index.html
<br /><br />![jenkins-webpage](/../main/Pics/jenkins-webpage.png)

<br /><br /><br /><br /><br /><br /><br />_If there is error in pipeline you can check the error by clicking on the name which the step is failed and cliking on number of build History -> see console output_
