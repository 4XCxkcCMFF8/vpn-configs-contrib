PrivateInternetAccess OpenVPN - Transmission
===
This Docker container lets you run Transmission with WebUI while connecting to PIA VPN for security. You can be up and running in no time, please read the instructions below.

# Building and running the container
To build this container, clone the repository and cd into it. There are a few options on how to configure the container, let's look at the simplest one first. In order to connect to PIA VPN you need to put your username and password in the piaconfig/credentials.txt file.

Once this is done, you're ready to build and start your container.

### Build it:
```
docker build -t="docker-transmission-openvpn" .
```
### Run it:
```
sudo docker run --privileged  -d -v /your/storage/path/:/data -p 9091:9091 docker-transmission-openvpn
```

This will build the image with default settings. What this means is that the VPN connects to PIA Netherlands with your credentials, starts Transmission WebUI with authentication disabled and Transmission will store your torrents to /your/storage/path/completed. Transmission assumes "completed, incomplete and watch" exists in /your/storage/path

### But I want to provide my own Transmission settings!
OK, so you're advanced. If you want to change the Transmission settings from the defaults, create your own settings.json file or base it on the default config. Then make the container use it like this:
```
sudo docker run --privileged  -d -v /your/storage/path/:/data -v /your/path/to/settings.json:/etc/transmission-daemon/settings.json -p 9091:9091 docker-transmission-openvpn
```

The container will now use your local settings.json file for its configuration.
NB: do not change this file while container is running. Transmission persist its config on shutdown, and this will override your changes. Stop the contaienr, do configurations, then start it again.

### Access the WebUI
But what's going on? My http://my-host:9091 isn't responding?
This is because the VPN is active, and since docker is running in a different ip range than your client the response to your request will be treated as "non-local" traffic and therefore be routed out through the VPN interface.

### How to fix this
There are several ways to fix this. You can pipe and do fancy iptables or ip route configurations on the host and in the Docker image. But I found that the simplest solution is just to proxy my traffic. Start a Nginx container like this:
```
docker run -d -v /path/to/nginx.conf:/etc/nginx.conf:ro -p 8080:80 nginx
```
Where /path/to/nginx.conf has this content:
```
events {
  worker_connections 1024;
}

http {
  server {
    listen 80;
    location / {
      proxy_pass http://host.ip.address.here:9091;
    }
  }
}
```
Your Transmission WebUI should now be avaliable at "http://your.host.ip.addr:8080/
Change the port in the docker run command if 8080 is not suitable for you.

Good luck!