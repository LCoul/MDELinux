# Local Computer
Connect to your Ansible control node server, for example using this command: ssh rod@ms-012.myworkspace.microsoft.com -p 45163.
ms-012.myworkspace.microsoft.com could also be an IP address.
Answer 'yes' when prompted if you are sure to continue connecting
Provide the login password when prompted
```ssh rod@ms-012.myworkspace.microsoft.com -p 45163```

# Ansible control node
1. Update and upgrade the server
```sudo apt update && sudo apt upgrade```

##View the hostname
```hostname```

## View the fully qualified domain name (FQDN) of the host
```hostname --fqdn```

## View the detail of the server using _lsb_release_ -a'
Notice the Linux distribution, the release (version), and the codename
``` lsb_release -a```

## Create a private/public key pair that you use to automate tasks using Ansible
```ssh-keygen -t rsa -C "ControlNodeKey" -f ansible/ControlNode```
```sudo vim ~/.ssh/config (add the following line: IdentityFile ~/.ssh/ControlNode)```


# Create folder in your working directory named ansible
```mkdir ansible```

# Create a file named hosts and add your Linux devices to the file
```sudo vim ansible/hosts```


# Ansible managed node

* Create an Ansible administrator user account
Run command 'id username' to verify that the user is member of the sudo group.
Run the command 'su - username' to login as the newly created user
>```sudo useradd -m lessi && sudo passwd lessi && sudo usermod -aG sudo lessi```<br>
>```id lessi```<br>
>```su - lessi```<br>
![Create admin user](/image-1.png)

## Create the .ssh folder and the authorized_keys inside that forder to hold your public keys


## Reference Documents
[Deploy Microsoft Defender for Endpoint on Linux with Ansible](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/linux-install-with-ansible?view=o365-worldwide)


