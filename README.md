####  This is an sample playbook file to install a wordpress site in an Ec2 instance
```
---
vim wordpressinstallation.yml

- name: "Wordpress install on amazon ec2"
  hosts: all
  become: yes
  vars:
    domain: domain.com
    mysql_database: "db"
    mysql_user: "user"
    mysql_root: "mysql&"
    mysql_password: "wpuser"

  tasks:
    
    - name: "Apache installtion"
      yum:
        name: httpd,php,php-mysql
        state: present
    
    - name: "creating virtual host"
      template:
        src:  virtual.tmp
        dest: "/etc/httpd/conf.d/{{ domain }}.conf"
        
    - name: "Creating documentroot for the domain"
      file:
        path: "/var/www/html/{{ domain }}"
        state: directory
        owner: apache
        group: apache
        
    - name: "Httpd restarting"
      service:
         name: httpd
         state: restarted
         enabled: true
         
    - name: "Mariadb installtion"
      yum:
        name: 
           - mariadb-server
           - MySQL-python
         
        state: present
         
    - name: "restarting mariadb"
      service:
         name: mariadb
         state: restarted
         enabled: true
         
    - name: "resting mysql root password"
      ignore_errors: true
      mysql_user:
        login_user: "root"
        login_password: ""
        user: "root"
        password: "{{ mysql_root }}"
        host_all: true
        
    - name: "Deleting annonymous users"
      mysql_user:
        login_user: "root"
        login_password: "{{ mysql_root }}"
        user: ""
        state: absent
        
    - name: "Creating database"
      mysql_db:
        login_user: "root"
        login_password: "{{ mysql_root }}"
        name: "{{ mysql_database }}"
        state: present
        
    - name: "Creating database user"
      mysql_user:
        login_user: "root"
        login_password: "{{ mysql_root }}"
        name: "{{ mysql_user }}"
        password: "{{ mysql_password }}"
        state: present
        priv: "{{ mysql_database }}.*:ALL"
    
    - name: "Downloading wordpress"
      get_url:
        url: "https://wordpress.org/wordpress-4.9.tar.gz"
        dest: /tmp/wordpress.tar

    - name: "Woordpress extract"
      unarchive:
        src: /tmp/wordpress.tar
        dest: /tmp/
        remote_src: true

    - name: "Copying content to remote server website path"
      copy:
        src: /tmp/wordpress/
        dest: "/var/www/html/{{ domain }}"
        remote_src: true
        owner: apache
        group: apache

    - name: "Uploading wp-config.php file"
      template:
        src: wp-config.php.tl
        dest: "/var/www/html/{{ domain }}/wp-config.php"
        owner: apache
        group: apache

    - name: "restaring services"
      service:
        name: "{{ item }}"
        state: restarted
      with_items:
          - httpd
          - mariadb
````
#### Output 

````
[root@ip-172-31-3-200 ec2-user]# ansible-playbook wordpressinstallation.yml

PLAY [Wordpress install on amazon ec2] ***********************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************
ok: [172.31.7.238]

TASK [Apache installtion] ************************************************************************************************************
ok: [172.31.7.238]

TASK [creating virtual host] *********************************************************************************************************
ok: [172.31.7.238]

TASK [Creating documentroot for the domain] ******************************************************************************************
ok: [172.31.7.238]

TASK [Httpd restarting] **************************************************************************************************************
changed: [172.31.7.238]

TASK [Mariadb installtion] ***********************************************************************************************************
ok: [172.31.7.238]

TASK [restarting mariadb] ************************************************************************************************************
changed: [172.31.7.238]

TASK [resting mysql root password] ***************************************************************************************************
fatal: [172.31.7.238]: FAILED! => {"changed": false, "msg": "unable to connect to database, check login_user and login_password are correct or /root/.my.cnf has the credentials. Exception message: (1045, \"Access denied for user 'root'@'localhost' (using password: NO)\")"}
...ignoring

TASK [Deleting annonymous users] *****************************************************************************************************
ok: [172.31.7.238]

TASK [Creating database] *************************************************************************************************************
ok: [172.31.7.238]

TASK [Creating database user] ********************************************************************************************************
ok: [172.31.7.238]

TASK [Downloading wordpress] *********************************************************************************************************
ok: [172.31.7.238]

TASK [Woordpress extract] ************************************************************************************************************
ok: [172.31.7.238]

TASK [Copying content to remote server website path] *********************************************************************************
ok: [172.31.7.238]

TASK [Uploading wp-config.php file] **************************************************************************************************
changed: [172.31.7.238]

TASK [restaring services] ************************************************************************************************************
changed: [172.31.7.238] => (item=httpd)
changed: [172.31.7.238] => (item=mariadb)

PLAY RECAP ***************************************************************************************************************************
172.31.7.238               : ok=16   changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1   
````

#### Vitrual host file 

````
[root@ip-172-31-3-200 ec2-user]# cat virtual.tmp
<virtualhost *:80>
    servername {{ domain }}
    documentroot /var/www/html/{{ domain }}
    directoryindex index.php index.html
</virtualhost>
````
### wp-config file
```
[root@ip-172-31-3-200 ec2-user]# cat virtual.tmp
<virtualhost *:80>
    servername {{ domain }}
    documentroot /var/www/html/{{ domain }}
    directoryindex index.php index.html
</virtualhost>
[root@ip-172-31-3-200 ec2-user]# cat wp-config.php.tl
<?php

define( 'DB_NAME', '{{ mysql_database }}' );
define( 'DB_USER', '{{ mysql_user }}' );
define( 'DB_PASSWORD', '{{ mysql_password }}' );
define( 'DB_HOST', 'localhost' );
define( 'DB_CHARSET', 'utf8' );
define( 'DB_COLLATE', '' );

define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );

$table_prefix = 'wp_';

define( 'WP_DEBUG', false );

if ( ! defined( 'ABSPATH' ) ) {
        define( 'ABSPATH', dirname( __FILE__ ) . '/' );
}
require_once( ABSPATH . 'wp-settings.php' );
```


