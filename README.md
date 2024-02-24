<details>
    <summary>Deploy Manually</summary>
</details>

<details>
    <summary>Deploy with a Script</summary>
</details>

<details>
    <summary><b>Deploy with Ansible</b></summary>

### Connect to Ansible Control Node
From a shell (for example PowerShell), connect to your Ansible control node server with the following command:<br> _<**ssh rod@IPAddress -p 45163**>_<br>
The IPAddress could also be the FQDN of the server, **-p** specifies the ssh port if TCP port 22 is not the default. Answer 'yes' when prompted if you are sure to continue connecting, and provide the login password when prompted.<br>
```PowerShell
ssh rod@IPAddress -p 45163
```

## Ansible Control Node
### Basic Configurations
View the details of the control node
Update and upgrade the server<br>
```bash
 sudo apt update && sudo apt upgrade
 ```
View the hostname<br>
```bash
hostname
```
View the fully qualified domain name (FQDN) of the host<br>
```bash
hostname --fqdn
```
View the detail of the server using _<lsb_release -a>_.<br> 
Notice the Linux distribution, the release (version), and the codename<br>
```bash
lsb_release -a
```
Create a private/public key pair that you use to automate tasks using Ansible<br>
```bash
ssh-keygen -t rsa -C "ControlNodeKey" -f ansible/ControlNode
sudo vim ~/.ssh/config (add the following line: IdentityFile ~/.ssh/ControlNode)
```

Create folder in your working directory named ansible<br>
```bash
mkdir ansible
```
Create a file named hosts and add your Linux devices to the file<br>
```bash
sudo vim ansible/hosts
```
### Install Ansible
```bash
ansible-playbook -K install_mdatp.yml -i hosts
```
![Install Ansible](/install_mdatp.mp4)

### Uninstall Ansible
```bash
ansible-playbook -K uninstall_mdatp.yml -i hosts
```
![Uninstall Ansible](/)

## Ansible Managed Nodes

Create an Ansible administrator user account running the following command:<br>
_<sudo useradd -m user && sudo passwd user && sudo usermod -aG sudo user>_<br>
- **sudo useradd -m user**: creates a user (-m creates the user's directory).
- **sudo passwd user**: sets the user password.
- **sudo usermod -aG user**: adds the user to the sudo group.<br>

Run the _<id - user>_ command to verify that the user is member of the sudo group.<br>
Run the _<su - user>_ command to login as the newly created user.

```bash
sudo useradd -m lessi && sudo passwd lessi && sudo usermod -aG sudo lessi
id lessi
su - lessi
```
For example:
![Create admin user](/image-1.png)
</details>

<br>

## Reference Documents
[Deploy Microsoft Defender for Endpoint on Linux with Ansible](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/linux-install-with-ansible?view=o365-worldwide)<br>
[Installing Ansible - Ansible Documentation](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)


