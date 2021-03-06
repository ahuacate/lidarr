# Lidarr Build
This recipe is for setting up Lidarr. The following assumes your are familiar with Lidarr and its preference menus and configuration options.

Network Prerequisites are:
- [x] Layer 2 Network Switches
- [x] Network Gateway is `192.168.1.5`
- [x] Network DNS server is `192.168.1.5` (Note: your Gateway hardware should enable you to a configure DNS server(s), like a UniFi USG Gateway, so set the following: primary DNS `192.168.1.254` which will be your PiHole server IP address; and, secondary DNS `1.1.1.1` which is a backup Cloudfare DNS server in the event your PiHole server 192.168.1.254 fails or os down)
- [x] Network DHCP server is `192.168.1.5`

Other Prerequisites are:
- [x] Synology NAS, or linux variant of a NAS, is fully configured as per [SYNOBUILD](https://github.com/ahuacate/synobuild#synobuild).
- [x] Proxmox node fully configured as per [PROXMOX-NODE BUILDING](https://github.com/ahuacate/proxmox-node/blob/master/README.md#proxmox-node-building).
- [x] pfSense is fully configured as per [HAProxy in pfSense](https://github.com/ahuacate/proxmox-reverseproxy/blob/master/README.md#haproxy-in-pfsense)
- [x] Deluge LXC with Deluge SW installed as per [Deluge LXC - Ubuntu 18.04](https://github.com/ahuacate/proxmox-lxc-media/blob/master/README.md#40-deluge-lxc---ubuntu-1804).
- [x] NZBGet LXC with NZBGet SW installed as per [NZBget LXC - Ubuntu 18.04](https://github.com/ahuacate/proxmox-lxc-media/blob/master/README.md#300-nzbget-lxc---ubuntu-1804)
- [x] Jackett installed as per [Jackett LXC - Ubuntu 18.04](https://github.com/ahuacate/proxmox-lxc-media/blob/master/README.md#500-jackett-lxc---ubuntu-1804)
- [x] Lidarr LXC with Lidarr SW installed as per [Lidarr LXC - Ubuntu 18.04](https://github.com/ahuacate/proxmox-lxc-media/blob/master/README.md#1000-lidarr-lxc---ubuntu-1804)

Tasks to be performed are:
- [1.00 Manually Configure Lidarr Settings](#100-manually-configure-lidarr-settings)
	- [2.01 Configure Media Management](#201-configure-media-management)
	- [2.02 Configure Profiles - Quality Profiles](#202-configure-profiles---quality-profiles)
	- [2.03 Configure Profiles - Metadata Profiles](#203-configure-profiles---metadata-profiles)
	- [2.04 Configure Profiles - Delay Profiles](#204-configure-profiles---delay-profiles)
	- [2.05 Configure Indexers](#205-configure-indexers)
	- [2.05a  Add Jackett as a Indexer](#205a--add-jackett-as-a-indexer)
	- [2.05b Add Headphones](#205b-add-headphones)
	- [2.05c  Add Usenet Indexers](#205c--add-usenet-indexers)
	- [2.06 Configure Download Clients](#206-configure-download-clients)
	- [2.06a  Deluge Download Client](#206a--deluge-download-client)
	- [2.06b  NZBGet Download Client](#206b--nzbget-download-client)
	- [2.07 Configure Import Lists](#207-configure-import-lists)
	- [2.07 Configure Connect](#207-configure-connect)
	- [2.08 Configure General](#208-configure-general)
	- [2.09 Configure Metadata](#209-configure-metadata)
	- [2.10 Configure General](#210-configure-general)
	- [2.11 Configure UI](#211-configure-ui)
- [3.00 Create & Restore Lidarr Backups](#300-create--restore-lidarr-backups)
	- [3.01 Create a Base Settings Backup](#301-create-a-base-settings-backup)
	- [3.02 Restore to Lidarr Base Settings](#302-restore-to-lidarr-base-settings)
	- [3.03 Restore the lastest Lidarr backup](#303-restore-the-lastest-lidarr-backup)
- [00.00 Patches & Fixes](#0000-patches--fixes)


## 1.00 Manually Configure Lidarr Settings
Browse to http://192.168.50.117:8686/ and login to Lidarr. Click the `Settings Tab` and set the state to show Advanced Settings. Configure all your tabs as follows.

### 2.01 Configure Media Management
Good practice is to include the media quality in filenames for all content (i.e music, TV series, movies). Here are my `Track Naming` formats. 

| Media Management  | Value
| :---  | :---
| **Track Naming**
| Rename Tracks | `☑`
| Replace Illegal Characters| `☑`
| Standard Track Format | `{Artist Name} - {Album Title} - {track:00} - {Track Title} - [{Quality Title} {MediaInfo AudioCodec} {MediaInfo AudioBitRate}]`
| Multi Disc Track Format | `{Medium Format} {medium:00}/{Artist Name} - {Album Title} - {track:00} - {Track Title} - [{Quality Title} {MediaInfo AudioCodec} {MediaInfo AudioBitRate}]`
| Artist Folder Format | `{Artist Name}`
| Album Folder Format | `{Artist Name} - {Album Title} ({Release Year})`

![alt text](https://raw.githubusercontent.com/ahuacate/lidarr/master/images/management.png)

### 2.02 Configure Profiles - Quality Profiles
I prefer `lossless` audio but if not available `high quality lossy` is okay too. In the event `lossless` becomes available at a later stage I want Lidarr to perform a upgrade.

Create a new profile (i.e `Ahuacate`) as conplete the fields as follows:
![alt text](https://raw.githubusercontent.com/ahuacate/lidarr/master/images/quality_profiles.png)

### 2.03 Configure Profiles - Metadata Profiles
To minimise my selection to a rationale volume of content I use `Studio` and `Compilation` in my settings.
![alt text](https://raw.githubusercontent.com/ahuacate/lidarr/master/images/metadata_profiles.png)

### 2.04 Configure Profiles - Delay Profiles
Edit Delay Profiles. Add 1440 minutes to both usenet and torrent delay fields.
![alt text](https://raw.githubusercontent.com/ahuacate/lidarr/master/images/delay_profiles.png)

### 2.05 Configure Indexers
This is where you configure Lidarr to use Usenet as your primary search indexer and Torrents (Jackett) as the secondary indexer. For torrents Lidarr uses Jackett which must be installed as shown [HERE](https://github.com/ahuacate/jackett).

### 2.05a  Add Jackett as a Indexer
Create a new torrent indexer using the `Torznab Jackett Preset` template and fill out the details as shown below.

| Add Torznab | Value
| :---  | :---:
| Name | `Jackett`
| Enable RSS | `☐`
| Enable Automatic Search | `☑`
| Enable Interactive Search | `☑`
| URL | `http://192.168.50.120:9117`
| API Path | `/torznab/all/api`
| API Key | `s9tcqkddvjpkmis824pp6ucgpwcd2xnc`
| Categories | `3000,3010,3020,3030,3040`
| Early Download Limit| leave blank
| Additional Parameters | leave blank

And click `Save`. The finished Jackett configuration looks like:

![alt text](https://raw.githubusercontent.com/ahuacate/lidarr/master/images/torznab.png)

### 2.05b Add Headphones
If you havent done so get a VIP account with [Headphones](https://headphones.codeshy.com/vip/).

| Add Headphones VIP | Value
| :---  | :---:
| Name | `Headphones`
| Enable RSS | `☑`
| Enable Automatic Search | `☑`
| Enable Interactive Search | `☑`
| Categories | `3000,3010,3020,3030,3040`
| Username | insert here
| Password | insert here
| Early Download Limit| leave blank

And click 'Save'.

![alt text](https://raw.githubusercontent.com/ahuacate/lidarr/master/images/headphones.png)

### 2.05c  Add Usenet Indexers
Add all your Usenet indexers providers with the `Newsnab` presets (or custom if your provider is not listed).

Finally edit the `Options` Retention to `2000` days.

![alt text](https://raw.githubusercontent.com/ahuacate/lidarr/master/images/indexer.png)

### 2.06 Configure Download Clients
Add the following download clients.

### 2.06a  Deluge Download Client
First create a new download client using the `Torrent > Deluge` template and fill out the details as shown below.

| Add Deluge | Value | Notes
| :---  | :---: | :---
| Name | `Deluge`
| Enable| `Yes`
| Host | `192.168.30.113`
| Port | `8112`
| URL Base| leave blank
| Password| `insert your deluge password` | This is your Deluge login password.
| Category | `lidarr-music`
| Recent Priority | Last
| Older Priority | Last
| Add Paused | `☐`
| Use SSL | `☐`

And click `Test` to check it works. If successful, click `Save`.

![alt text](https://raw.githubusercontent.com/ahuacate/lidarr/master/images/deluge.png)

### 2.06b  NZBGet Download Client
First create a new download client using the `Usenet > NZBGet` template and fill out the details as shown below.

| Add NZBGet | Value | Notes
| :---  | :---: | :---
| Name | `NZBGet`
| Enable| `Yes`
| Host | `192.168.30.112`
| Port | `6789`
| URL Base| leave blank
| Username | `client`
| Password| `insert your client password` | This is your NZBGet client password.
| Category | `lidarr-music`
| Recent Priority | Normal
| Older Priority | Normal
| Add Paused | `☐`
| Use SSL | `☐`

And click `Test` to check it works. If successful, click `Save`.

![alt text](https://raw.githubusercontent.com/ahuacate/lidarr/master/images/nzbget.png)

Other `download tab` settings must be set as follows:

![alt text](https://raw.githubusercontent.com/ahuacate/lidarr/master/images/download.png)

### 2.07 Configure Import Lists
If you want you can add Spotify or Last.fm to import your playlists. This example uses a personal Spotify playlist(s).

| Spotify Playlists | Value | Notes
| :---  | :---: | :---
| Name | `SpotifyPlaylist` | *Best to use your Spotify account username so you can add multiple Spotify playlists to Lidarr (i.e Spotify Adolf, Spotify Eva or Spotify Kids)*
| Enable Automatic Add | `☑`
| Monitor | `Specific Album`
| Root Folder | `/mnt/music`
| Quality Profile | `Lossless`
| Lidarr Tags | leave blank
| Playlists | Select playlists to import from Spotify
| Authenticate with Spotify | Click `Authenticate with Spotify`

![alt text](https://raw.githubusercontent.com/ahuacate/lidarr/master/images/spotify.png)

### 2.07 Configure Connect
Create a new connection using the `Emby (Media Browser)` template and fill out the details as shown below.

| Add - Emby (Media Browser) | Value | Notes
| :---  | :---: | :---
| Name | `Jellyfin`
| On Grab | `☐`
| On Release Import | `☑`
| On Upgrade | `☑`
| On Download Failure | `☐`
| On Import Failure | `☐`
| On Rename | `☑`
| On Track Retag | `☑`
| On Health Issue | `☐`
| Tags | leave blank
| Host | `192.168.50.111` 
| Port | `8096`
| Use SSL | `☐`
| API Key | Insert your Jellyfin API key |	*Note, create one in Jellyfin for Lidarr*
| Send Notifications | `☐`
| Update Library | `☑`

And click `Test` to check it works. If successful, click `Save`.

![alt text](https://raw.githubusercontent.com/ahuacate/lidarr/master/images/jellyfin.png)

### 2.08 Configure General
Here are required edits: 1) URL Base; and, 2) setting the security section to enable username and login.

| Start-Up | Value | Notes
| :---  | :---: | :---
| Bind Address | `*`
| Port Number | 8686
| URL Base | `/lidarr`
| Enable SSL | No
| Open Browser on start | Yes
| **Security**
| Authentication | `Basic (Browser Pop-up)`
| Username | `storm` | *Note, or whatever username you choose*
| Password | `insert password here` | *Add a complex password and record it i.e oTL&9qe/9Y&RV*
| API Key | leave default

And click `Save`.

![alt text](https://raw.githubusercontent.com/ahuacate/lidarr/master/images/general.png)

### 2.09 Configure Metadata

![alt text](https://raw.githubusercontent.com/ahuacate/lidarr/master/images/metadata.png)

### 2.10 Configure General

![alt text](https://raw.githubusercontent.com/ahuacate/lidarr/master/images/general.png)

### 2.11 Configure UI

![alt text](https://raw.githubusercontent.com/ahuacate/lidarr/master/images/ui.png)


## 3.00 Create & Restore Lidarr Backups
Lidarr has a built in backup service. Lidarr will execute a backup every 7 days creating a zip file located in `/home/media/.config/Lidarr/Backups/scheduled`.

But it's good idea to make a clean backup of your working Lidarr settings, including all settings such as passwords etc, before adding any movie media. The clean backup file MUST be stored outside of the Proxmox Lidarr LXC container for safe keeping. Then in the event of you needing to recreate your Lidarr LXC you can use this backup file to quickly restore all your Lidarr settings.

This backup file must be named `lidarr_backup_base_settings.zip` and be located on your NAS in folder `/mnt/backup/lidarr` for the following scripts to work.

### 3.01 Create a Base Settings Backup
Perform after you have completed Steps 1.00 or Steps 2.00. This file must be stored on your NAS for future rebuilds

Browse to http://192.168.50.116:7878 and login to Lidarr. Click the `Systems Tab` > `Logs Tab` > `Table/Files/Updates Tabs` and click `Clear Logs` on all.

Then click `System Tab` > `Backup Tab` and click `Backup` to create a new backup file which will be shown with a name like `lidarr_backup_2019.09.24_05.39.55.zip`. Now right click on this newly created file (at the top of list) and save to your NAS share `/proxmox/backup/lidarr` (locally mounted as /mnt/backup/lidarr). Rename your backup file from `lidarr_backup_2019.09.24_05.39.55.zip` to `lidarr_backup_base_settings.zip`.

### 3.02 Restore to Lidarr Base Settings
With the Proxmox web interface go to `typhoon-01` > `116 (lidarr)` > `>_ Shell` and type the following:
```
sudo systemctl stop lidarr.service &&
sleep 5 &&
rm -r /home/media/.config/Lidarr/lidarr.db* &&
rm -r /home/media/.config/Lidarr/config.xml &&
su -c 'unzip -o /mnt/backup/lidarr/lidarr_backup_base_settings.zip 'lidarr.db*' 'config.xml' -d /home/media/.config/Lidarr' media &&
chown 1605:65605 /home/media/.config/Lidarr/lidarr.db* &&
chown 1605:65605 /home/media/.config/Lidarr/config.xml &&
sudo systemctl restart lidarr.service
```

### 3.03 Restore the lastest Lidarr backup
If you want to restore to your last backup (this backup is a maximum of 7 days of age) use the Proxmox web interface and go to `typhoon-01` > `116 (lidarr)` > `>_ Shell` and type the following: 
```
sudo systemctl stop lidarr.service &&
sleep 5 &&
rm -r /home/media/.config/Lidarr/lidarr.db* &&
rm -r /home/media/.config/Lidarr/config.xml &&
newest=$(ls -t /home/media/.config/Lidarr/Backups/scheduled/*.zip | head -1) &&
echo $newest &&
unzip -o "$newest" 'lidarr.db*' -d /home/media/.config/Lidarr &&
chown 1605:65605 /home/media/.config/Lidarr/lidarr.db* &&
sudo systemctl restart lidarr.service
```

---

## 00.00 Patches & Fixes
Nothing yet.
