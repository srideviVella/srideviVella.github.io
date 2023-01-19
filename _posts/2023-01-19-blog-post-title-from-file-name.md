## SSH Tunneling in Funnel HackTheBox

Now this is all good, but how to access the psql application in <a href="https://app.hackthebox.com/starting-point"> Funnel machine </a> in HackTheBox. Let's assume the vm as the funnel machine from now on.

### PSQL application in the Funnel
In the machine, there is psql application running on the default port, 5432. Till getting the user access the machine is fairly straightforward, things get bizarre from then(atleast to me). To interact with the application, the user needs sudo permissions to install psql command which is not possible at this point. The application port is not open to the internet either as nmap showed only 2 ports open(ftp and ssh). So, instead port forwarding can be used. But the question is to which type of port forwarding to choose, local or remote. 

There is an application running on a remote server(the funnel VM) which also happens to be a SSH server as we are able to login through SSH.  So, from our local machine we can use the local port forwarding to access the PSQL application in the funnel.

Hmm, why not use remote port forwarding on the vm as we are having ssh session on it and have itself serve as the SSH server? That might not be as easy as local port forwarding, as the ssh command(with -R option) should be run on the vm and certain configurations changes should be set on it. It cannot be achieved as the user is not having root permissions and might be logged on a real machine. So, the easiest approach is to use local port forwarding.

The following command sets up the local port forwarding:
```
ssh -L 8081:localhost:5432 christine@IP_Of_VM
```

Here, -L means its a local port forwarding. 8081 is the port at which the application can be accessed locally. First the machine connects to the vm, which is server. The destination address is itself, so localhost/0.0.0.0 can be used directly in the command. It doesn't refer to our local machine on which command is being executed. After executing the command, it will open a ssh session to the vm.

Note: For some reason, usage Ip address in place of localhost is giving "Connection refused" error while psql command is used.

Now, how to access the application from local machine? Simply use the following command:
```
psql -h localhost -U christine -p 8081 --list
```
The above command first asks for password of the user and then lists the databases available.

### Application access through Dynamic Port Forwarding
Even Dynamic port forwarding can be used instead of the local port forwarding as well. It is used when there are multiple applications running on different ports to reduce the manual errors while running the commands. It essentially uses a proxy running at a port on the local machine. In a browser it is easy to use a proxy by modifying proxy settings in it. But our application is not a web application. The answer is proxychains. In Linux, there is a tool called proxychains to let all the requests go through that proxy. To let the proxy handle all the requests, the configuration file at "/etc/proxychains4.conf" and the following line should be added.


![proxychains config file](https://user-images.githubusercontent.com/102641432/213583939-1711eaf5-31f0-4251-ab09-a13e42e393c8.PNG)


```
ssh -D 8081 christine@IP_Of_VM
```
![Connection](https://user-images.githubusercontent.com/102641432/213583996-a6ecf7af-365d-4f04-a6a0-b2b434ef5aad.PNG)


Here -D means it is a Dynamic port forwarding, the port 8081 represents that the proxy runs at the port. 

Now to let the requests go through to the VM, we need prefix each command with proxychains as shown below:
```
proxychains psql -h localhost --list -U christine
```

![psql command](https://user-images.githubusercontent.com/102641432/213584040-b799adc0-dc7f-4972-a7fe-ecaf3eff8587.PNG)

Note, we are not giving the port and the default port is for psql is 5432. This is because proxy directly invokes the port on the machine. There is no mapping needed.
