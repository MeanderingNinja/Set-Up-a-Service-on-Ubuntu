# Create-a-Service-on-Ubuntu
## [What is a Systemd Service?](https://linuxhandbook.com/create-systemd-services/)
A service is a background process, known as daemons in systems like Unix or Linux. A [system unit file](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files) is used to define a service.
### What is a [process](https://tldp.org/LDP/tlk/kernel/processes.html)?
Processes carry out tasks within the operating system. A process can be thought of as a computer program in action. A background process is one that is run independently of a user. 
### [How to list and manage services in Linux](https://www.hostinger.com/tutorials/manage-and-list-services-in-linux/)?
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
### What is a unit?
In systemd, a unit refers to any resource that the system knows how to operate on and manage. These are basically a standardized representation of system resources that can be managed by the suite of daemons and manipulated by the provided utilities.
### Where are unit files located?
The system’s copy of unit files are generally kept in the `/lib/systemd/system` directory. When software installs unit files on the system, this is the location where they are placed by default. You should not edit files in this directory. If you wish to modify the way that a unit functions, the best location to do so is within the /etc/systemd/system directory.
### Anatomy of a unit file
The internal structure of unit files are organized with sections. Sections are denoted by a pair of square brackets “[” and “]” with the section name enclosed within. Section names are well-defined and case-sensitive. Within these sections, unit behavior and metadata is defined through the use of simple directives using a key-value format with assignment indicated by an equal sign.
#### [Unit] Section Directives
The first section found in most unit files is the [Unit] section. This is generally used for defining metadata for the unit and configuring the relationship of the unit to other units. This section is often placed at the top because it provides an overview of the unit. Some common directives that you will find in the [Unit] section: 
- `Description=`: This directive can be used to describe the name and basic functionality of the unit. It is returned by various systemd tools, so it is good to set this to something short, specific, and informative.
#### [Install] Section Directives
The last section is often the [Install] section. This section is optional and is used to define the behavior or a unit if it is enabled or disabled. Enabling a unit marks it to be automatically started at boot. Because of this, only units that can be enabled will have this section. The directives within dictate what should happen when the unit is enabled:
- `WantedBy=`: This directive is the most common way to specify how a unit should be enabled. This directive allows you to specify a dependency relationship. When a unit with this directive is enabled, a directory will be created within `/etc/systemd/system` named after the specified unit with .wants appended to the end. Within this, a symbolic link to the current unit will be created, creating the dependency. For instance, if the current unit has WantedBy=multi-user.target, a directory called `multi-user.target.wants` will be created within `/etc/systemd/system` (if not already available) and a symbolic link to the current unit will be placed within.
- `RequiredBy=`: This directive is very similar to the `WantedBy=` directive, but instead specifies a required dependency that will cause the activation to fail if not met.
#### Unit-Specific Section Directives (The [Service] Section)
Sandwiched between the previous two sections, you will likely find unit type-specific sections. I'll only list The [Service] Section here. The [Service] section is used to provide configuration that is only applicable for services.
- `Type=`: One of the basic things that should be specified within the [Service] section. This categorizes services by their process and daemonizing behavior. This is important because it tells systemd how to correctly manage the servie and find out its state. `simple` is the default if the `Type=` and `Busname=` directives are not set, but the `ExecStart=` is set. 
The directives that actually defined how to manage our services:
- `ExecStart=`: This specifies the full path and the arguments of the command to be executed to start the process. This may only be specified once (except for “oneshot” services). If the path to the command is preceded by a dash “-” character, non-zero exit statuses will be accepted without marking the unit activation as failed.
- `Restart=`: This indicates the circumstances under which systemd will attempt to automatically restart the service. This can be set to values like “always”, “on-success”, “on-failure”, “on-abnormal”, “on-abort”, or “on-watchdog”. These will trigger a restart according to the way that the service was stopped.
- `RestartSec=`: If automatically restarting the service is enabled, this specifies the amount of time to wait before attempting to restart the service.
- `TimeoutSec=`: This configures the amount of time that systemd will wait when stopping or stopping the service before marking it as failed or forcefully killing it. You can set separate timeouts with TimeoutStartSec= and TimeoutStopSec= as well.
### How I created a service for my cat bathroom monitoring system project
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



