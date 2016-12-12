# Official Docker container for Plex Media Server

# plexinc/pms-docker

With our easy-to-install Plex Media Server software and your Plex apps, available on all your favorite phones, tablets, streaming devices, gaming consoles, and smart TVs, you can stream your video, music, and photo collections any time, anywhere, to any device.

## Usage
```
docker create \
--name plex \
-p 32400:32400/tcp \
-p 3005:3005/tcp \
-p 8324:8324/tcp \
-p 32469:32469/tcp \
-p 1900:1900/udp \
-p 32410:32410/udp \
-p 32412:32412/udp \
-p 32413:32413/udp \
-p 32414:32414/udp \
-e TZ="<timezone>" \
-e PLEX_CLAIM="<claimToken>" \
-e ADVERTISE_IP="http://<hostIPAddress>:32400/" \
-h <HOSTNAME>
-v <path/to/plex/database>:/config \
-v <path/to/transcode/temp>:/transcode \
-v <path/to/tv/series>:/data/tvshows \
-v <path/to/movies>:/data/movies \
-v <path/to/another/media>:/data/mediaTypeA \
-v <path/to/some/other/media>:/data/mediaTypeB \
plexinc/pms-docker
```

## Parameters

- `-p 32400:32400/tcp` Forwards port 32400 from the host to the container.  This is the primary port that Plex uses for communication and is required for Plex Media Server to operate.
- `-p *` Forwards complete set of other ports used by Plex to the container.  For a full explanation of which you may need, please see the help article: https://support.plex.tv/hc/en-us/articles/201543147-What-network-ports-do-I-need-to-allow-through-my-firewall
- `-v <path/to/plex/database>:/config` The path where you wish Plex Media Server to store its configuration data.  This database can grow to be quite large depending on the size of your media collection.  This is usually a few GB but for large libraries or libraries where index files are generated, this can easily hit the 100s of GBs.
- `-v <path/to/transcode/temp>:/transcode` The path where you would like Plex Media Server to store its transcoder temp files.  If not provided, the storage space within the container will be used.  Expect sizes in the 10s of GB.
- `-v <path/to/media>:/data/*` These are provided as examples for providing media into the container.  The exact structure of how the media is organized and presented inside the container is a matter of user preference.  You can use as many or as few of these parameters as required to provide your media to the container
- `-e KEY="value"` These are environment variables which configure the container.  See below for a description of their meanings.

The following are the recommended parameters.  With the exception of `VERSION`, each of the following parameters to the container are treated as first-run parameters only.  That is, all other paraters are ignored on subsequent runs of the server.  We recommend that you set the following parameters:

- **HOSTNAME** Sets the hostname inside the docker container. For example `-h PlexServer` will set the servername to PlexServer.
- **TZ** Set the timezone inside the container.  For example: `Europe/London`.  The complete list can be found here: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
- **PLEX_CLAIM** The claim token for the server to obtain a real server token.  If not provided, server is will not be automatically logged in.  If server is already logged in, this parameter is ignored.
- **ADVERTISE_IP** This variable defines the additional IPs on which the server may be be found.  For example: `http://10.1.1.23:32400`.  This is recommended because the IP address seen within the container is usually not the IP address of the host.  This adds to the list where the server advertises that it can be found.
- **VERSION** The version of PMS to install (defaults to `latest`).  Can be `latest`, `public`, or an explicit version number.  If the server is not logged in or you do not have Plex Pass on your account, this will be restricted to publicly available versions only.

These parameters are usually not required but some special setups may benefit from their use.  As in the previous section, each is treated as first-run parameters only:

- **PLEX_UID** The user id of the `plex` user created inside the container.
- **PLEX_GID** The group id of the `plex` group created inside the container
- **CHANGE_CONFIG_DIR_OWNERSHIP** Change ownership of config directory to the plex user.  Defaults to `true`
- **CHANGE_DIR_RIGHTS** Change permissions of data directory (media) to be readable by plex user.  Defaults to `false`
- **ALLOWED_NETWORKS** IP/netmask entries which allow access to the server without requiring authorization.  We recommend you set this only if you do not sign in your server.  For example `192.168.1.0/24,172.16.0.0/16` will allow access to the entire `192.168.1.x` range and the `172.16.x.x` range.

### Networking
If `--net=host` is used, then the port forwarding paramaters (`-p …`) as well as the `ADVERTISE_IP` variable may be excluded.  This type of networking runs the container using the same networking stack as the host and as such it is no longer hidden behind the docker networking setup.  However, many docker implementations do not run docker on the actual host itself but rather run docker within a virtual machine.  In this case, the `--net-host` parameter may not provide the desired results.

(Plex Pass only) If the normal docker networking is used, after the server has been set up, you should configure the `LAN Networks` preference to contain the network of your LAN.  This instructs the Plex Media Server to treat these IP addresses as part of your LAN when applying bandwidth controls.  The syntax is the same as the `ALLOWED_NETWORKS` above.  For example `192.168.1.0/24,172.16.0.0/16` will allow access to the entire `192.168.1.x` range and the `172.16.x.x` range.

## Users/Groups
Permissions of mounted media outside the container do apply to the Plex Media Server running within the container.  As stated above, the Plex Media Server runs as a specially created `plex` user within the container.  This user may not exist outside the container and so the `PLEX_UID` and `PLEX_GID` parameters are used to set the user id and group id of this user within the container.  If you wish for the Plex Media Server to run under the same permissions as your own user, execute the following to find out these ids:

```
$ id `whoami`
```

You'll see a line like the following:

```
uid=1001(myuser) gid=1001(myuser) groups=1001(myuser)
```

In the above case, if you set the `PLEX_UID` and `PLEX_GID` to `1001`, then the permissions will match that of your own user.

## Installation
The Plex Media Server is installed when the container is first run.  Additionally it will attempt to upgrade every time the container is started.  If you wish to upgrade the version within the container, simply stop and start the container when you have a network connection.  The startup script will automatically fetch the version associated with the `VERSION` variable and install it before starting the Plex Media Server.

The values of `VERSION` are as follows:

- **latest**: Fetches the latest version your account can access.  If the server is logged in and your account has Plex Pass, this will be the latest Plex Pass version.  Otherwise this will be the latest public version.
- **public**: Fetches the latest public version.  Even if your server is logged in and has Plex Pass, this will still only fetch the latest public version.
- **`<specific version>`**: Fetches the exact version specified.  This is subject to availability of that particular version.  This cannot be used to fetch Plex Pass versions of the server for users that do not have Plex Pass on their accounts.  For version `1.2.7`, this value would be `1.2.7.2987-1bef33a`.

## Useful information
- Shell access to the container while it is running: `docker exec -it plex /bin/bash`
- See the logs given by the startup script in real time: `docker logs -f plex`
- Restart the application and upgrade to the latest version: `docker restart plex`