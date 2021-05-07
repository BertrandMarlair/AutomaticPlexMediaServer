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

