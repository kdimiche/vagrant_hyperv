# vagrant_hyperv
Notes for configuring Windows 10 Fall Creators with Vagrant using the Hyper-V (`hyperv`) provider and synced folders


## Environment Variables
Set user environemnt variable for vagrant default provider

### User

```PowerShell
[Environment]::SetEnvironmentVariable("VAGRANT_DEFAULT_PROVIDER", "hyperv", "User")
[Environment]::SetEnvironmentVariable("VAGRANT_SMB_PASSWORD", "<USER_NAME>", "User") 
[Environment]::SetEnvironmentVariable("VAGRANT_SMB_PASSWORD", '<PASSWORD>', "User") 
```

### Per PowerShell Session:

```PowerShell
$env:VAGRANT_DEFAULT_PROVIDER = "hyperv"
```

## Vagrant Synced Folders
Out of the box, I wasn't able to get synced folders working.
After research into SMB, CIFS, Firewall settings and user accounts, I determined this process to get the synced folders working:

* Enable the Windows Feature for Hyper-V
* [Create new Hyper-V `Virtual Switch` named `External Switch`](https://docs.microsoft.com/en-us/windows-server/virtualization/hyper-v/get-started/create-a-virtual-switch-for-hyper-v-virtual-machines)
* Check user account type
 * If the output of `get-localuser <USER_NAME> | fl *`  contains `PrincipalSource: MicrosoftAccount`, create a new local user account.
* Set environment variables (see above)
* Update Vagrantfile with
```
    config.vm.synced_folder '.', '/vagrant', {
      type: 'smb',
      mount_options: [
        'vers=3.0',
      ],
      # Specify IP Address of Hyper-V Virtual Switch named `External` if multiple NICs are found.
      # smb_host: "192.168.1.4",
      smb_username: ENV['VAGRANT_SMB_USERNAME'],
      smb_password: ENV['VAGRANT_SMB_PASSWORD']
    }
```
* `vagrant up`
* select `2) External Switch`
* Update the Share `Security` to allow local user access to the share.
 * Vagrant sets up the share and grants the `Share Permissions` tp `Everyone` with `Full Control`)
* `vagrant reload`


If the user/password isn't a local user with access to the share:

```
PS C:\Users\kevin\projects\CentOS7> vagrant reload
==> default: Attempting graceful shutdown of VM...
    default: Configuring the VM...
==> default: Starting the machine...
==> default: Waiting for the machine to report its IP address...
    default: Timeout: 120 seconds
    default: IP: 192.168.1.13
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 192.168.1.13:22
    default: SSH username: vagrant
    default: SSH auth method: private key
==> default: Machine booted and ready!
==> default: Preparing SMB shared folders...

Vagrant requires administrator access to create SMB shares and
may request access to complete setup of configured shares.
==> default: Mounting SMB shared folders...
    default: C:/Users/kevin/projects/CentOS7 => /vagrant
Failed to mount folders in Linux guest. This is usually because
the "vboxsf" file system is not available. Please verify that
the guest additions are properly installed in the guest and
can work properly. The command attempted was:

mount -t cifs -o vers=3.0,credentials=/etc/smb_creds_vgt-e2b92efdf442f452a0623dcccc6738d4-6ad5fdbcbf2eaa93bd62f92333a2e6e5,uid=1000,gid=1000,sec=ntlmssp,iocharset=utf8,mapchars,noperm,soft //192.168.1.4/vgt-e2b92efdf442f452a0623dcccc6738d4-6ad5fdbcbf2eaa93bd62f92333a2e6e5 /vagrant

mount error(13): Permission denied
Refer to the mount.cifs(8) manual page (e.g. man mount.cifs)
```

After granting Share Security accesss:

```
PS C:\Users\kevin\projects\UbuntuBionic64> vagrant up; vagrant reload
Bringing machine 'default' up with 'hyperv' provider...
==> default: Verifying Hyper-V is enabled...
==> default: Verifying Hyper-V is accessible...
==> default: Importing a Hyper-V instance
    default: Creating and registering the VM...
    default: Successfully imported VM
    default: Please choose a switch to attach to your Hyper-V instance.
    default: If none of these are appropriate, please open the Hyper-V manager
    default: to create a new virtual switch.
    default:
    default: 1) DockerNAT
    default: 2) External Switch
    default: 3) Default Switch
    default:
    default: What switch would you like to use? 2
    default: Configuring the VM...
==> default: Starting the machine...
==> default: Waiting for the machine to report its IP address...
    default: Timeout: 120 seconds
    default: IP: 192.168.1.16
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 192.168.1.16:22
    default: SSH username: vagrant
    default: SSH auth method: private key
    default: Warning: Connection aborted. Retrying...
    default: Warning: Connection reset. Retrying...
    default:
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default:
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH key...
==> default: Machine booted and ready!
==> default: Preparing SMB shared folders...

Vagrant requires administrator access to create SMB shares and
may request access to complete setup of configured shares.
==> default: Mounting SMB shared folders...
    default: C:/Users/kevin/projects/UbuntuBionic64 => /vagrant
==> default: Attempting graceful shutdown of VM...
    default: Configuring the VM...
==> default: Starting the machine...
==> default: Waiting for the machine to report its IP address...
    default: Timeout: 120 seconds
    default: IP: 192.168.1.16
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 192.168.1.16:22
    default: SSH username: vagrant
    default: SSH auth method: private key
==> default: Machine booted and ready!

Vagrant requires administrator access for pruning SMB shares and
may request access to complete removal of stale shares.
==> default: Preparing SMB shared folders...

Vagrant requires administrator access to create SMB shares and
may request access to complete setup of configured shares.
==> default: Mounting SMB shared folders...
    default: C:/Users/kevin/projects/UbuntuBionic64 => /vagrant
==> default: Machine already provisioned. Run `vagrant provision` or use the `--provision`
==> default: flag to force provisioning. Provisioners marked to run always will still run.
```
