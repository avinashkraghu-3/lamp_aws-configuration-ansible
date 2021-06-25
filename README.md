
# lamp_aws-configuration-ansible

$$$$$$
before run this script you must remove ec2_new.pem file 
$$$$$$

###Create Aws cli using Pem file- Using following command###  ---> you can use aws cli using to generate pem file
aws ec2 create-key-pair --key-name ec2 --output text > ec2.pem

###install ansible 
install ansible using command --> check google then install ansible 

###Installing boto3
┌─[✗]─[avi@parrot]─[~/ansible]

└──╼ $pip install boto boto3 → Both boto and boto3 packages are needed for this lab


###Upgrade boto and boto3 using following command###
============================================

┌─[✗]─[avi@parrot]─[~/ansible]
└──╼ $pip install boto3 --upgrade
Requirement already satisfied: boto3 in /home/avi/.local/lib/python3.9/site-packages (1.16.50)
Collecting boto3
  Downloading boto3-1.17.99-py2.py3-none-any.whl (131 kB)
     |████████████████████████████████| 131 kB 1.5 MB/s 
Requirement already satisfied: jmespath<1.0.0,>=0.7.1 in /usr/lib/python3/dist-packages (from boto3) (0.10.0)
Collecting botocore<1.21.0,>=1.20.99
  Downloading botocore-1.20.99-py2.py3-none-any.whl (7.6 MB)
     |████████████████████████████████| 7.6 MB 4.6 MB/s 
Collecting s3transfer<0.5.0,>=0.4.0
  Downloading s3transfer-0.4.2-py2.py3-none-any.whl (79 kB)
     |████████████████████████████████| 79 kB 916 kB/s 
Requirement already satisfied: urllib3<1.27,>=1.25.4 in /home/avi/.local/lib/python3.9/site-packages (from botocore<1.21.0,>=1.20.99->boto3) (1.26.5)
Requirement already satisfied: python-dateutil<3.0.0,>=2.1 in /home/avi/.local/lib/python3.9/site-packages (from botocore<1.21.0,>=1.20.99->boto3) (2.7.5)
Requirement already satisfied: six>=1.5 in /home/avi/.local/lib/python3.9/site-packages (from python-dateutil<3.0.0,>=2.1->botocore<1.21.0,>=1.20.99->boto3) (1.12.0)
Installing collected packages: botocore, s3transfer, boto3
  Attempting uninstall: botocore
    Found existing installation: botocore 1.19.50
    Uninstalling botocore-1.19.50:
      Successfully uninstalled botocore-1.19.50
  Attempting uninstall: s3transfer
    Found existing installation: s3transfer 0.3.3
    Uninstalling s3transfer-0.3.3:
      Successfully uninstalled s3transfer-0.3.3
  Attempting uninstall: boto3
    Found existing installation: boto3 1.16.50
    Uninstalling boto3-1.16.50:
      Successfully uninstalled boto3-1.16.50
Successfully installed boto3-1.17.99 botocore-1.20.99 s3transfer-0.4.2
┌─[avi@parrot]─[~/ansible]
└──╼ $pip install boto --upgrade
Requirement already satisfied: boto in /home/avi/.local/lib/python3.9/site-packages (2.49.0)
Collecting boto
  Using cached boto-2.49.0-py2.py3-none-any.whl (1.4 MB)
  Downloading boto-2.48.0-py2.py3-none-any.whl (1.4 MB)
     |████████████████████████████████| 1.4 MB 1.8 MB/s 
┌─[avi@parrot]─[~/ansible]


###Storing your keys in Ansible vault###
After creating the IAM account, we’ll need to store the AWS keys. Since they are sensitive data, we should use Ansible vault for this.

┌─[avi@parrot]─[~/ansible]

└──╼ $ansible-vault create aws_keys.yml

Open the file
Using vim editor

┌─[avi@parrot]─[~/ansible]


└──╼ vim ansible-vault create aws_keys.yml

aws_access_key: AKIAJLxxxxxxxxxxxxx43UA
aws_secret_key: iMcMxxxxxv9k+bdLqMGxxxxxxxxxxxxxxxxuSKFnUt

Once you save the file, all the content will be encrypted. Let’s check that


┌─[avi@parrot]─[~/ansible]
└──╼cat aws_keys.yml 
$ANSIBLE_VAULT;1.1;AES256
63333038396266346466383037653433613336643164316566353030663162303434323339316330
36613334323463346163630a656233303535333534346262616465
3437306339313235636263663334343364326237303366
61313764363331356330386639323666373433323733383636373635656335313234643364333832
3066


###Setting up the hosts file

┌─[avi@parrot]─[~/ansible]

└──╼vim hosts

Add the following entry
[local]
localhost ansible_connection=local ansible_python_interpreter=python3 

└──╼ $cat hosts
[local]
localhost ansible_connection=local 

[all:vars]
ansible_python_interpreter=/usr/bin/python
┌─[avi@parrot]─[~/ansible]
└──╼ $


#### create ec2 instance Keypair ansible play-book####



---
- hosts: local
  connection: local
  gather_facts: no
  tasks:
   - name: Create New EC2 key
     ec2_key:
      name: ec2_new → keyPair name
      region: us-east-2
     register: ec2_key_result

   - name: Save private key
     copy: content="{{ ec2_key_result.key.private_key }}" dest="./ec2_new.pem" mode=0600
     when: ec2_key_result.changed







########Part 1: Building the EC2 instance########




└──╼ $cat ec2.yml 
---
- hosts: local
  connection: local
  gather_facts: False
  vars:
    instance_type: t2.micro
    security_group: webservers_sg
    image: ami-00399ec92321828f5
    keypair: ec2_new → keypair name
    region: us-east-2
    count: 1
  vars_files:
    - aws_keys.yml


They keypair refers to the name of the public/private key pair that you created earlier.


#######Task 1: Creating a security group#######



 tasks:
    - name: Create a security group
      ec2_group:
        name: "{{ security_group }}"
        description: The webservers security group
        region: "{{ region }}"
        aws_access_key: "{{ aws_access_key }}"
        aws_secret_key: "{{ aws_secret_key }}"
        rules:
          - proto: tcp
            from_port: 22
            to_port: 22
            cidr_ip: 0.0.0.0/0
          - proto: tcp
            from_port: 80
            to_port: 80
            cidr_ip: 0.0.0.0/0
          - proto: tcp
            from_port: 443
            to_port: 443
            cidr_ip: 0.0.0.0/0
        rules_egress:
          - proto: all
            cidr_ip: 0.0.0.0/0



#######Task 2: Creating and launching the EC2 instance#####



   - name: Launch the new EC2 Instance
      ec2:
        aws_access_key: "{{ aws_access_key }}"
        aws_secret_key: "{{ aws_secret_key }}"
        group: "{{ security_group }}"
        instance_type: "{{ instance_type }}"
        image: "{{ image }}"
        wait: true 
        region: "{{ region }}"
        keypair: "{{ keypair }}"
        count: "{{count}}"
      register: ec2




#######Task 3: Adding the newly created instance to the hosts file#####


   - name: Add the newly created host so that we can further contact it
      add_host:
        name: "{{ item.public_ip }}"
        groups: webservers
      with_items: "{{ ec2.instances }}"





######Task 4: Tag the instance#########


   - name: Add tag to Instance(s)
      ec2_tag:
        aws_access_key: "{{ aws_access_key }}"
        aws_secret_key: "{{ aws_secret_key }}"
        resource: "{{ item.id }}" 
        region: "{{ region }}" 
        state: "present"
      with_items: "{{ ec2.instances }}"
      args:
        tags:
          Type: webserver 



#####Task 5: Finishing up instance creation#####


   - name: Wait for SSH to come up
      wait_for:
        host: "{{ item.public_ip }}"
        port: 22 
        state: started 
      with_items: "{{ ec2.instances }}"





----------------------------------------Testing----------------------------------------------------------------------
####Part 6: Deploying Apache ####

- hosts: webservers
  remote_user: ubuntu
  become: yes
  gather_facts: no
  pre_tasks:
   - name: Update apt-get repo and cache
     apt: 
      upgrade: yes
      update_cache: yes
      cache_valid_time: 86400
   - name: 'install python'
     raw: 'sudo apt-get -y install python'
  tasks:
   - name: Install Apache
     apt:
       name: apache2
       state: present
   - service: 
       name: apache2
       state: started
       enabled: yes

----------------------------------------Testing----------------------------------------------------------------------

###Create a new file called ansible.cfg in the current working directory and add the following:
[defaults]
host_key_checking = False
private_key_file = /home/avi/ansible/ec2_new.pem



####Part 7: Deploying LAMP ####
==========================

- hosts: webservers
  remote_user: ubuntu
  become: yes
  gather_facts: no
  vars_files:
    - vars/default.yml
  pre_tasks:
   - name: Update apt-get repo and cache
     apt: 
      upgrade: yes
      update_cache: yes
      cache_valid_time: 86400
  tasks:
    - name: Install prerequisites
      apt: name={{ item }} update_cache=yes state=latest force_apt_get=yes
      loop: [ 'aptitude' ]

    - name: "Add repository for PHP 7.0."
      apt_repository: 
        repo="ppa:ondrej/php" 
        update_cache=yes

  #Apache Configuration
    - name: Install LAMP Packages
      apt: name={{ item }} update_cache=yes state=latest
      loop: [ 'apache2', 'mysql-server', 'python3-pymysql', 'php', 'php-mysql', 'libapache2-mod-php' ]

    - name: Create document root
      file:
        path: "/var/www/{{ http_host }}"
        state: directory
        owner: "{{ app_user }}"
        mode: '0755'

    - name: Set up Apache virtualhost
      template:
        src: "files/apache.conf.j2"
        dest: "/etc/apache2/sites-available/{{ http_conf }}"
      notify: Reload Apache

    - name: Enable new site
      shell: /usr/sbin/a2ensite {{ http_conf }}
      notify: Reload Apache

    - name: Disable default Apache site
      shell: /usr/sbin/a2dissite 000-default.conf
      when: disable_default
      notify: Reload Apache

  # MySQL Configuration
    - name: Sets the root password
      mysql_user:
        name: root
        password: "{{ mysql_root_password }}"
        login_unix_socket: /var/run/mysqld/mysqld.sock

    - name: Removes all anonymous user accounts
      mysql_user:
        name: ''
        host_all: yes
        state: absent
        login_user: root
        login_password: "{{ mysql_root_password }}"

    - name: Removes the MySQL test database
      mysql_db:
        name: test
        state: absent
        login_user: root
        login_password: "{{ mysql_root_password }}"

    - name: Creates database for Testing
      mysql_db:
        name: "{{ mysql_db }}"
        state: present
        login_user: root
        login_password: "{{ mysql_root_password }}"
      tags: [ mysql ]

    - name: Create MySQL user for  Testing
      mysql_user:
        name: "{{ mysql_user }}"
        password: "{{ mysql_password }}"
        priv: "*.*:ALL,GRANT"
        state: present
        login_user: root
        login_password: "{{ mysql_root_password }}"
      tags: [ mysql ]


  # UFW Configuration
    - name: "UFW - Allow HTTP on port {{ http_port }}"
      ufw:
        rule: allow
        port: "{{ http_port }}"
        proto: tcp

  # PHP Info Page
    - name: Sets Up PHP Info Page
      template:
        src: "files/info.php.j2"
        dest: "/var/www/{{ http_host_1 }}/info.php"

    - name: wget adminer
      command: wget https://github.com/vrana/adminer/releases/download/v4.8.1/adminer-4.8.1.php

    - name: rename the adminer
      command: mv adminer-4.8.1.php adminer.php

    - name: move adminer to apache folder
      command: mv adminer.php /var/www/{{ http_host_1 }}
        

  handlers:
    - name: Reload Apache
      service:
        name: apache2
        state: reloaded

    - name: Restart Apache
      service:
        name: apache2
        state: restarted
