# Create a Service on Ubuntu
## [What is a Systemd Service?](https://linuxhandbook.com/create-systemd-services/)
A service is a background process, known as daemons in systems like Unix or Linux. A [system unit file](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files) is used to define a service.
#### What is a [process](https://tldp.org/LDP/tlk/kernel/processes.html)?
Processes carry out tasks within the operating system. A process can be thought of as a computer program in action. A background process is one that is run independently of a user. 
#### [How to list and manage services in Linux](https://www.hostinger.com/tutorials/manage-and-list-services-in-linux/)?
- List all services in Linux
`sudo systemctl list-unit-files --type service --all`
- List only the services that are active
`sudo systemctl | grep running`
- Start a service on Linux
`sudo systemctl start [service_name]`
- Stop a service
`sudo systemctl stop [service_name]`
- Check the status of a service
`sudo systemctl status [service_name]`
- Have a service run while the operating system is being loaded
`sudo systemctl enable [service_name]`
- Remove it from the initial load
`sudo systemctl disable [service_name]`
- Verify which port is being used by a service (in Ubuntu)
`sudo apt install netstat-nat` `sudo netstat -plnt`

## [Understanding system unit files](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files)

#### What is a unit?
In systemd, a unit refers to any resource that the system knows how to operate on and manage. These are basically a standardized representation of system resources that can be managed by the suite of daemons and manipulated by the provided utilities.

#### Where are unit files located?
The system’s copy of unit files are generally kept in the `/lib/systemd/system` directory. When software installs unit files on the system, this is the location where they are placed by default. **You should not edit files in this directory.** If you wish to modify the way that a unit functions, the best location to do so is within the `/etc/systemd/system` directory.

#### Anatomy of a unit file
The internal structure of unit files are organized with sections. Sections are denoted by a pair of square brackets “[” and “]” with the section name enclosed within. Section names are well-defined and case-sensitive. Within these sections, unit behavior and metadata is defined through the use of simple directives using a key-value format with assignment indicated by an equal sign.
##### [Unit] Section Directives
**The first section found in most unit files is the [Unit] section.** This is generally used for defining metadata for the unit and configuring the relationship of the unit to other units. This section is often placed at the top because it provides an overview of the unit. Some common directives that you will find in the [Unit] section: 
- `Description=`: This directive can be used to describe the name and basic functionality of the unit. It is returned by various systemd tools, so it is good to set this to something short, specific, and informative.
##### [Install] Section Directives
**The last section is often the [Install] section.** This section is optional and is used to define the behavior or a unit if it is enabled or disabled. Enabling a unit marks it to be automatically started at boot. Because of this, only units that can be enabled will have this section. The directives within dictate what should happen when the unit is enabled:
- `WantedBy=`: This directive is the most common way to specify how a unit should be enabled. This directive allows you to specify a dependency relationship. When a unit with this directive is enabled, a directory will be created within `/etc/systemd/system` named after the specified unit with .wants appended to the end. Within this, a symbolic link to the current unit will be created, creating the dependency. For instance, if the current unit has WantedBy=multi-user.target, a directory called `multi-user.target.wants` will be created within `/etc/systemd/system` (if not already available) and a symbolic link to the current unit will be placed within.
- `RequiredBy=`: This directive is very similar to the `WantedBy=` directive, but instead specifies a required dependency that will cause the activation to fail if not met.
##### Unit-Specific Section Directives (The [Service] Section)
Sandwiched between the previous two sections, you will likely find unit type-specific sections. I'll only list The [Service] Section here. The [Service] section is used to provide configuration that is only applicable for services.
- `Type=`: One of the basic things that should be specified within the [Service] section. This categorizes services by their process and daemonizing behavior. This is important because it tells systemd how to correctly manage the servie and find out its state. `simple` is the default if the `Type=` and `Busname=` directives are not set, but the `ExecStart=` is set. 
The directives that actually defined how to manage our services:
- `ExecStart=`: This specifies the full path and the arguments of the command to be executed to start the process. This may only be specified once (except for “oneshot” services). If the path to the command is preceded by a dash “-” character, non-zero exit statuses will be accepted without marking the unit activation as failed.
- `Restart=`: This indicates the circumstances under which systemd will attempt to automatically restart the service. This can be set to values like “always”, “on-success”, “on-failure”, “on-abnormal”, “on-abort”, or “on-watchdog”. These will trigger a restart according to the way that the service was stopped.
- `RestartSec=`: If automatically restarting the service is enabled, this specifies the amount of time to wait before attempting to restart the service.
- `TimeoutSec=`: This configures the amount of time that systemd will wait when stopping or stopping the service before marking it as failed or forcefully killing it. You can set separate timeouts with TimeoutStartSec= and TimeoutStopSec= as well.
## How I created a service for my cat bathroom monitoring system project
The following unit file was created under `/etc/systemd/system/` and is named `cat_data_watcher.service`.  
```
[Unit]
Description=My cat_data_watcher daemon!
[Service]
User=cat_dev
#Code to execute
#Can be the path to an executable or code itself
#WorkingDirectory=/home/ubuntu/mydaemon
ExecStart=/home/cat_dev/.local/bin/cat_data_watcher
Type=simple
TimeoutStopSec=10
Restart=on-failure
RestartSec=5
[Install]
WantedBy=multi-user.target
```
**Then start the service, using the following command:**
```
sudo systemctl daemon-reload
sudo systemctl start cat_data_watcher
sudo systemctl status cat_data_watcher

# If everything is looking good, enable it so that it will run on boot 
sudo systemctl enable cat_data_watcher
# Restart the server and check if it's still running
```
**Get service log of this service to debug if needed:**
```
sudo journalctl --unit=cat_data_watcher
```
Now, the service is set up. This particular service I created watches new csv files in `/var/nfs/cat_watcher_output` (produced by the **CatWatcher program** running in the Nano device). When a new file is found, **it loads the data to the database `metabase_catwatcher_db`**. 

## How I Created a Metabase Service with Nginx
### Preparation 
- Install Java Runtime Environment ([JRE](https://adoptium.net/installation/))
- Create a directory `/opt/catwatcher/metabase`, where the downloaded [metabase jar file](https://www.metabase.com/start/oss/jar) is stored. 
- Go to the metabase directory and [run the JAR](https://www.metabase.com/docs/latest/installation-and-operation/running-the-metabase-jar-file) ```java -jar metabase.jar```
- Make sure a Postgresql database is set up for the Metabase. The database should match `MB_DB_TYPE`, `MB_DB_DBNAME`, `MB_DB_USER`, and `MB_DB_PASS` environment variables stated in the metabase config file `/etc/default/metabase` (See Step 3)
- Make sure nginx is set up to proxy requests to metabase. The content of the nginx config file I use is `/etc/nginx/sites-available/default`. The content is as follows:
```
# proxy requests to Metabase instance
server {
          listen 80;
          listen [::]:80;
          # localhost can be replaced as 192.168.1.157
          server_name localhost; 
          location / {
           proxy_pass http://192.168.1.157:3000; # Using 127.0.0.1 does not work
          }
        }
```
Run the commands after creating the nginx config file: 
```
# Test if the config file is valid
sudo nginx -t  
# Restart the nginx service
sudo systemctl restart nginx
```

### Steps
#### 1. Create a Metabase Service Unit File 
Services are typically registered at `/etc/systemd/system/<servicename>`. Use the following command to create a new file:
```sudo nano /etc/systemd/system/metabase.service```
Write the following content to the newly created unit file
```
[Unit]
Description=Metabase server
After=syslog.target
After=network.target
 
[Service]
WorkingDirectory=/home/cat_dev/cat_tech/metabase_jar
ExecStart=/usr/bin/java -jar /opt/catwatcher/metabase/metabase.jar
EnvironmentFile=/etc/default/metabase
# User is changed to cat_dev to avoid error
User=metabase 
Type=simple
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=metabase
SuccessExitStatus=143
TimeoutStopSec=120
Restart=always
  
[Install]
WantedBy=multi-user.target
```
#### 2. Create syslog conf
Create a syslog conf to make sure systemd can handle the logs properly
```
sudo nano /etc/rsyslog.d/metabase.conf
```
Write the following content in the newly created syslog conf file
```
if $programname == 'metabase' then /var/log/metabase.log
& stop
```
Restart the syslog service to load the new config
```
sudo systemctl restart rsyslog.service
```
#### 3. Create the Metabase Config file for environment variables 
[Metabase Environment variables](https://www.metabase.com/docs/latest/configuring-metabase/environment-variables) provide a good way to customize and configure your Metabase instance on your server. On Debian systems, services typically expect to have accompanying configs inside `etc/default/<service-name>`
Create a new file to store the environment variables
```
sudo nano /etc/default/metabase
```
Write in the following content:
```
MB_PASSWORD_COMPLEXITY=normal
MB_JETTY_HOST=192.168.1.157
MB_JETTY_PORT=3000
MB_DB_TYPE=postgres
MB_DB_DBNAME=metabase_catwatcher_db
MB_DB_PORT=5432
MB_DB_USER=metabase_catwatcher_user
MB_DB_PASS=metabase_catwatcher_pw
MB_DB_HOST=localhost  # 20220822 changed to 192.168.1.157
MB_EMOJI_IN_LOGS=true
# Add any other env vars you want available to Metabase
```
#### 4. Register the Metabase service 
```
sudo systemctl daemon-reload
sudo systemctl start metabase.service
sudo systemctl status metabase.service
```
Enable the service to startup during boot
```
sudo systemctl enable metabase.service
```
That's it!

