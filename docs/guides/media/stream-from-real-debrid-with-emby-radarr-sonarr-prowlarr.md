---
title: Stream from Real-Debrid with Emby and Radarr, Sonarr, Prowlarr
description: This guide describes how to setup a torrent-like "arr stack", to achieve "stremio-like", streaming your media from Real-Debrid using Emby, without having to store it locally or even touch a torrent client or a VPN!
---
# "Infinite streaming" from Real Debrid with Emby, Prowlarr, Radarr, and Sonarr

This page describes a more complex, flexible version of our [plex_debrid-based, minimal solution](/guides/media/stream-from-real-debrid-with-plex/) to create your "infinite" Emby library, utilizing [Real-Debrid][real-debrid] for all media, streaming through your ElfHosted Emby instance.

The solution leverages the popular Arr tools to search, source, and sort your media, stealthily using symlinks to skip storage stresses!

!!! tip "New accounts get $10 credit, enough to run this stack for 7 days!"
    Our services are subscribed to in "[ElfBuckz][elfbuckz]", our in-store account credit. All new users get $10 ElfBuckz upon [account creation](https://store.elfhosted.com/my-account/), in order to "kick the tyres" :red_car: before commitment.

## Requirements

* [x] [Real-Debrid account][real-debrid] and [API token](https://real-debrid.com/apitoken)
* [x] [Prowlarr][prowlarr]
* [x] [Radarr][radarr]
* [x] [Sonarr][sonarr]
* [x] [Zurg][zurg]
* [x] [Zurg Rclone Mount](https://store.elfhosted.com/product/rclone-real-debrid/)
* [x] [Emby][emby]
* [x] [RDTClient][rdtclient]
* [x] [Autoscan][autoscan]


Optional:

* [ ] [Jellyseerr][jellyseerr]

[Get up to 50% off the required apps, conveniently bundled! :gift:](https://store.elfhosted.com/product/advanced-infinite-arr-emby-debrid-streaming-bundle){ .md-button .md-button--primary }

## How does it work?

Here's a diagram (*it's not as complicated as it looks!*), followed by some explanations:

```mermaid
flowchart TD
    Z[fa:fa-user User] --> |Stream|K
    Z[fa:fa-user User] --> A
    Z[User]--> |Manual addition|D
    Z[User]--> |Requests|C
    A[Lists] --> C[Jellyseer]
    C <--> |Add to Arrs|D[Radarr/Sonarr]
    D --> R[Autoscan]
    D <--> |Search for media|E[Prowlarr]
    D <--> |Add to downloader|G[RDTClient]
    E --> |Search trackers|F[torrent.io]
    G <--> |1. Add torrent|H[real-debrid]

    R[Autoscan] --> K[Emby] 

    K[Emby] --> |"Read file (symlink)"|L["/storage/symlinks"]

    K[Emby] --> |"Resolve symlink (actual file)"|J["/storage/realdebrid-zurg"]
    %% G --> |Confirm download|J

    G[RDTClient] --> |3. Create symlink|P["/storage/symlinks"]

    H --> Q[Zurg+rclone]
    Q --> J

    Q --> O
    G[RDTClient] --> |2. Confirm download|O["/storage/realdebrid-zurg"]
```

1. The user adds content to their [Radarr][radarr] / [Sonarr][sonarr], or [Jellyseerr][jellyseerr]
2. Radarr / Sonarr notice the the new addition, and search Prowlarr (*torrentio indexer*) for appropriate files (*matching size, quality, language parameters*)
3. When an appropriate release is found, Radarr / Sonarr schedules a download via [RDTClient][rdtclient], which is presenting a qBittorrent-like API
4. RDTClient adds the torrent to Real-Debrid, and then monitors the zurg rclone mount to confirm the download has succeeded. Upon success, RDTClient **symlinks** the download to Radarr/Sonarr's `completed` directory, and they process is as if it were a local download (*renaming and moving to the media path*)
5. Aars trigger autoscan, which in turn triggers Emby to re-scan the media path containing the new content, and the Plex library is updated!

!!! question "Why not just use [plex_debrid][plex-debrid]?"

    1. For one thing, it only works with Plex :facepalm:
    2. You can use the Aars ability to select custom qualities per movie / show
    3. Integration into Jellyseerr / Arrs is well-understood and mature, and lets friends-and-family request content (*without complicated Plex Watchlist integrations*)

## How to set it up

Here's a little more detail..

1. Get a [Real-Debrid account][real-debrid] subscription for your ElfHosted account (*can't be used elsewhere at the same time, else you risk being banned*), and copy your [API token](https://real-debrid.com/apitoken)
2. Paste the token into the [Zurg product][store/zurg] on the store, and add to your cart
3. Add at the [Zurg rclone mount][store/zurgmount] and [Emby][store/emby] to your cart, and buy with [ElfBuckz][elfbuckz]!

On purchase, you'll get the necessary components deployed, but some configuration will be required, see below:

### Setup zurg

Edit your zurg config via [FileBrowser][filebrowser], and confirm that the following is set:

```
retain_rd_torrent_name: true
```

If you needed to make a change, restart zurg from FileBrowser console, using `elfbot restart zurg`

Confirm Zurg is setup correctly by browsing it from the link on your your ElfHosted dashboard.

### Setup Emby

Log into [Emby][emby], enable hardware transcoding, and setup the media libraries in `/storage/symlinks/{movies,series,movies-4k,series-4k, etc}`. Setup an API key for autoscan, and keep it for later.

### Setup RDTClient

Configure [RDTClient], by pasting your debrid provider's details into the config section (*the rest is pre-configured*).

### Setup Prowlarr

Navigate to Prowlarr, and configure ElfHosted's torrentio internal indexer (*search "torrentio"*). In the properties of the indexer, set your `Real-Debrid API Key`, test, and save.

Run `Sync App Indexers` to add the indexers (*via Prowlarr*) to Radarr and Sonarr.

### Setup Radarr / Sonarr

Configure your **Root Folders** to `/storage/symlinks/<whatever>`, since this is where the symlinks will appear. You do **not** need `/storage/realdebrid-zurg` (*in fact, its read-only nature will just confuse the Arr!*)

### Setup Autoscan

Radarr / Sonarr are pre-configured for [Autoscan][autoscan], but you'll need to edit Autoscan's config (`config/autoscan/` in [FileBrowser][filebrowser]), and add a token for Emby, in order for the autoscanning to work.

## Success!

That's it! You've got the basic plumbing in place for "unlimited storage" from Real-Debrid, managed by the Arrs and streamed by Emby!

!!! warning "Beware Real-Debrid IP bans"
    Be aware that Real-Debrid states:
    
    > You can use your account from any public IP address but you can't use your account from more than one public IP address at the same time.

    So if you use RealDebrid **outside** of ElfHosted, it's likely that your account may be warned / banned.
    
## How do I get help?

1. For general use of the [individual tools](/apps/), refer to the each app's upstream site
2. For specific support re your ElfHosted configuration / account, see the [ElfHosted support options](/get-help)

--8<-- "common-links.md"