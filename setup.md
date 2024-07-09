Containerized Mysql 8.0.26 setup (Master and Slave)
Step 1: Docker and docker-compose setup.
	chmod 700 docker-setup.sh then ./docker-setup.sh
Step 2: Execute the below commands to create folders and file for mysql setup
mkdir -p /opt/mysql
mkdir -p /opt/mysql/mysql_data
mkdir -p /opt/mysql/mysql_config
cp ./docker-compose.yml /opt/mysql
cp ./mysql_config/my.cnf /opt/mysql/mysql_config
Step 3: Setting mysql root password
nano /root/db_root_password.txt
enter password for mysql root user
Step 4: Update my.cnf file as per requirement
nano /opt/mysql/mysql_config/my.cnf
update the server-id (must be unique for master and replica server)
add the setting as you needed (like max-connections, thread-cache-size and etc)
Step 5: Updating container name and starting container
cd /opt/mysql
update the container name in docker-compose file (mysql_master/mysql_slave)
nano docker-compose.yml
docker-compose up -d
Step 6: Repeat step 1-6 to setup the replication server on another machine
Step 7: Create user in master for replication and grant the replication privileges
docker exec -it mysql_master mysql -u root -p
CREATE USER 'replica_user'@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'replica_user'@'%';
FLUSH PRIVILEGES;
SHOW MASTER STATUS;
Note the log file and position from SHOW MASTER STATUS for later user.
 
Step 8: Configure the Replica Server
docker exec -it mysql_replica mysql -u root -p
CHANGE MASTER TO
MASTER_HOST='mysql-master-ip',
MASTER_USER='replica_user',
MASTER_PASSWORD='replica_password',
MASTER_LOG_FILE='mysql-bin.xxxxxx',
MASTER_LOG_POS=xxxx;
START SLAVE;
SHOW SLAVE STATUS\G
confirm that Slave_IO_Running and Slave_SQL_Running show as “Yes“

Note: add "relay-log=relay-bin" in replica my.cnf file so if the server get crashed it can continue the replication

troubleshooting of replica
start slave;
stop slave;
reset slave;
