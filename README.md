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
