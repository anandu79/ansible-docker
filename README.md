# ansible-docker

Here, we are going to install docker in a remote instance. For that launch 2 Amazon linux instnces from AWS console (one master instance which has Ansible installed in it and client instance).

## Ansible installation in the master instance

Use the below command to install Ansible in the master instance.

```
sudo amazon-linux-extras install ansible2 -y
```

Verify the installation by using the command:

```
ansible --version
```

## Key file creation

Create a key file in the working directorty and add key in it:

```
vim aws.pem
```

Change permission of the key file:

```
chmod 400 aws.pem
```

## hosts file creation

Create a hosts file inside the working directory and add the remote server in it:

```
vim hosts
```

Add the below contents:

```
[amazon]    
remote-server-ip ansible_user="ec2-user" ansible_port=22 ansible_ssh_private_key_file="aws.pem"
```

## Check connectivity

Check connectivity to the remote instance by using any of the below commands:

```
ansible -i hosts amazon -f 1 -m ping
```

or 

```
ansible -i hosts all -f 1 -m ping
```

## Playbook file creation

Create a playbook file as shown below to install Docker in the remote instance.

```
vim playbook.yml
```

Add the below contents in it:

```
---  
- name: "Setting up docker container using ansible"  
  become: true  
  hosts: all  
  vars:  
    package:   
      - "docker"  
  tasks:  
      
    - name: "Updating all packages"  
      yum:  
        name: "*"  
        state: latest  
    
    - name: "Reboooting system"   # =======> Reboot a machine, wait for it to go down, come back up, and respond to commands.
      reboot:   
        reboot_timeout: 300  
        test_command: uptime      # =======> Executes a test (uptime) command in the remote instance.
   
   - name: "uptime" 
      shell: uptime 
      register: uptime_output 
   
   - name: "Message" 
      debug: 
        var: uptime_output.stdout_lines                
    
    - name: "Docker installation"  
      yum:  
        name: "{{ package }}"  
        state: present  
    
    - name: "adding existing user to group docker"  
      user:  
        name: ec2-user  
        groups: docker  
        append: yes            
   
    - name: "restarting/enabling docker service"  
      service:  
        name: docker  
        state: restarted  
        enabled: true
```

## Executing the playbook file

Before executing the playbook file, check if there are any syntax errors by using the command:

```
ansible-playbook -i hosts playbook.yml --syntax-check
```

If there are no errors, execute the playbook file:

```
ansible-playbook -i hosts playbook.yml
```

## Output

After, execution, we will receve the below output and docker will be installed in the remote instance.

![docker](https://github.com/anandu79/ansible-docker/blob/main/images/output.jpg)
