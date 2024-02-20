# Local Computer
Connect to your Ansible control node server, for example using this command:<br>
_ssh rod@ms-012.myworkspace.microsoft.com -p 45163_ (ms-012.myworkspace.microsoft.com could also be an IP address). Answer 'yes' when prompted if you are sure to continue connecting, and provide the login password when prompted.<br>
```PowerShell
ssh rod@ms-012.myworkspace.microsoft.com -p 45163
```
<details>
<summary>Ansible control node</summary>


Update and upgrade the server
```bash
sudo apt update && sudo apt upgrade
```
View the hostname
```bash
hostname
```
View the fully qualified domain name (FQDN) of the host
```bash
hostname --fqdn
```
View the detail of the server using _lsb_release -a_'. Notice the Linux distribution, the release (version), and the codename.
```bash
lsb_release -a
```
Create a private/public key pair that you use to automate tasks using Ansible
```bash
ssh-keygen -t rsa -C "ControlNodeKey" -f ansible/ControlNode
sudo vim ~/.ssh/config (add the following line: IdentityFile ~/.ssh/ControlNode)
```
Create folder in your working directory named ansible
```bash
mkdir ansible
```
Create a file named hosts and add your Linux devices to the file
```bash
sudo vim ansible/hosts
```
</details>

# Ansible managed node

* Create an Ansible administrator user account
Run command __id username'__ to verify that the user is member of the sudo group.
Run the command __su - username__ to login as the newly created user.

    ```bash
    sudo useradd -m lessi && sudo passwd lessi && sudo usermod -aG sudo lessi
    id lessi
    su - lessi
    ```
    ![Create admin user](/image-1.png)



## Reference Documents
[Deploy Microsoft Defender for Endpoint on Linux with Ansible](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/linux-install-with-ansible?view=o365-worldwide)


