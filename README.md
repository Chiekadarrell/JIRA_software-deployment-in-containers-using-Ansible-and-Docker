# JIRA_software-deployment-in-containers-using-Ansible-and-Docker
![e194738a-a1d4-4932-b5b2-7049dde1007c](https://github.com/user-attachments/assets/fa08b432-3def-4197-bfab-ee6f33b7cf23)

**1.** Launch 3 Amazon Linux 2 instances and name them **“Ansible_Control”**, **“Worker_Node_1”** and **“Worker_Node_2”**. (The security group of Ansible_Control is configured to allow just SSH requests from a desired IP (e.g., my IP) address. The security group of the Worker_Node_1 and Worker_Node_2 is configured to allow SSH traffic from the Ansible_Control and SSH traffic from a desired IP (e.g., my IP).

![image](https://github.com/user-attachments/assets/8eb3457d-36ff-4010-917e-afd96b8ddee6)


## Ansible security group inbound rule
![image](https://github.com/user-attachments/assets/4612ec49-7fd5-45cc-94d4-de949cfa2fe6)



## Security Group inbound rules of Worker Nodes
![image](https://github.com/user-attachments/assets/0ec9d426-1d8a-4ecc-939c-4123e6c7add4)



**2.** Login to 3 three instances using Git Bash

![image](https://github.com/user-attachments/assets/c78952d6-1712-4db9-91b9-182280b251c2)


To determine which Git Bash belongs to which node, rename the servers.
- In each server, Switch as a root user
  ```sh
  sudo su –

- Get into the **“hostname”** file in the /etc/ directory then edit the hostname of each server
  ```sh
  nano /etc/hostname

![image](https://github.com/user-attachments/assets/483ed208-458b-4459-8acf-dfd1c7b146bb)



- Save the changes and reboot the servers
  ```sh
  reboot

- Log into the instances again and you will realize that you can identify which Git bash belongs to which server

![image](https://github.com/user-attachments/assets/39688edf-41b7-44f4-b26d-b9883681c79a)




**3.** For all 3 servers create a common user called “ansible”, set a common password, and allow for “Password authentication”  using the following commands for all 3 servers.
  ```sh
  sudo su - # Login as Root User
  useradd ansible
  passwd ansible # pass a password
  sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config # change password authentication permission to "yes"
  ```

- Navigate to the /etc/ssh/sshd_config path, and uncomment **"PermitRootLogin"** in the sshd_config file.
  ```sh
  nano /etc/ssh/sshd_config
  ```
- Restart the sshd service.
  ```sh
  sudo systemctl restart sshd
  ```
  
**4.** Add ansible to the “sudoers” group in each server.
- Navigate to the sudoers group file via the following command.
  ```sh
  nano /etc/sudoers
  ```

- Add **“ansible”** user in each of the following points in the file.

![image-20240717-200615](https://github.com/user-attachments/assets/15f10820-ad74-41a2-b0da-b6c17d4f4241)

- In the **“Ansible_Control”** Node switch to the created “ansible” user.
  ```sh
  sudo su ansible
  ```

- Try reaching the worker nodes one after the other from the Control Node. A password will be required.
  ```sh
  ssh ansible@private_ip
  ```

You will have something like the above picture

![image-20240717-201024](https://github.com/user-attachments/assets/aeb53d25-35d4-433a-8ab1-43581e9ed07d)

**5.** Create a Keypair in the Control node and copy it in the Worker Node so that it won't require a password when the Control node “SSH” into worker nodes. This will facilitate the configuration of the worker nodes or running the playbook via the control node since it won’t be hindered or blocked by asking for a password which we might not be available at every moment to pass that in.

- While in the control Node, ensure you are at the home directory of the ansible user or run the following lines to do so.
  ```sh
  sudo su ansible 
  cd ~
  ```

- Generate key pair and give required permissions to the generated keypair file(.ssh) so it can be copied to the worker nodes. After the code generates a key-pair, you can hit **“Enter”** till the end.
  ```sh
  ssh-keygen -t rsa # generate keypair
  sudo chmod 700 /home/ansible/.ssh # gives permission to .ssh file to copy it to worker_node
  ```

![image-20240717-201549](https://github.com/user-attachments/assets/a8ff4009-a5b7-426e-870c-aa2b3a95183c)

- Copy Keypair to each worker node.
  ```sh
  ssh-copy-id ansible@private_ip
  ```

![image-20240717-201731](https://github.com/user-attachments/assets/e7d09fc4-fb1c-4369-ac17-c1249ffe9173)

- Once done, try to “SSH” again to your worker nodes and you won’t be prompted to pass a password.

![image-20240717-201823](https://github.com/user-attachments/assets/d348adb7-e97e-4db7-ab4c-823812bd6029)

**6.** Installing Ansible and Docker packages.

- Install the ansible package in the Control Node.
  ```sh
  sudo amazon-linux-extras install ansible2 -y # install ansible in control node
  ansible --version # check if ansible is installed
  ```

- SSH into each worker node through the Control and install the Docker engine.
  ```sh
  sudo amazon-linux-extras install docker -y # install docker
  sudo service docker start # start docker
  sudo systemctl enable docker # enable docker
  sudo systemctl status docker # ensure docker runs
  sudo docker run hello-world # verify if docker is install
  ```

**7.** Update the Inventory file in Control Host so that Ansible knows identified the nodes it communicates with when the playbook is run

- In the home directory of the “ansible” user, run this code.
  ```sh
  cd /etc/ansible/
  sudo nano hosts
  ```

- At example 2 section uncomment **"[webservers]"** then pass the private Ip addresses of the Worker nodes beneath.

![image-20240717-202404](https://github.com/user-attachments/assets/c185fb20-ad2a-4389-b399-df8530ca5a56)

**8.** Creating directories and files in the Control Node for the docker-compose file and ansible-playbook

- To create the Docker compose file, run the following lines.
  ```sh
  mkdir ~/jira-docker # make a directory called jira-docker in home directory
  cd ~/jira-docker # navigate to the directory
  sudo nano docker-compose.yml # create a file called docker-compose.yml and paste in docker compose code
  ```

- Paste in the docker-compose code in the **docker-compose.yml** file
  

- To create an Ansible Playbook file, run the following lines.
  ```sh
  mkdir ~/ansible-playbooks # make a directory called sndible-playbook in home directory
  cd ~/ansible-playbooks # navigate to the directory
  sudo nano deploy_jira.yml # create a file called deploy_jira.yml and paste in ansible playbook file
  ```

- Paste in this ansible-playbook code in the **deploy_jira.yml** file

**9.** Run playbook

- Ensure you are in the “cd /ansible_playbooks/deploy_jira.yml”, then run the playbook.
  ```sh
  ansible-playbook deploy_jira.yml
  ```

  output:

 ![image](https://github.com/user-attachments/assets/2f1278d1-0c5b-4a16-ade8-dd92bc6bdd66)


**10.** Creating a Load balancer to route traffic to the Worker Nodes.

- Create a Load balancer Target Group. The target group protocol should be HTTP and port 8080 since in the docker-compose file, we created containers that are open on port 8080 and are mapped on port 8080 of worker nodes. Hit Next.

![image](https://github.com/user-attachments/assets/1770f95a-cc89-4e86-b332-59dc3cd92219)


- Select the worker nodes and “include as pending below” then create the target group.

![image](https://github.com/user-attachments/assets/b90f3d8d-3670-492e-8a51-cbeeadbe67ac)


Create the Load balancer.

- Choose Application Load balancer

![image](https://github.com/user-attachments/assets/36e4559a-7ee0-4b24-9ac5-6fcc015b2c24)


- Name the Load balancer and at the level of the Network setting, choose the desired VPC and all subnets.

- At the level of Listeners and Routing, choose an HTTP protocol listening from port 80, and pass the created target group. Then create the load balancer.

![image](https://github.com/user-attachments/assets/b2eb9546-036d-48b2-9ad9-fdb8ef76c7a5)


- Create a Security group that allows HTTPS and HTTPS traffic from anywhere.

![image](https://github.com/user-attachments/assets/84b7a141-3861-4f0e-8ac9-409309f9c9ea)



**11.** Testing

- Edit the inbound rule of the worker nodes security group to accept HTTP and HTTPS traffic from the Load balancer security group.

![image](https://github.com/user-attachments/assets/082b57cf-f085-4c46-a8e7-cb620f869f86)


- Copy the DNS of the load balancer and paste it on the Browser, you should see this.

![image-20240717-205519](https://github.com/user-attachments/assets/362693bc-66aa-4eaf-bc6c-8989ff77bc35)



  
