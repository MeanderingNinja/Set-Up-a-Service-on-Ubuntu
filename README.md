# Create-a-Service-on-Ubuntu
## [What is a Systemd Service?](https://linuxhandbook.com/create-systemd-services/)
A service is a background process, known as daemons in systems like Unix or Linux. A [system unit file](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files) is used to define a service.
### What is a [process](https://tldp.org/LDP/tlk/kernel/processes.html)?
Processes carry out tasks within the operating system. A process can be thought of as a computer program in action. A background process is one that is run independently of a user. 
### [How to list services in Linux](https://www.hostinger.com/tutorials/manage-and-list-services-in-linux/)?
`sudo systemctl list-unit-files --type service --all`

