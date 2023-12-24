# Project to Learn how to Use Ansible

## Background
I wanted to learn more about how to use ansible as an automation utility, so I decided to take a simple project which has a well documented installation procedure, and to automate it to the max extent practical.  Along the way I wanted to learn better ways to leverage Ansible to help me not only in my homelab, but in my professional life.

## Goals of the Project
- [] Never have to actually login to the the rpi via SSH and run any command from the ternminal
- [] Install Pi-hole via ansible on a Raspberry Pi 3, that has raspian installed
- [] Configure the rpi to be a recurrsive DNS

## What I learned along the way

### Pi-hole installation process
The Pi-hole installatino process is fairly straight forward.  All you have to do is run `sudo curl -sSL https://install.pi-hole.net | bash` and it downloads and kicks off the installation process.  However when did that with Ansible I ran into my first road block.  The install script is not designed to run in an unattended method.  Specifically, it loads up a handful of dialog boxes for user interaction to select various settings.  After looking through the install script I noted the following:

```
reconfigure=false
runUnattended=false
INSTALL_WEB_SERVER=true
# Check arguments for the undocumented flags
for var in "$@"; do
    case "$var" in
        "--reconfigure" ) reconfigure=true;;
        "--unattended" ) runUnattended=true;;
        "--disable-install-webserver" ) INSTALL_WEB_SERVER=false;;
    esac
done
```

At first glance I thought I could just run the install script with the flag `--unattended`, but that did not change the installation at all.  After further review I noted that the `setVars.conf` file needed to be placed down in the required location and the `--unattended` flag set.  The following code snippet shows that if the `setVars` already exists **and** the `runUnattended` variable is tru then it will disable the user interaction portions of the script

```
    # If the setup variable file exists,
    if [[ -f "${setupVars}" ]]; then
        # if it's running unattended,
        if [[ "${runUnattended}" == true ]]; then
            printf "  %b Performing unattended setup, no dialogs will be displayed\\n" "${INFO}"
            # Use the setup variables
            useUpdateVars=true
            # also disable debconf-apt-progress dialogs
            export DEBIAN_FRONTEND="noninteractive"
        else
            # If running attended, show the available options (repair/reconfigure)
            update_dialogs
        fi
    fi
```

### Running Ansible to install Pi-hole in an unattended mode
I noted a couple of things that needed to be accomplished prior to starting the installation.  

1. I needed to create the pihole user and group
1. I needed to create the `/etc/pihole` directory
1. I needed to create and populate the `setupVars.conf` file within the `/etc/pihole` directory
1. The directory and file needed to have the correct owner/group/permissions

Enter Ansible to the rescue.  

the `install-pihole-playbook.yaml` 


# Ip-Tables
Here is some setup that I foudn was needed along the way, that assits in performance


