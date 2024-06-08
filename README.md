# MySQL Master-Slave Replication Setup using Docker

1. Setup a one-master-two-slave database
2. Use Garfana to monitor it

### Setup Instructions
### Step 1: Start the Containers
Launch the all services defined in `docker-compose.yml`
```sh
docker compose up -d
```
### Step 2: Accessing the MySQL Servers
Access the bash shell of each container using these commands:
```sh
docker exec -it mysql-master bash
docker exec -it mysql-slave1 bash
docker exec -it mysql-slave2 bash
```
Connect to MySQL
```sh
mysql -uroot -p
```
### Step 3: Configure MySQL Users on All Servers
Set up permissions for MySQL exporter
```sh
CREATE USER 'exporter'@'%' IDENTIFIED BY 'password' WITH MAX_USER_CONNECTIONS 3;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'%';
FLUSH PRIVILEGES;
```
### Step 4: Master Server Configuration
Grant the replication privileges
```sh
GRANT REPLICATION SLAVE ON *.* TO 'user'@'%';
FLUSH PRIVILEGES;
```
Display the master log status
```sh
SHOW MASTER STATUS;
```
### Step 6: Find Master IP Address from Docker
Find container id
```sh
docker ps -a
docker inspect {master_server_container}
```
### Step 5: Slaves Server Configuration
On each slave server, configure the replication using the master's IP address, log file, and position
```sh
CHANGE MASTER TO 
  MASTER_HOST='master_ip_address_from_docker',
  MASTER_USER='user',
  MASTER_PASSWORD='password',
  MASTER_LOG_FILE='log_file_name_from_master_status',
  MASTER_LOG_POS=log_file_position_from_master_status,
  MASTER_PORT=3306,
  MASTER_CONNECT_RETRY=60,
  GET_MASTER_PUBLIC_KEY=1;
```
Initiate the replication process
```sh
START SLAVE;
```
### Step 6: Verification
Ensure replication is functioning properly
```sh
SHOW SLAVE STATUS\G
```
Slave_IO_Running and Slave_SQL_Running values should be 'Yes'
### Step 7: Prometheus
Go to `http://localhost:9090/targets?search=` to check if all database are up 
### Step 8: Grafana
Go to http://localhost:3000 and log in with user=ADMIN, password=ADMIN
Navigate to Configuration -> Data Sources -> Add Data Source, select Prometheus, and click Add
Enter http://prometheus:9090 as the URL for the Prometheus server.
Click Save & Test
Navigate to the dashboard
Use template ID 14057 to create a new dashboard to monitor
