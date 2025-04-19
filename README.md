# Prerequisites
* Ensure your OCEAN membership is current ;-)  
* Get your IBM i credentials from OCEAN


# Installs

* Download & install Java - https://www.java.com/en/download/manual.jsp  
    * OpenJDK works too, if you prefer not to use anything from Oracle

* Download & install ACS - http://ibm.biz/IBMi_ACS  
    * Requires an IBM ID (easy to do and mostly painless)

* Download & install Git for Windows - https://git-scm.com/download/win  

* Download & install Visual Studio Code - https://code.visualstudio.com/download  


# Setting up ACS
* Open ACS
* Go to **Management > System Configurations**  
    ![System Configurations](/images/system_configurations.png)  
* Click the `New` button in the system configurations window  
* In the **General** tab:  
    * System Name: `OCSKUNKS.oceanusergroup.org`  
    * Description: `OCSKUNKS direct connection`  
    * Use SSL: `checked`  
    ![General Tab](/images/general_tab.png)  

* In the **Connection** tab:  
    * Pasword Prompting: `Use default user name to prompt once for each system`  
    * Default user name: *Your IBM i user profile*  
    * IP address lookup: `One day`  
    ![Connection Tab](/images/connection_tab.png)  

* Verify the connection:   
    * Go back to the **General** tab and click the `Verify` button:   
    ![Verify Connection](/images/verify.png)  

    * If this is the first time you connect, you will need to add the OCSKUNKS CA Certificate:   
    ![Accept Certificate](/images/accept_certificate.png)  
 
    * You should see this:  
    ![Verify results](/images/results.png)  


# Setting up SSH  

### Generate local SSH key  
* Press Ctrl+\` to open the terminal inside Visual Studio Code (displays at the bottom of the screen).  
    > The \` should be the key above the Tab key  

* Enter the following command to change your current working directory to your home:  
    `cd ~`  

* Print your current working directory with this command:  
    `pwd`  
    > This should print something like `c/Users/YourWindowsUsername`  

* Check to see if you already have an SSH key using this command:  
    `cat .ssh/id_ed25519.pub`  


    If you see this error, you will need to generate a key:  
    `cat: .ssh/id_ed25519.pub: No such file or directory`  

    * Change directory into the .ssh folder with the following command:
    `cd .ssh`

    ⚠️ If you see this error, you will need to create the .ssh directory:
    `cd: .ssh: No such file or directory`
        * Create the .ssh directory:
        `mkdir .ssh`
        * Then change directory into the .ssh folder:
        `cd .ssh`
      
    * Enter the following command to generate an SSH key:  
    `ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519`   
    Press Enter twice to leave the passphrase blank  

        > `ssh-keygen` is the command to run (generate key)  
        > `-t ed25519` specifies the type of key to create  
        > `-f ~/.ssh/id_ed25519` specifies the filename of the generated key  

### Copy your public SSH key to IBM i  
⚠️ Be sure you copy your ***public*** key (should end with `.pub`)  

#### Using ACS  
Open System Configurations, highlight your system, and press Edit.  
Select the SSH Key setup tab, and press the "Copy SSH key(s) to server" button.  
![SSH Key Setup](/images/ssh_key_setup.png)

Select the `id_ed25519.pub` key  

#### Using SQL  

Copy the public key to your clipboard:  
* Run the `cat ~/.ssh/id_ed25519.pub` command to view the public key  
Copy the public key to your clipboard  

Use SQL to add the public key

```
CALL QSYS2.IFS_WRITE_UTF8(
    PATH_NAME =>'/home/{IBM i Profile}/.ssh/authorized_keys',
    LINE => '{public_key_from_clipboard}',
    OVERWRITE => 'APPEND',
    END_OF_LINE => 'LF'
);
```
⚠️ Be sure to change `{IBM i Profile}` to *your* user profile.  
⚠️ Be sure to paste your public key here `{public_key_from_clipboard}`.  


# Set permissions 

In the terminal, enter the following commands:  
`chmod 755 ~`  
`chmod 700 ~/.ssh`  
`chmod 600 ~/.ssh/id_ed25519`  
`chmod 600 ~/.ssh/id_ed25519.pub`  


Open a green-screen terminal, and use `QSH` to run the following commands:  
`chmod 755 ~`  
`chmod 700 ~/.ssh`  
`chmod 600 ~/.ssh/authorized_keys`  


# Set up Visual Studio Code

### Set the default shell  
Open Visual Studio Code.  
Press F1 to open the command prompt (displays at the top of the screen).  
Type "Terminal: Select Default Profile" in the search bar and press Enter.  
Select "Git Bash" from the drop-down list.  

### Install Code for IBM i  
Open the extensions tab and install the "IBM i Development Pack" extension.  

### Add a connection 
Open the IBM i activity panel.  
Click the "+" in the SERVERS tab.  
Add your connection:  
* Connection Name: *Can be whatever makes sense to you*
* Host or IP Address: `OCSKUNKS.oceanusergroup.org`  
* Port: *enter the SSH port number here*  
* Username: *your IBM i user profile*  
* Private Key: Press the "Select File" button and find your `id_ed25519` key  
    ⚠️ This time you select your ***private*** key (should end NOT with `.pub`)  
![Connection Settings](/images/new_connection.png)  
