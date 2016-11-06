# freeswitch-fcc-blacklist: automatically add entries from the FCC robocall blacklist to FreeSWITCH
[mod_blacklist](https://freeswitch.org/confluence/display/FREESWITCH/mod_blacklist) sets up a database of blacklisted numbers for FreeSWITCH systems. Using dialplan logic, blacklisted numbers can be sent to alternate destinations outside of your standard call flow. Maintaining a large, up-to-date system blacklist can significantly reduce the number of spam and other unwanted calls that reach your system.

The [Federal Communications Commission](http://fcc.gov) [publishes data on consumer complaints](http://opendata.fcc.gov) each week. Ward Mundy [has written a script](http://nerdvittles.com/?p=19477) to convert this FCC consumer complaint data to a blacklist for the [Asterisk](http://asterisk.org) PBX.

I have adapted Ward's script for the FreeSWITCH blacklist module. My adaptation is compatible with both standard installs of FreeSWITCH and those that use the [Fusion PBX](http://fusionpbx.com) GUI. It is compatible with both standard and Debian configuration directory structures. After initial configuration, the script runs non-interactively; you may want to set up a `Cron` or `Systemd` job to automate blacklist updates.

## Installation
### Install mod_blacklist
YOu will first need to install mod_blacklist on your system. If you've installed FreeSWITCH from source, run `make` then `make install` from `mod/app/blacklist` in your source directory. If you've installed FreeSWITCH from Debian packages, run `apt install freeswitch-mod-blacklist` as root. If not already present, add the following line to `/usr/local/freeswitch/conf/autoload_configs/modules.conf.xml` (standard installs) or `/etc/freeswitch/autoload_configs/modules.conf.xml` (Debian systems) somewhere between the `<modules>` and `</modules>` tags:

    <load module="mod_blacklist"/>

### Obtaining the Script
Clone this script's `git` repository with the following commands:

    cd /usr/src
    git clone https://github.com/codeofdusk/freeswitch-fcc-blacklist

### Generate Configuration
When the script is ran for the first time, it will attempt to detect your system's configuration and walk you through the modification of some configuration files. Run the script with the following commands:

    cd /usr/src/freeswitch-fcc-blacklist
    chmod +x import-fcc-blacklist
    ./import-fcc-blacklist

### Add Dialplan Logic
Add an extension to your FreeSWITCH dialplan to handle blacklisted numbers. It should be added before your standard call flow. If you are using Fusion PBX, add an inbound route with your DID as the destination, your alternate destination for blacklisted callers as the action and an order less than your standard inbound route. Click save and reopen your inbound route. Add a new line, with tag condition, data `^true$`, order 1 and the following type:

    ${blacklist(check FCC ${regex(${caller_id_number}|^\+([0-9]+)$|%1)})}

If you configure freeSWITCH by hand, create a new extension in your dialplan before your call flow, with the following line:

    <condition field="${blacklist(check FCC ${regex(${caller_id_number}|^\+([0-9]+)$|%1)})}" expression="^true$" >

### Optional: Automate Blacklist Updates
You may wish to create a `Cron` or `Systemd` job to automate blacklist updates. To add a `cron` job to update weekly, run the following command:

    echo "$(($RANDOM%60)) $(($RANDOM%24)) * * $(($RANDOM%2 +6 )) root /usr/src/freeswitch-fcc-blacklist/import-fcc-blacklist > /dev/null" >> /etc/crontab
