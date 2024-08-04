
# fedora40_btrfs_timeshift_setup



## fedora_custom_partitioning_while_installation
First you have to do some changes to the partition labels while installing fedora. While installing, you have to select "custom" in the partitioning section. Then select filesystem as "btrfs". Then create "efi partition" of size 512MB, "swap" of "RAM Size", "var" of 30GB, root(/) - your preferred size(I prefer 30GB - 100GB) and change the name of root from "/" to "@" and make home(/home) partition too and change the name from "/home" to "@home".
## dnf_configuration
After finishing your installation you have to change the dnf configuration in order to get better download speeds while updating the system.

Type the below command:

    sudo nano /etc/dnf/dnf.conf
Then add the following lines to the dnf configuration file:
    
    # Added for Speed:
    fastestmirror=True
    defaultyes=True
    max_parallel_downloads=5
    keepcache=True

Then save it by pressing "ctrl+o" then "enter" then "ctrl+x".

Now update your system

    sudo dnf update
    sudo dnf upgrade
Reboot the system after updating.
## install_timeshift
Install "timeshift" and some other packages which are necessary

    sudo dnf install timeshift make git inotify-tools
Open timeshift and select "btrfs" then select the storage hdd or ssd, then select the partitions to make snapshots of. Then create a snapshot in case if the system breaks this created snapshot saves you. To make the grub menu visible execute the below command in the terminal:

    sudo grub2-editenv - unset menu_auto_hide
## configure_grub-btrfs
Execute the below commands:

    mkdir git
    cd git
    git clone https://github.com/Antynea/grub-btrfs.git
    cd grub-btrfs
    sed -i 's/#GRUB_BTRFS_GRUB_DIRNAME=/GRUB_BTRFS_GRUB_DIRNAME=/' config
    sed -i '/#GRUB_BTRFS_MKCONFIG=/c\GRUB_BTRFS_MKCONFIG=/sbin/grub2-mkconfig' config
    sed -i 's/#GRUB_BTRFS_SCRIPT_CHECK=/GRUB_BTRFS_SCRIPT_CHECK=/' config
    sudo make install
Then create the grub configuration file by executing the below command:

    sudo grub2-mkconfig
You should get the output at the end like these below lines, as you created one snapshot after setting up timeshift it should show in the output something like "Found 1 snapshot". In my case I already setup snapshots using grub-btrfs and timeshift. so it is showing 5 snapshots.

    ### BEGIN /etc/grub.d/41_snapshots-btrfs ###
    Detecting snapshots ...
    Found snapshot: 2024-08-04 18:29:00 | timeshift-btrfs/snapshots/2024-08-04_18-29-00/@ | boot daily              | N/A |
    Found snapshot: 2024-08-04 13:00:06 | timeshift-btrfs/snapshots/2024-08-04_13-00-02/@ | boot                    | N/A |
    Found snapshot: 2024-08-03 16:48:18 | timeshift-btrfs/snapshots/2024-08-03_16-48-18/@ | ondemand weekly monthly | N/A |
    Found snapshot: 2024-08-03 11:17:31 | timeshift-btrfs/snapshots/2024-08-03_11-17-31/@ | ondemand                | N/A |
    Found snapshot: 2024-08-03 10:57:12 | timeshift-btrfs/snapshots/2024-08-03_10-57-12/@ | ondemand                | N/A |
    if [ ! -e "${prefix}/grub-btrfs.cfg" ]; then
    echo ""
    else
    submenu 'Fedora Linux snapshots' {
    configfile "${prefix}/grub-btrfs.cfg"
    }
    fi
    Found 5 snapshot(s)
    Unmount /tmp/grub-btrfs.9PTDY8EWx3 .. Success
    ### END /etc/grub.d/41_snapshots-btrfs ###
    done
By default, this setup watches the directory “/.snapshots” (the default directory for Snapper). Since Timeshift uses a different directory, we have to modify the configuration for the setup.
Execute the below command in terminal:

    sudo systemctl edit --full grub-btrfsd
change the below line

    ExecStart=/usr/bin/grub-btrfsd --syslog /.snapshots
into

    ExecStart=/usr/bin/grub-btrfsd --syslog --timeshift-auto
Start the grub-btrfsd service

    sudo systemctl start grub-btrfsd
To make it start at boot and show submenu for snapshots, execute the below command:

    sudo systemctl enable grub-btrfsd
Reboot the system and the changes will take effect. Create another snapshot and poweroff the system. Then poweron the system and see if all the created snapshots are shown in the snapshots submenu in the grub menu. If everything works enjoy.


