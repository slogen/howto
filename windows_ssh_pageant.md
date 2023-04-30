# Setting up Pageant auth for windows SSH

    
## Configure Windows OpenSSH

### Enable pubkey authentication

By default, pubkey authentication in the windows server is *disabled*???

Enable that by editing %PROGRAMDATA%\ssh\sshd_config:

    PubkeyAuthentication yes

You *may* also want to disable password-based authentication:

    PasswordAuthentication no

### Administrators

If your remote user has adminsitrator privileges, either remove the restriction for administrators in the config file, or add their public key to %PROGRAMDATA/ssh/administrators_authorized_keys

    #Match Group administrators
    #       AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys

### Restart the SSH server:

    net stop "openssh ssh server" && net start "openssh ssh server"


## Install putty & pageant

  * Download recent putty from: https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
  * Install

### Setup Pageant openssh-config thingy

Start your pageant to enable loading keys, but not decrypting them:

    pageant --encrypted "G:\My Drive\Keys\hej_ed25519.ppk" --openssh-config %HOMEDRIVE%%HOMEPATH%\pageant.conf

### Include pageant generated ssh config

Make windows ssh include the pageant.conf

    echo Include %HOMEDRIVE%%HOMEPATH%\pageant.conf >>%HOMEDRIVE%%HOMEPATH%\.ssh\config
    
### Automatic start

Use "Windows-R" to run "shell:startup", which will open the windows startup folder (which is now well hidden???)

Add a shortcut here to pageant, adding the args from above into "TARGET":

    pageant --encrypted "G:\My Drive\Keys\hej_ed25519.ppk" --openssh-config %HOMEDRIVE%%HOMEPATH%\pageant.conf
