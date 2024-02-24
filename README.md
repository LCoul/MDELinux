<details>
    <summary><b>Deploy Manually</b></summary>
1. Create your Linux virtual machines in MyWorkspace
2. From a PowerShell session, connect to a Linux VM (distro = RHEL in this case)
    a. Enter the following ssh rod@mw-072.myworkspace.microsoft.com -p 45630 and press enter. Then enter "yes" and provide your password when prompted.
    
    
    b. sudo yum update to get the update packages
    c. sudo yum upgrade
    d. Create a user (not really needed - this is to just create a user with a password that you can easily remember) and add the user to the wheel (sudo) group - you can do this on multiple lines or a single line.
    e. Type sudo -i to switch to the root user.
        i. Multiple lines of commands to create a user:
        adduser bob[to create new user]
        passwd bob[to configure a password for the user]
        usermod -aG wheel bob[to make user a sudo user]
        id bob[to verify user sudo status]
        su - bob[to log in as bob]
        pwd [to view the user's working directory]
        
        Or
        
        ii. Single line of command: useradd bob && passwd bob && usermod -aG wheel bob
        
        Now, you can connect to your Linux device using the new user's (bob) credentials with the following line for example: ssh bob@mw-072.myworkspace.microsoft.com -p 45630
        iii. But even better, do you really want to be bother using a password to authenticate? I am guessing no; so, access your Linux device in a very secure manner with a certificate-based authentication!
        
        On your local device (Microsoft issued or other), do the following from a PowerShell session:
        
        Generate a private/public key pair and provide the name LocalHostKey for example when prompted and do not provide any password (two files will be created, one for the private key and one for the public key)
        ssh-keygen -t rsa -C "LocalHost" 
        
        Replace filePath with the path to the file, for example "E:\Repo\MDE\LocalHostKey" and the command will be $keyFile = "E:\Repo\MDE\LocalHostKey"
        $keyFile = "filePath"
        
        Run the following command and note FullControl access for System and Administrators, and Modify and Synchronize for the current user, which are overly permissive, and Linux will not allow authentication with such permissions.
        Get-Acl $keyFile | Format-List
        
        Get the permissions that users and user groups have to access the file
        $acl = Get-Acl $keyFile 
        
        Get the current username on the device
        $username = [System.Security.Principal.WindowsIdentity]::GetCurrent().Name
        
        Create a new access rule object with the permissions for the ACL and apply the ACL to the file
        $accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule($username,"Read","Allow")
        $acl.SetAccessRule($accessRule)
        $acl | Set-Acl $keyFile
        
        Disabling the inheritance and removing the existing access rules
        $acl.SetAccessRuleProtection($true,$false) # $acl | Set-Acl $keyFile
        
        # After applying the ACL and disabling the inheritance, make sure FullControl is no longer granted to the current user
        Get-Acl $keyFile | Format-List
        
        # Finally copy the public key, you'll upload that to your Linux device
        Get-Content .\LocalHostKey.pub and copy the public key; you'll upload that to your Linux machine
        
        On your Linux machine
        mkdir ~/.ssh
        sudo vim ~/.ssh/authorized_keys
        Type i and paste the public key
        Type "ESC" then :wq to exit
        cat ~/.ssh/authorized_keys to verify the presence of the public key on the Linux machine.
    
    Now you can connect to your Linux device without a password - example:
    ssh -i "LocalHostKey" bob@mw-072.myworkspace.microsoft.com -p 45173
    From the current system, you can also copy the public key to other systems with the following command for example:
    sudo scp ~/.ssh/authorized_keys lessi@10.0.0.78:~/.ssh
    
3. Install MDE
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
    <summary><b>Deploy with a Script</b></summary>
</details>

<details>
    <summary><b>Deploy with Ansible</b></summary>

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


