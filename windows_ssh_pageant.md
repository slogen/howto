# Setting up Pageant auth for windows SSH

## Install

  * Download recent putty from: https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
  * Install

## Setup Pageant openssh-config thingy

Start your pageant to enable loading keys, but not decrypting them:

    pageant --encrypted "G:\My Drive\Keys\hej_ed25519.ppk" --openssh-config %HOMEDRIVE%%HOMEPATH%\pageant.conf
    
## Configure Windows OpenSSH

Make windows ssh include the pageant.conf

    echo Include %HOMEDRIVE%%HOMEPATH%\pageant.conf >>%HOMEDRIVE%%HOMEPATH%\.ssh\config
    
## Enable pubkey authentication

By default, pubkey authentication in the windows server is *disabled*???

Enable that by editing %PROGRAMDATA%\ssh\sshd_config:

    PubkeyAuthentication yes

You *may* also want to disable password-based authentication:

    PasswordAuthentication no

Restart the SSH server:

    net stop "openssh ssh server" && net start "openssh ssh server"

## Administrators

If your remote user has adminsitrator privileges, either remove the restriction for administrators in the config file, or add their public key to %PROGRAMDATA/ssh/administrators_authorized_keys

    #Match Group administrators
    #       AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys

