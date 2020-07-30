# mediabox
mediabox is a media server in a box. It includes 7 distinct apps: sonarr, radarr, transmission, sabnzbd, jackett, nzbhydra, and of course, plex.

First of all, there are two indexers. Indexers are the tools that hold the metadata about the content you want to download. These are sites like ThePirateBay and YTS (for Torrents) or NZBGeek and DrunkenSlug (for Usenet/NZBs).
 
Then, there are two searchers. Searchers are the tools that you use to find the shows or movies you want to watch. You will use them the most. The searchers are sonarr (for TV) and radarr (for Movies).

Lastly, there are two downloaders. Downloaders do what they sound like they do. They receive signals from the searchers to download specific content. When done, they signal back to the searchers to pass off to Plex. The downloaders are Transmission (for Torrents) and SABnzbd (for Usenet/NZBs).

Finally, there is Plex which is your media server. You can use this directly at port `32400`, or you can access it via [plex.tv](https://plex.tv) after linking your account. You can also add it to a smart TV, your phone, etc. After downloading, the downloader notifies the requisite searcher of the finished state. The searcher then renames the files accordingly for Plex and puts them in the library. After a few minutes, Plex will scan and notice new files at which point you can watch your media.

# setup
First, make sure you have **docker** *and* **docker-compose**. Then:
```
git clone https://github.com/simpleauthority/mediabox.git
cd mediabox
docker-compose up -d
```

# apps
Once you have the stack running, you will find the apps at the following places by default:
| app          	| port 	| local URL             	|
|--------------	|------	|-----------------------	|
| sonarr       	| 8989 	| http://localhost:8989 	|
| radarr       	| 7878 	| http://localhost:7878 	|
| transmission 	| 9091 	| http://localhost:9091 	|
| sabnzbd      	| 8080 	| http://localhost:8080 	|
| jackett      	| 9117 	| http://localhost:9117 	|
| nzbhydra     	| 5076 	| http://localhost:5076 	|

# further configuration
1. In Sonarr, you will want to go to **Settings** -> **General** -> **Security** and choose:
    - **Authentication**: Forms (Login page)
    - **Username**: enter a username
    - **Password**: enter a password

2. In Radarr, do the same thing as Sonarr (step 1).

3. For Jackett, change the APIKey field in `config/jackett/Jackett/ServerConfig.json`. This means you will need to take that same APIKey value and edit each indexer in both Sonarr and Radarr at **Settings** -> **Indexers**. Click each one there and update the API key.

4. Update the Transmission password by editing `config/transmission/env` and changing the `PASS=` value. If you change the `USER=` value, be sure to also update it in the next step. After that, go to Sonarr and Radarr and update the **password** at **Settings** -> **Download Client** -> **Transmission**. Test, and then save.

5. If you don't plan to use Usenet at all, follow these steps:
    - `docker-compose down`
    - `rm -r config/{nzbhydra2,sabnzbd}`
    - Finally, remove these sections from `docker-compose.yml`:
    ```yml
    sabnzbd:
      image: linuxserver/sabnzbd
      container_name: sabnzbd
      restart: unless-stopped
      env_file:
        - ./config/sabnzbd/env
      ports:
        - 8080:8080
      volumes:
        - ./config/sabnzbd/:/config/
        - ./storage/:/storage/
    nzbhydra:
      image: linuxserver/nzbhydra2
      container_name: nzbhydra
      restart: unless-stopped
      ports:
        - 5076:5076
      volumes:
        - ./config/nzbhydra2/:/config/
    ```

6. If you **do plan to use Usenet**, you should change nzbhydra's API key at **Config** -> **Main** -> **API key** and turn on user authentication at **Config** -> **Authorization**:

    - **Auth type**: Login form
    - **Add new user**: fill out as necessary

    Usenet requires some extras. Most importantly, you will need to subscribe to Usenet. After that, you will also need to subscribe to a Usenet Indexer. The most common setup is to subscribe to two different Usenet providers (more on why later) and a few good Usenet Indexers (2 or 3).

    Which Usenet provider you subscribe to doesn't matter, so long as when you buy a *second* subscription that the new one is on a different backbone. Otherwise all you've done is purchase redundancy. See the [Usenet Network Map](https://www.reddit.com/r/usenet/wiki/providers#wiki_usenet_services_map) and choose your subscriptions from a new backbone each time. **If you don't buy in unique backbones, your download results will be the same. If the backbone doesn't have the file, it doesn't have the file!**)

    The Usenet provider gives you download and posting access to the Usenet Network. You can manually read files, but you can't **index** it. This is why you need a couple Usenet Indexers. These are your torrent equivalents of ThePirateBay, etc. They usually operate on a monthly/biannual/annual payments (annual best) and let you index as much as you want. I recommend NZBGeek.info and NZB.su.

    Once you have your providers and your indexers ready, proceed to [NZBHydra](http://localhost:5076). Go to **Config** -> **Indexers** -> **Add new indexer** -> **Add custom newznab indexer**. Name it the name of your indexer, enter their API url (found on profile page) as well as API key (same place). If they declare an API hit limit or download limit, fill those out. Finally, press Submit. Do this for as many indexers as you have.

    Then, proceed to [SABnzbd](http://localhost:8080). Choose the language you know best and then press **Start Wizard**. Fill out *one* of your Usenet servers here and then press **Test Server** to check. If all's well, press **Next**. Finally, press **Go to SABnzbd**. Click the cog in the top right, go to **Servers** and if you have a *second* (or more) Usenet server(s) to add then add them here in the same fashion.

    Go to **Categories** and delete the default ones (tv, movies, software, audio) and add:
    
    1. sonarr
    2. radarr

    Go to **General** and fill in **SABnzbd Username** and **SABnzbd Password**. While there, also regenerate the **API Key** and **NZB Key**. Finally, click **Save Changes** at the bottom and copy the **API Key** to the clipboard.

    Go to Sonarr and Radarr and add SABnzbd and NZBHydra:

    1. **Settings** -> **Download Client** -> **+** -> **SABnzbd**
    - **Name**: SABnzbd
    - **Host**: sabnzbd
    - **Port**: 8080
    - **API Key**: API Key you copied previously
    - **Username**: Username you entered previously
    - **Password**: Password you entered previously
    - **Category**: sonarr/radarr (depending on which app you are setting up)

    2. **Settings** -> **Indexers** -> **+** -> **Newznab**
    - **Name**: NZBHydra
    - **URL**: http://nzbhydra:5076
    - **API Key**: API Key you copied previously

    For Sonarr:
    - **Categories**: 5000,5020,5030,5040,5050,5060
    - **Anime Categories**: 5070

    For Radarr:
    - **Categories**: 2000,2020,2030,2040,2050,2060

    Test and save.

    Finally, go to **Settings** -> **Profiles** -> **Delay Profiles** and press the wrench on the right. Change **Protocol** to **Prefer Usenet** and profit.

7. Last of all, head over to [Plex](http://localhost:32400/web) and set it on up. This one is self explanatory.

# adding indexers to Jackett
1. **Add indexer**
2. Click **Type** until **Semi-Private** is on top
3. Scroll until **Type** is showing **Public**
4. Choose one you like, and press **+**
5. Back at the main screen, click **Copy Torznab Feed**
6. If the indexer is for TV, go to Sonarr. If for movies, go to Radarr. If both, go to both:
-  **Settings** -> **Indexers** -> **+** -> **Torznab Custom**
-  **Name**: whatever
-  **URL**: paste clipboard
- **API Key**: copy from Jackett
7. Click the wrench on your indexer and glance down at **Capabilities**:
- Add the requisite **Category** numbers to the **Categories** field in Sonarr/Radarr. If the category represents anime, such as TV/Anime or Movie/Animation then put it in the **Anime Categories** field. 
8. Test and save.