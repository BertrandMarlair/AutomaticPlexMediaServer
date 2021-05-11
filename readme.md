# Automatic Plex Media Player

This setup will automatically download fimls or series and put it on Plex Media Server.

## Setup

### Setup deluge

#### Create labels

Create Radarr label -> Add move on downloaded to /downloads/radarr
Create Sonarr label -> Add move on downloaded to /downloads/sonarr

#### Setup jacket

- Add some indexers to jackett

#### Setup Radarr

- add some torznab indexer from jackett with http://jackett:9117/... (you can copy the link of the indexer on jackett)
- add the api key, you can found it on jackett too
- Set your language preferences
- Set your video quality preference
- add a downloader -> deluge (http://deluge:8112)

#### Setup Sonarr

- add some torznab indexer from jackett with http://jackett:9117/... (you can copy the link of the indexer on jackett)
- add the api key, you can found it on jackett too
- Set your language preferences
- Set your video quality preference
- add a downloader -> deluge (http://deluge:8112)

#### Setup traktarr
- start first the docker config and get the api key from Radarr and Sonarr
- Stop the server
- Edit the file config on .config/traktarr/config.json
- Use the api key of Radarr and Sonarr
- Go to trakt.tv and login
- go to settings and Your API Apps
- Create and app, give a name and use urn:ietf:wg:oauth:2.0:oob as Redirect URI.
- Add permission of /checkin and /scrobble
- Create and copy the Client ID and Client Secret -> use it on .config/traktarr/config.json
- authorize the app on trakt.tv
- start the docker compose
- open a nex terminal and run "sudo docker-compose exec trakt sh"
For each user you want to access the private lists for (i.e. watchlist and/or custom lists), you will need to to authenticate that user.

Repeat the following steps for every user you want to authenticate:

Run the following command:

traktarr trakt_authentication
You wil get the following prompt:

- We're talking to Trakt to get your verification code. Please wait a moment...
- Go to: https://trakt.tv/activate on any device and enter A0XXXXXX. We'll be polling Trakt every 5 seconds for a reply
Go to https://trakt.tv/activate.

Enter the code you see in your terminal.

Click Continue.

If you are not logged in to Trakt.tv, login now.

Click Accept.

You will get the message: "Woohoo! Your device is now connected and will automatically refresh in a few seconds.".

You've now authenticated the user.

You can repeat this process for as many users as you like.


# Traefik Local

Setup a local Traefik web proxy with DNS resolution on all \*.lan and \*.test domains.

Also setup a local trusted Root CA and create a TLS certificate for using https in local (shout out to [mkcert](https://github.com/FiloSottile/mkcert)).

This guide is for MacOS, but will work on Linux with minor modifications.

## 0. Prerequisites

- [Docker](https://docs.docker.com/docker-for-mac/install/)
- [Homebrew](https://brew.sh/)

## 1. Setup resolvers

```sh
# Setup MacOS to take into account our local docker resolver
sudo mkdir -p /etc/resolver
echo "nameserver 127.0.0.1" | sudo tee -a /etc/resolver/lan > /dev/null
echo "nameserver 127.0.0.1" | sudo tee -a /etc/resolver/test > /dev/null
```

## 2. Setup a local Root CA

```sh
brew install mkcert
brew install nss # only if you use Firefox

# Setup the local Root CA
mkcert -install

# Local Root CA files are located under ~/Library/Application\ Support/mkcert
# Look at https://github.com/FiloSottile/mkcert is you need instructions to install them on another device
```

## 3. Setup a Traefik container w/ https

```sh
# Clone this repository
git clone https://github.com/SushiFu/traefik-local.git
cd traefik-local/

# Create an external network docker, all future containers
# which need to be exposed by domain name should use this network
docker network create docker

# Create a local TLS certificate
# You could add any domain you need ending by .lan or .test
# *.this.test will create a wildcard certificate so any subdomain in the form like.this.test will also work.
# Unfortunately you cannot create *.test wildcard certificate your browser will not allow it.
mkcert -cert-file certs/local.crt -key-file certs/local.key "domo.lan" "*.domo.lan"

# Start Traefik
docker-compose pull
docker-compose up -d

# Go on https://traefik.this.test you should have the traefik web dashboard serve over https
```

## 4. Setup your dev containers

```sh
# On your docker-compose.yaml file

# Add the external network web at the end of the file
networks:
  default:
    external:
      name: docker

# Add these labels on the container
    labels:
      - traefik.enable=true
      - traefik.http.routers.my-frontend.entrypoints=http,https
      - traefik.http.routers.my-frontend.rule=Host(`my-frontend.domo.lan`) # You can use any domain allowed by your TLS certificate
      - traefik.http.routers.my-frontend.tls=true
      - traefik.http.routers.my-frontend.service=my-docker-compose-service-name
      - traefik.http.services.my-frontend.loadbalancer.server.port=3636 # Adapt to the exposed port in the service

# Protip: For web applications, use the same origin domain for your frontend and backend to avoid cookies sharing issues.
# By example: https://domo.lan (frontend) and https://deluge.domo.lan (backend)
```

### Create Traefik user 
*** echo $(htpasswd -nB user) | sed -e s/\\$/\\$\\$/g ***