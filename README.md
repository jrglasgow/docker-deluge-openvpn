# OpenVPN and Deluge with Deluge Web

This container contains OpenVPN and Deluge with a configuration
where Deluge is running only when OpenVPN has an active tunnel.
It bundles configuration files for many popular VPN providers to make the setup easier.

Based wholy on the work of transmission with openvpn by haugene at https://github.com/haugene/docker-transmission-openvpn
Note also this is partially complete so your mileage may vary


You will need to specify your provider and credentials with environment variables,
as well as mounting volumes where the data should be stored.
An example run command to get you going is provided below.

Also worth mentioning.
If you want to route web traffic through the same tunnel that Deluge is using there
is a pre-installed Tinyproxy which will expose a proxy on port 8888 when enabled.
And if you're using PIA as provider it will update Deluge hourly with assigned open port - TBC

GL HF! And if you run into problems, please check the README twice and maybe try the gitter chat before opening an issue :)

### about:maintenance

This image was created for my own use, but sharing is caring so it had to be open source.
The number of users, issues and pull-requests have gone up quite drastically since that
and that's great! It's been a lot of fun watching the activity level go up
and my pet project improve with it.

Maintaining this takes time, so like donating, please donate to the originating author, here's a button:

[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=73XHRSK65KQYC)

You can also help out by submitting pull-requests or helping others with
open issues or in the gitter chat. A big thanks to everyone who has contributed so far!
And if you could be interested in joining as collaborator, let me know.


## Run container from Github
To run the container use this command:

```
$ docker build --cap-add=NET_ADMIN --device=/dev/net/tun -d \
              -v /your/storage/path/:/data \
              -v /etc/localtime:/etc/localtime:ro \
              -e OPENVPN_PROVIDER=PIA \
              -e OPENVPN_CONFIG=Netherlands \
              -e OPENVPN_USERNAME=user \
              -e OPENVPN_PASSWORD=pass \
              -e WEBPROXY_ENABLED=false \
              -e LOCAL_NETWORK=192.168.0.0/16 \
              --log-driver json-file \
              --log-opt max-size=10m \
              -p 9091:9091 \
              uri:https://github.com/sscraggles/docker-deluge-openvpn/
```

You must set the environment variables `OPENVPN_PROVIDER`, `OPENVPN_USERNAME` and `OPENVPN_PASSWORD` to provide basic connection details.

The `OPENVPN_CONFIG` is an optional variable. If no config is given, a default config will be selected for the provider you have chosen.
Find available OpenVPN configurations by looking in the openvpn folder of the GitHub repository. The value that you should use here is the filename of your chosen openvpn configuration *without* the .ovpn file extension. For example:

```
-e "OPENVPN_CONFIG=ipvanish-AT-Vienna-vie-c02"
```

You can also provide a comma separated list of openvpn configuration filenames.
If you provide a list, a file will be randomly chosen in the list, this is useful for redundancy setups. For example:
```
-e "OPENVPN_CONFIG=ipvanish-AT-Vienna-vie-c02,ipvanish-FR-Paris-par-a01,ipvanish-DE-Frankfurt-fra-a01"
```
If you provide a list and the selected server goes down, after the value of ping-timeout the container will be restarted and a server will be randomly chosen, note that the faulty server can be chosen again, if this should occur, the container will be restarted again until a working server is selected.  

To make sure this work in all cases, you should add ```--pull-filter ignore ping``` to your OPENVPN_OPTS variable.  

As you can see, the container also expects a data volume to be mounted.
This is where Transmission will store your downloads, incomplete downloads and look for a watch directory for new .torrent files.
By default a folder named transmission-home will also be created under /data, this is where Transmission stores its state.

### Supported providers

This is a list of providers that are bundled within the image. Feel free to create an issue if your provider is not on the list, but keep in mind that some providers generate config files per user. This means that your login credentials are part of the config an can therefore not be bundled. In this case you can use the custom provider setup described later in this readme. The custom provider setting can be used with any provider.

| Provider Name                | Config Value (`OPENVPN_PROVIDER`) |
|:-----------------------------|:-------------|
| Anonine | `ANONINE` |
| AnonVPN | `ANONVPN` |
| BlackVPN | `BLACKVPN` |
| BTGuard | `BTGUARD` |
| Cryptostorm | `CRYPTOSTORM` |
| Cypherpunk | `CYPHERPUNK` |
| FrootVPN | `FROOT` |
| FrostVPN | `FROSTVPN` |
| Giganews | `GIGANEWS` |
| HideMe | `HIDEME` |
| HideMyAss | `HIDEMYASS` |
| IntegrityVPN | `INTEGRITYVPN` |
| IPredator | `IPREDATOR` |
| IPVanish | `IPVANISH` |
| Ivacy | `IVACY` |
| IVPN | `IVPN` |
| Mullvad | `MULLVAD` |
| Newshosting | `NEWSHOSTING` |
| NordVPN | `NORDVPN` |
| OVPN | `OVPN` |
| Perfect Privacy | `PERFECTPRIVACY` |
| Private Internet Access | `PIA` |
| PrivateVPN | `PRIVATEVPN` |
| proXPN | `PROXPN` |
| PureVPN | `PUREVPN` |
| RA4W VPN | `RA4W` |
| SaferVPN | `SAFERVPN` |
| SlickVPN | `SLICKVPN` |
| Smart DNS Proxy | `SMARTDNSPROXY` |
| SmartVPN | `SMARTVPN` |
| TigerVPN | `TIGER` |
| TorGuard | `TORGUARD` |
| Trust.Zone | `TRUSTZONE` |
| TunnelBear | `TUNNELBEAR`|
| UsenetServerVPN | `USENETSERVER` |
| Windscribe | `WINDSCRIBE` |
| VPNArea.com | `VPNAREA` |
| VPN.AC | `VPNAC` |
| VPN.ht | `VPNHT` |
| VPNBook.com | `VPNBOOK` |
| VPNTunnel | `VPNTUNNEL` |
| VyprVpn | `VYPRVPN` |

### Required environment options
| Variable | Function | Example |
|----------|----------|-------|
|`OPENVPN_PROVIDER` | Sets the OpenVPN provider to use. | `OPENVPN_PROVIDER=provider`. Supported providers and their config values are listed in the table above. |
|`OPENVPN_USERNAME`|Your OpenVPN username |`OPENVPN_USERNAME=asdf`|
|`OPENVPN_PASSWORD`|Your OpenVPN password |`OPENVPN_PASSWORD=asdf`|

### Network configuration options
| Variable | Function | Example |
|----------|----------|-------|
|`OPENVPN_CONFIG` | Sets the OpenVPN endpoint to connect to. | `OPENVPN_CONFIG=UK Southampton`|
|`OPENVPN_OPTS` | Will be passed to OpenVPN on startup | See [OpenVPN doc](https://openvpn.net/index.php/open-source/documentation/manuals/65-openvpn-20x-manpage.html) |
|`LOCAL_NETWORK` | Sets the local network that should have access. Accepts comma separated list. | `LOCAL_NETWORK=192.168.0.0/24`|

### Firewall configuration options
When enabled, the firewall blocks everything except traffic to the peer port and traffic to the rpc port from the LOCAL_NETWORK and the internal docker gateway.

If TRANSMISSION_PEER_PORT_RANDOM_ON_START is enabled then it allows traffic to the range of peer ports defined by TRANSMISSION_PEER_PORT_RANDOM_HIGH and TRANSMISSION_PEER_PORT_RANDOM_LOW.

| Variable | Function | Example |
|----------|----------|-------|
|`ENABLE_UFW` | Enables the firewall | `ENABLE_UFW=true`|
|`UFW_ALLOW_GW_NET` | Allows the gateway network through the firewall. Off defaults to only allowing the gateway. | `UFW_ALLOW_GW_NET=true`|
|`UFW_EXTRA_PORTS` | Allows the comma separated list of ports through the firewall. Respsects UFW_ALLOW_GW_NET. | `UFW_EXTRA_PORTS=9910,23561,443`|

### Web proxy configuration options

This container also contains a web-proxy server to allow you to tunnel your web-browser traffic through the same OpenVPN tunnel.
This is useful if you are using a private tracker that needs to see you login from the same IP address you are torrenting from.
The default listening port is 8888. Note that only ports above 1024 can be specified as all ports below 1024 are privileged 
and would otherwise require root permissions to run.

| Variable | Function | Example |
|----------|----------|-------|
|`WEBPROXY_ENABLED` | Enables the web proxy | `WEBPROXY_ENABLED=true`|
|`WEBPROXY_PORT` | Sets the listening port | `WEBPROXY_PORT=8888` |

### User configuration options

By default everything will run as the root user. However, it is possible to change who runs the transmission process.
You may set the following parameters to customize the user id that runs transmission.

| Variable | Function | Example |
|----------|----------|-------|
|`PUID` | Sets the user id who will run transmission | `PUID=1003`|
|`PGID` | Sets the group id for the transmission user | `PGID=1003` |

### Dropping default route from iptables (advanced)

Some VPNs do not override the default route, but rather set other routes with a lower metric.  
This might lead to te default route (your untunneled connection) to be used.

To drop the default route set the environment variable `DROP_DEFAULT_ROUTE` to `true`.

*Note*: This is not compatible with all VPNs. You can check your iptables routing with the `ip r` command in a running container.

### Custom pre/post scripts

If you ever need to run custom code before or after transmission is executed or stopped, you can use the custom scripts feature.
Custom scripts are located in the /scripts directory which is empty by default.
To enable this feature, you'll need to mount the /scripts directory.

Once /scripts is mounted you'll need to write your custom code in the following bash shell scripts:

| Script | Function |
|----------|----------|
|/scripts/transmission-pre-start.sh | This shell script will be executed before transmission start |
|/scripts/transmission-post-start.sh | This shell script will be executed after transmission start |
|/scripts/transmission-pre-stop.sh | This shell script will be executed before transmission stop |
|/scripts/transmission-post-stop.sh | This shell script will be executed after transmission stop |

Don't forget to include the #!/bin/bash shebang and to make the scripts executable using chmod a+x

#### Use docker env file
Another way is to use a docker env file where you can easily store all your env variables and maintain multiple configurations for different providers.
In the GitHub repository there is a provided DockerEnv file with all the current transmission and openvpn environment variables. You can use this to create local configurations
by filling in the details and removing the # of the ones you want to use.

Please note that if you pass in env. variables on the command line these will override the ones in the env file.

See explanation of variables above.
To use this env file, use the following to run the docker image:
```
$ docker run --cap-add=NET_ADMIN --device=/dev/net/tun -d \
              -v /your/storage/path/:/data \
              -v /etc/localtime:/etc/localtime:ro \
              --env-file /your/docker/env/file \
              -p 9091:9091 \
              uri:https://github.com/sscraggles/docker-deluge-openvpn/
```

## Known issues, tips and tricks

#### Use Google DNS servers
Some have encountered problems with DNS resolving inside the docker container.
This causes trouble because OpenVPN will not be able to resolve the host to connect to.
If you have this problem use dockers --dns flag to override the resolv.conf of the container.
For example use googles dns servers by adding --dns 8.8.8.8 --dns 8.8.4.4 as parameters to the usual run command.

#### Restart container if connection is lost
If the VPN connection fails or the container for any other reason loses connectivity, you want it to recover from it. One way of doing this is to set environment variable `OPENVPN_OPTS=--inactive 3600 --ping 10 --ping-exit 60` and use the --restart=always flag when starting the container. This way OpenVPN will exit if ping fails over a period of time which will stop the container and then the Docker deamon will restart it.

#### Running it on a NAS
Several popular NAS platforms supports Docker containers. You should be able to set up and configure this container using their web interfaces. Remember that you need a TUN/TAP device to run the container. To set up the device it's probably simplest to install a OpenVPN package for the NAS. This should set up the device. If not, there are some more detailed instructions below.

#### Questions?
If you are having issues with this container please submit an issue on GitHub.
Please provide logs, docker version and other information that can simplify reproducing the issue.
Using the latest stable verison of Docker is always recommended. Support for older version is on a best-effort basis.

## Adding new providers
If your VPN provider is not in the list of supported providers you could always create an issue on GitHub and see if someone could add it for you. But if you're feeling up for doing it yourself, here's a couple of pointers.

You clone this repository and create a new folder under "openvpn" where you put the .ovpn files your provider gives you. Depending on the structure of these files you need to make some adjustments. For example if they come with a ca.crt file that is referenced in the config you need to update this reference to the path it will have inside the container (which is /etc/openvpn/...). You also have to set where to look for your username/password.

There is a script called adjustConfigs.sh that could help you. After putting your .ovpn files in a folder, run that script with your folder name as parameter and it will try to do the changes descibed above. If you use it or not, reading it might give you some help in what you're looking to change in the .ovpn files.

Once you've finished modifying configs, you build the container and run it with OPENVPN_PROVIDER set to the name of the folder of configs you just created (it will be lowercased to match the folder names). And that should be it!

So, you've just added your own provider and you're feeling pretty good about it! Why don't you fork this repository, commit and push your changes and submit a pull request? Share your provider with the rest of us! :) Please submit your PR to the dev branch in that case.

### Using a custom provider

If you want to run the image with your own provider without building a new image, that is also possible. For some providers, like AirVPN, the .ovpn files are generated per user and contains credentials. They should not be added to a public image. This is what you do:

Add a new volume mount to your `docker run` command that mounts your config file:
`-v /path/to/your/config.ovpn:/etc/openvpn/custom/default.ovpn`

Then you can set `OPENVPN_PROVIDER=CUSTOM`and the container will use the config you provided. If you are using AirVPN or other provider with credentials in the config file, you still need to set `OPENVPN_USERNAME` and `OPENVPN_PASSWORD` as this is required by the startup script. They will not be read by the .ovpn file, so you can set them to whatever.

Note that you still need to modify your .ovpn file as described in the previous section. If you have an separate ca.crt file your volume mount should be a folder containing both the ca.crt and the .ovpn config.

## Controlling Transmission remotely
The container exposes /config as a volume. This is the directory where the supplied transmission and OpenVPN credentials will be stored.
If you have transmission authentication enabled and want scripts in another container to access and
control the transmission-daemon, this can be a handy way to access the credentials.
For example, another container may pause or restrict transmission speeds while the server is streaming video.

## Running on ARM (Raspberry PI)
Since the Raspberry PI runs on an ARM architecture instead of x64, the existing x64 images will not
work properly. To support users that wish to run this container on a Raspberry Pi, there are 2 additional
Dockerfiles created. The Dockerfiles supported by the Raspberry PI are Dockerfile.armhf -- there is
also an example docker-compose-armhf file that shows how you might use Transmission/OpenVPN and the
corresponding nginx reverse proxy on an RPI machine.

## Make it work on Synology NAS
Here are the steps to run it on a Synology NAS (Tested on DSM 6) :

- Connect as _admin_ to your Synology SSH
- Switch to root with command `sudo su -`
- Enter your _admin_ password when prompted
- Create a TUN.sh file anywhere in your synology file system by typing `vim /volume1/foldername/TUN.sh`
replacing _foldername_ with any folder you created on your Synology
- Paste @timkelty 's script :
```
#!/bin/sh

# Create the necessary file structure for /dev/net/tun
if ( [ ! -c /dev/net/tun ] ); then
	if ( [ ! -d /dev/net ] ); then
		mkdir -m 755 /dev/net
	fi
	mknod /dev/net/tun c 10 200
fi

# Load the tun module if not already loaded
if ( !(lsmod | grep -q "^tun\s") ); then
	insmod /lib/modules/tun.ko
fi
```
- Save the file with [escape] + `:wq!`
- Go in the folder containing your script : `cd /volume1/foldername/`
- Check permission with `chmod 0755 TUN.sh`
- Run it with `./TUN.sh`
- Return to initial directory typing `cd`
- Create the DNS config file by typing `vim /volume1/foldername/resolv.conf`
- Paste the following lines :
```
nameserver 8.8.8.8
nameserver 8.8.4.4
```
- Save the file with [escape] + `:wq!`
- Create your docker container with a the following command line: 
 
      # Tested on DSM 6.1.4-15217 Update 1, Docker Package 17.05.0-0349
      docker run \
          --cap-add=NET_ADMIN \
          --device=/dev/net/tun \
          -d \
          -v /volume1/foldername/resolv.conf:/etc/resolv.conf \
          -v /volume1/yourpath/:/data \
          -e "OPENVPN_PROVIDER=PIA" \
          -e "OPENVPN_CONFIG=Netherlands" \
          -e "OPENVPN_USERNAME=XXXXX" \
          -e "OPENVPN_PASSWORD=XXXXX" \
          -e "LOCAL_NETWORK=192.168.0.0/24" \
          -e "OPENVPN_OPTS=--inactive 3600 --ping 10 --ping-exit 60" \
          -e "PGID=100" \
          -e "PUID=1234" \
          -p 9091:9091 \
          --sysctl net.ipv6.conf.all.disable_ipv6=0 \
          --name "deluge-openvpn-syno" \
          uri:https://github.com/sscraggles/docker-deluge-openvpn/

- To make it work after a nas restart, create an automated task in your synology web interface : go to **Settings Panel > Task Scheduler ** create a new task that run `/volume1/foldername/TUN.sh` as root (select '_root_' in 'user' selectbox). This task will start module that permit the container to run, you can make a task that run on startup. These kind of task doesn't work on my nas so I just made a task that run every minute.
- Enjoy

## systemd Integration

On many modern linux systems, including Ubuntu, systemd can be used to start the transmission-openvpn at boot time, and restart it after any failure.

Save the following as `/etc/systemd/system/transmission-openvpn.service`, and replace the OpenVPN PROVIDER/USERNAME/PASSWORD directives with your settings, and add any other directives that you're using.

This service is assuming that there is a `bittorrent` user set up with a home directory at `/home/bittorrent/`. The data directory will be mounted at `/home/bittorrent/data/`. This can be changed to whichever user and location you're using.

OpenVPN is set to exit if there is a connection failure. OpenVPN exiting triggers the container to also exit, then the `Restart=always` definition in the `transmission-openvpn.service` file tells systems to restart things again.

```
[Unit]
Description=haugene/transmission-openvpn docker container
After=docker.service
Requires=docker.service

[Service]
User=bittorrent
TimeoutStartSec=0
ExecStartPre=-/usr/bin/docker kill transmission-openvpn
ExecStartPre=-/usr/bin/docker rm transmission-openvpn
ExecStartPre=/usr/bin/docker pull haugene/transmission-openvpn
ExecStart=/usr/bin/docker run \
        --name transmission-openvpn \
        --cap-add=NET_ADMIN \
        --device=/dev/net/tun \
        -v /home/bittorrent/data/:/data \
        -e "OPENVPN_PROVIDER=TORGUARD" \
        -e "OPENVPN_USERNAME=bittorrent@example.com" \
        -e "OPENVPN_PASSWORD=hunter2" \
        -e "OPENVPN_CONFIG=Netherlands" \
        -e "OPENVPN_OPTS=--inactive 3600 --ping 10 --ping-exit 60" \
        -e "TRANSMISSION_UMASK=0" \
        -p 9091:9091 \
        --dns 8.8.8.8 \
        --dns 8.8.4.4 \
        uri:https://github.com/sscraggles/docker-deluge-openvpn/
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Then enable and start the new service with:

```
$ sudo systemctl enable /etc/systemd/system/transmission-openvpn.service
$ sudo systemctl restart transmission-openvpn.service
```

If it is stopped or killed in any fashion, systemd will restart the container. If you do want to shut it down, then run the following command and it will stay down until you restart it.

```
$ sudo systemctl stop transmission-openvpn.service
# Later ...
$ sudo systemctl start transmission-openvpn.service
```
