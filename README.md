<details>
    <summary><b>Deploy Manually: RedHat Server</b></summary>

### 1. Connect to the server
From a Terminal session, connect to a Linux VM using the command: **_ssh <user>@<ip_address>_** or **_ssh <user>@<ip_address> -p <port_number>_** if you are connecting to a port other then TCP port 22. The 'IP address' can also be the FQDN of the server you are connecting to.
```bash
ssh <user>@<ip_address>
```
or
```bash
ssh <user>@<ip_address> -p <port_number>
```
Press enter. Then answer "yes" and provide your password when prompted.
    
### 2. Update the server
sudo yum update && sudo yum upgrade
### 3. Create a user 
The user will be added the user to the 'wheel' group, so the user can manage the server.<br>
This step is not really needed. But this is to avoid login onto the server as root. You can do this will multiple lines of commands or a single line of command.
#### Create a user with a series of commands
Switch to the root user.
```bash
sudo -i
```
Create the user and set the user's home directory with '-m'
```bash
adduser -m bob
```
Configure the user's password
```bash
passwd bob
```
Add the user to the 'wheel' (sudo) group
```bash
usermod -aG wheel bob
```
Verify the user belongs to the 'wheel' group
```bash
id bob
```
Login as the new user
```bash
su - bob
```
View the user's working directory
```bash
pwd
```
or

#### Create a user with a single line 
```bash
sudo useradd -m bob && sudo passwd bob && usermod -aG wheel bob
```
Now, you can connect to your Linux device using the new user's (bob) credentials:
```bash
ssh bob@<ip_address>
```
Certificate-based authentication is also an option: Example of a Windows device with PowerShell<br>
On your local device (Windows), do the following from a PowerShell session:
Generate a private/public key pair and provide the name LocalHostKey for example when prompted and do not provide any password (two files will be created, one for the private key 'LocalHostKey' and one for the public key 'LocalHostKey.pub').
```PowerShell
ssh-keygen -t rsa -C "LocalHost" -f LocalHostKey
```
Create a variable to hold the location of the private key, for example:
```PowerShell
$keyFile = "E:\Repo\MDE\LocalHostKey"
```
Run the following command and note FullControl access for System and Administrators, and Modify and Synchronize for the current user, which are overly permissive, and a Linux system will not allow authentication with such permissions.
```PowerShell
Get-Acl $keyFile | Format-List
```    
Get the permissions that users and user groups have to access the file
```PowerShell
$acl = Get-Acl $keyFile 
```      
Get the current username on the device
```PowerShell
$username = [System.Security.Principal.WindowsIdentity]::GetCurrent().Name
```
        
Create a new access rule object with the permissions for the ACL and apply the ACL to the file
```PowerShell
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule($username,"Read","Allow")
$acl.SetAccessRule($accessRule)
$acl | Set-Acl $keyFile
```  
Disable the inheritance and remove the existing access rules
```PowerShell
$acl.SetAccessRuleProtection($true,$false)
$acl | Set-Acl $keyFile
```  
After applying the ACL and disabling the inheritance, make sure FullControl is no longer granted to the current user
```PowerShell
Get-Acl $keyFile | Format-List
```     
Finally copy the public key, you'll upload that to your Linux device
```PowerShell
Get-Content .\LocalHostKey.pub
```
        
On your Linux machine
```bash
mkdir ~/.ssh
sudo vim ~/.ssh/authorized_keys
```
Type 'i' and paste the public key<br>
Type 'ESC' then ':wq' to exit

Verify the presence of the public key on the Linux machine with the following command:
```bash
cat ~/.ssh/authorized_keys
``` 
Now you can connect to your Linux device without a password:
```PowerShell
ssh -i "LocalHostKey" bob@<ip_address>
```

    From the current system, you can also copy the public key to other systems with the following command for example:
    sudo scp ~/.ssh/authorized_keys lessi@10.0.0.78:~/.ssh
    
### 4. Install MDE
    a. Locate the installer script
        i. Use hostnamectl command to identify system related information including release version.
        ii. Install yum-utils if it isn't already installed: sudo yum install yum-utils
        iii. sudo yum-config-manager --add-repo=https://packages.microsoft.com/config/rhel/9.0/prod.repo
        
    b. Application installation
        i. yum repolist to list all repositories
        ii. sudo yum --enablerepo=packages-microsoft-com-prod install mdatp to install the package from the production repository.
        iii. sudo mdatp edr tag set --name GROUP --value 'Rhel-Linux' to set the device tag.
        
        
        
    c. Download the onboarding package from Microsoft Defender XDR portal
        i. Create a folder to store MDE onboarding files: mkdir MDE and cd MDE to navigate in that directory
        ii. Transfer the onboarding package to your Linux machine: 
    
    
    In Linux, we can share files between computers using scp. scp utilizes ssh to securely transfer files. We use the following syntax to copy files from the source machine to the destination machine:
     scp /path/to/local/file username@destination:/path/to/destination, for example the below command will copy the onboarding package from your local computer into the MDE directory of the Linux device.
     scp -P 45173 "E:\Repo\MDE\WindowsDefenderATPOnboardingPackage.zip" bob@devlab-rhelz:/MDE
    
    
    On the Linux machine, type ls -l MDE (this LS in lowercase) in to verify the presence of the onboarded ZIP file
    cd MDE and unzip WindowsDefenderATPOnboardingPackage.zip to unzip the file. You'll get the MicrosoftDefenderATPOnboardingLinuxServer.py file

    a. Client configuration
    Initially the client device is not associated with an organization and the orgId attribute is blank.
    mdatp health --field org_id 
    sudo python3 MicrosoftDefenderATPOnboardingLinuxServer.py (Generating /etc/opt/microsoft/mdatp/mdatp_onboard.json ..)
    mdatp health --field org_id to verify that the device is now associated with your organization and reports a valid organization identifier.
    
    Check the health status of the product. A return value of 'true' denotes that the product is functioning as expected.
    mdatp health --field healthy
    mdatp health list | grep -i 'network\|passive_mode\|automatic_definition\|managed_by\|MDE\|managed\|real_time_protection\|behavior_monitoring\|edr'
    
    Check the status of the definitions update, return value should be up_to_date.
    mdatp health --field definitions_status
    
    Ensure real-time protection is enabled, the return value should be true.
    mdatp health --field real_time_protection_enabled
    if not, run the following: sudo mdatp config real-time-protection --value enabled
    
    Test MDE on Linux by simulating the download of a malicious file. The file should be quarantined.
    curl -o ~/eicar.com.txt https://secure.eicar.org/eicar.com.txt
    
    List the detected threats
    mdatp threat list
    
    
    https://aka.ms/LinuxDIY
    
    
    
Resources: Microsoft Defender for Endpoint on Linux resources | Microsoft Learn

</details>

<details>
    <summary><b>Deploy with a Script: RedHat Server</b></summary>
</details>

<details>
    <summary><b>Deploy with Ansible: Ubuntu Servers</b></summary>

### Connect to Ansible Control Node
From a shell (for example PowerShell), connect to your Ansible control node server with the following command:<br> _<**ssh rod@IPAddress -p 45163**>_<br>
The IPAddress could also be the FQDN of the server, **-p** specifies the ssh port if TCP port 22 is not the default. Answer 'yes' when prompted if you are sure to continue connecting, and provide the login password when prompted.<br>
```PowerShell
ssh rod@IPAddress -p 45163
```

### Configure Ansible Control Node
#### Basic Configurations
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
#### Install Ansible
```bash
ansible-playbook -K install_mdatp.yml -i hosts
```
![Install Ansible](/)

#### Uninstall Ansible
```bash
ansible-playbook -K uninstall_mdatp.yml -i hosts
```
![Uninstall Ansible](/)

### Configure Ansible Managed Nodes

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

### Reference Documents
[Deploy Microsoft Defender for Endpoint on Linux with Ansible](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/linux-install-with-ansible?view=o365-worldwide)<br>
[Installing Ansible - Ansible Documentation](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)


