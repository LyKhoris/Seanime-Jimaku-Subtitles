# Documentation: https://seanime.gitbook.io/seanime-extensions
Generated on 2025-12-15 06:56:20



<!-- Page: https://seanime.gitbook.io/seanime-extensions/seanime/core-apis -->


# Core APIs

## Types

Create a `core.d.ts` file containing the following content:

[<https://raw.githubusercontent.com/5rahim/seanime/refs/heads/main/internal/extension_repo/goja_plugin_types/core.d.ts>](<https://raw.githubusercontent.com/5rahim/seanime/refs/heads/main/internal/extension_repo/goja_plugin_types/core.d.ts>)

## Examples

### User preference

Returns the user preference value from [#add-user-configuration-optional](https://seanime.gitbook.io/seanime-extensions/content-providers/write-test-share#add-user-configuration-optional "mention")

```typescript
$getUserPreference("apiToken")
```

### Console

```typescript
console.log()
console.warn()
console.error()
```

### Fetch

**Example**


```typescript
/// <reference path="./core.d.ts" />

const res = await fetch("https://jsonplaceholder.typicode.com/todos/1")
const data = res.json()
```



### Doc

Parse HTML

```typescript
const $ = LoadDoc(`
<section id="content">
    <article class="post" data-id="1">
        <h2>First Post</h2>
        <p>This is the first post.</p>
        <a href="https://example.com/first-post" class="read-more">Read more</a>
    </article>
    <article class="post" data-id="2">
        <h2>Second Post</h2>
        <p>This is the second post.</p>
        <a href="https://example.com/second-post" class="read-more">Read more</a>
    </article>
    <article class="post" data-id="3">
        <h2>Third Post</h2>
        <p>This is the third post.</p>
        <a href="https://example.com/third-post" class="read-more">Read more</a>
    </article>
</section>
`);

const titles = $("section")
    .children("article.post")
    .filter((i, e) => e.attr("data-id") !== "1")
    .map((i, e) => e.children("h2").text())

console.log(titles) // [Second Post, Third Post] 
```

### ChromeDP

Headless browser powered by the user's installed Chrome/Chromium binary.

> **Warning**
>
> This is experimental.
> 
> Do not forget to call `close()`    &#x20;


```typescript
// -------- Content Providers ----------

const browser = await ChromeDP.newBrowser();
await browser.navigate("https://example.com");

// Wait for the dynamic item to appear
await browser.waitVisible("#dynamic-item");

// Get the text of the dynamic item
const itemText = await browser.text("#dynamic-item");

await browser.close();

// ------------- Plugins ---------------
function init() {
    $ui.register((ctx: $ui.Context) => {
        async function doSomething() {
            const browser = await ctx.chromeDP.newBrowser()
            // ...
            await browser.close()
        }
    })
}
```

### CryptoJS

```typescript
let message = "seanime";
let key = CryptoJS.enc.Utf8.parse("secret key");

console.log("Message:", message);

let encrypted = CryptoJS.AES.encrypt(message, key);
console.log("Encrypted:", encrypted); // map[iv toString]
console.log("Encrypted.toString():", encrypted.toString()); // AoHrnhJfbRht2idLHM82WdkIEpRbXufnA6+ozty9fbk=
console.log("Encrypted.toString(CryptoJS.enc.Base64):", encrypted.toString(CryptoJS.enc.Base64)); // AoHrnhJfbRht2idLHM82WdkIEpRbXufnA6+ozty9fbk=

let decrypted = CryptoJS.AES.decrypt(encrypted, key);
console.log("Decrypted:", decrypted.toString(CryptoJS.enc.Utf8));

let iv = CryptoJS.enc.Utf8.parse("3134003223491201");
encrypted = CryptoJS.AES.encrypt(message, key, { iv: iv });
console.log("Encrypted:", encrypted); // map[iv toString]

decrypted = CryptoJS.AES.decrypt(encrypted, key);
console.log("Decrypted without IV:", decrypted.toString(CryptoJS.enc.Utf8)); // "" <- Nothing

decrypted = CryptoJS.AES.decrypt(encrypted, key, { iv: iv });
console.log("Decrypted with IV:", decrypted.toString(CryptoJS.enc.Utf8)); // seanime


let a = CryptoJS.enc.Utf8.parse("Hello, World!");
console.log(a); // // Uint8Array [72 101 108 108 ...]

let b = CryptoJS.enc.Base64.stringify(a);
console.log(b); // SGVsbG8sIFdvcmxkIQ==

let c = CryptoJS.enc.Base64.parse(b);
console.log(c); // Uint8Array [72 101 108 108 ...]

let d = CryptoJS.enc.Utf8.stringify(c);
console.log(d); // Hello, World!
```

### $habari

Filename parser

```typescript
data := $habari.parse("Hyouka (2012) S1-2 [BD 1080p HEVC OPUS] [Dual-Audio]")
console.log(data.title)            // Hyouka
console.log(data.formatted_title)  // Hyouka (2012)
console.log(data.year)             // 2012
console.log(data.season_number)    // ["1", "2"]
console.log(data.video_resolution) // 1080p
```

### $clone

Useful for safely handling events.

```typescript
function init() {
    $app.onPreUpdateEntryEvent(e => {
        $store.set("onPreUpdateEntry", $clone(e))
    })
}
```

### $replace

The `$replace` function is used to overwrite properties of an object within an event.

> **Warning**
>
> This only works on values that are not undefined and on values that are references under the hood.


{% tabs %}
{% tab title="How it works" %}
{% code overflow="wrap" %}

```typescript
$app.onGetAnime((e) => {
    if(e.anime.id === 130003) {
        console.log(e.anime.title)
        // { 
        //    "english": "Bocchi the Rock!",
        //    "romaji": "Bocchi the Rock!", 
        //    "userPreferred": "Bocchi the Rock!"
        // }
    
        e.anime.title = { "english": "The One Piece is Real" }
        console.log(e.anime.title) 
        // { 
        //    "english": "The One Piece is Real",
        //    "romaji": "Bocchi the Rock!", 
        //    "userPreferred": "Bocchi the Rock!"
        // }
    
        // ✅ Overwrite the entire 'title' object
        $replace(e.anime.title, { "english": "The One Piece is Real" })
        console.log(e.anime.title)
        // { 
        //    "english": "The One Piece is Real",
        //    "romaji": undefined, 
        //    "userPreferred": undefined
        // }
        
        e.anime.synonyms[0] = "The One Piece" // ✅ Works
        $replace(e.anime.synonyms[0], "The One Piece") // ✅ Works
        
        // ⛔️ Doesn't work because 'id' is not a reference under the hood
        $replace(e.anime.id, 22)
        // ⛔️ Doesn't work if 'bannerImage' is undefined
        $replace(e.anime.bannerImage, "abc")
    }
    
    e.next();
})
```


{% endtab %}
{% endtabs %}

### $torrentUtils

```typescript
// Get a magnet link from a torrent file content
const res = await fetch("http://[...].torrent")
const content = res.text()
$torrentUtils.getMagnetLinkFromTorrentData(content)
```

### $toString

Converts binary data to string.

```typescript
const uint8Array = new Uint8Array(new ArrayBuffer(5));
uint8Array[0] = 104;
uint8Array[1] = 101;
uint8Array[2] = 108;
uint8Array[3] = 108;
uint8Array[4] = 111;

console.log($toString(uint8Array)); // hello
```

### $toBytes

Similar to the Web API `TextEncoder.encode`

```typescript
const b = $toBytes("hello")
console.log(b); // Uint8Array [104, 101, 108, 108, 111]

console($toString(b)); // hello
```

### $sleep

> **Warning**
>
> Use carefully


```typescript
// sleeps for 1s
$sleep(1000)
```



<!-- Page: https://seanime.gitbook.io/seanime-extensions/seanime/changelog -->


# Changelog

## v3.0.0

* Custom source extensions
* `isDrawer` prop to `tray.newTray`

## v2.8.4

* `app.d.ts`, `plugin.d.ts` have been fixed and updated
* `ctx.fieldRef` can now take a default value
* New `ctx.eventHandler(callback)` for inlined event handler registration
* New `className` prop for tray components



<!-- Page: https://seanime.gitbook.io/seanime-extensions/content-providers/write-test-share -->


# Write, test, share

Content providers are a type of extension used to add more sources to existing features in Seanime.

* Anime torrent providers
* Manga providers
* Online streaming providers
* Custom sources

## 1. Write and test

### Code the extension

{% content-ref url="anime-torrent-provider" %}
[anime-torrent-provider](https://seanime.gitbook.io/seanime-extensions/content-providers/anime-torrent-provider)
{% endcontent-ref %}

{% content-ref url="manga-provider" %}
[manga-provider](https://seanime.gitbook.io/seanime-extensions/content-providers/manga-provider)
{% endcontent-ref %}

{% content-ref url="online-streaming-provider" %}
[online-streaming-provider](https://seanime.gitbook.io/seanime-extensions/content-providers/online-streaming-provider)
{% endcontent-ref %}

{% content-ref url="custom-source" %}
[custom-source](https://seanime.gitbook.io/seanime-extensions/content-providers/custom-source)
{% endcontent-ref %}

### Test in the playground

1. Go to the `Extensions` page in Seanime.
2. Click on the `Playground` dropdown option.

<img src="https://i.postimg.cc/fyy9xGmG/Clean-Shot-2024-08-25-at-14-30-362x.webp" alt="Playground" data-size="original">

3. Select which type of extension you want to test and enter the code.

You will be able to select the **method (function)** you want to test. Different methods have different **simulation parameters** based on real in-app usage.<br>

<figure><img src="https://266901462-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F7Mat9fLDAotSl6o8o4P3%2Fuploads%2FkxxyvzknBvrL73Uspr8A%2Fimage.png?alt=media&#x26;token=36258759-33d0-4c2b-9b7d-3747b5eb6fd5" alt=""><figcaption></figcaption></figure>

## 2. Create a manifest file

### Create the file

> **Warning**
>
> Make the ID unique in order to avoid conflicts.
> 
> The name of the file should be the same as the ID.


**my-original-extension-id.json**


```json
{
    "id": "my-original-extension-id",
    "name": "My Extension Name",
    "description": "My Extension Description",
    "manifestURI": "",
    "version": "1.0.0",
    "author": "Author Name",
    "type": "",
    "language": "",
    "lang": "",
    "payload": ""
}
```



* `id`: ID of your extension.
* `name`: The name of the extension.
* `description`: A short description of the extension.
* `manifestURI`: The URI where the manifest file is hosted. Used by Seanime to check for updates. This can be empty if you don’t plan on hosting and sharing your extension.
* `version`: The version of the extension. `x.x.x` (e.g. 0.1.0)
* `author`: The author of the extension.
* `type`: The type of extension. See below for the available types.
* `anime-torrent-provider`, `manga-provider`, `onlinestream-provider` , `custom-source`
* `language`: The **programming language** of the extension.
* Can be **`typescript`, or `javascript`**.
* `lang`: **ISO 639-1** language of the extension’s content (e.g. “en”, “fr” etc.).
* Set it to **`multi`** if your extension supports multiple languages.

### Paste the payload

You have two options:

1. Paste the code of your extension in the `payload` field.
2. Paste a URL to the code of your extension in the `payloadURI` field and remove `payload` empty.

## 3. Share

If you want to share your extension with others, you can host the manifest file on GitHub and [share](https://seanime.rahim.app/community/extensions) the link to the file.

If you just want to use it for yourself, just place the JSON file in the `extensions` directory in your [data directory](https://seanime.rahim.app/docs/config#data-directory).

## 4. Update your extension

This is a simple process. Just update the `version` field in the JSON file and paste the new code in the `payload` field.

> **Warning**
>
> Your extension might become incompatible with a later version of Seanime.
> 
> Check the [Extension Changelog](https://seanime.gitbook.io/seanime-extensions/seanime/changelog) for breaking changes and update your code accordingly.


<figure><img src="https://i.postimg.cc/RVzjPvNQ/Clean-Shot-2024-08-27-at-18-49-172x.webp" alt="" width="375"><figcaption></figcaption></figure>

> **Warning**
>
> Do not change your extension ID between updates


## Add user configuration (optional)

You can make it so users can enter arbitrary values that you can use in variables inside your code. This is useful when your extension needs to use a personal API key for example.

<details>

<summary>Guide</summary>

<img src="https://266901462-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F7Mat9fLDAotSl6o8o4P3%2Fuploads%2FChRoJaG3DImgeDNpyyCm%2Fimage.png?alt=media&#x26;token=a059d8d0-25b0-4454-8936-0544a2cfac4a" alt="" data-size="original">

* Declare any number of **string** variables containing the configuration field keys you want to accept in the format `{{key}}`. These variables will be replaced with the values the user entered when the extension is loaded.
* In your manifest file, add a `userConfig` field.

> **Info**
>
> The field's 'name' should be the same as the key between the double curly brackets in your code.


```json
{
    //...
    "userConfig": {
        "requiresConfig": true,
        "version": 1,
        "fields": [
            {
                "name": "api",
                "label": "API URL",
                "type": "text",
                "default": "https://feed.animetosho.org/json"
            },
            {
                "name": "withSmartSearch",
                "label": "Enable Smart Search",
                "type": "switch",
                "default": "true"
            },
            {
                "name": "type",
                "label": "Provider Type",
                "type": "select",
                "default": "main",
                "options": [
                    {
                        "label": "Main",
                        "value": "main"
                    },
                    {
                        "label": "Special",
                        "value": "special"
                    }
                ]
            }
        ]
    }
}
```

* `requiresConfig`: Set to `true` to force the user to validate the configuration before the extension is loaded.
* `version`: The version of the configuration. Increment this number when you make changes to the configuration fields of your extension.

</details>



<!-- Page: https://seanime.gitbook.io/seanime-extensions/content-providers/anime-torrent-provider -->


# Anime Torrent Provider

> **Success**
>
> Difficulty: Easy


<details>

<summary>Use bootstrapping command</summary>

You can use this third-party tool to help you quickly bootstrap a folder locally

```bash
npx seanime-tool g-template
```

</details>

## Types

**anime-torrent-provider.d.ts**


```typescript
declare type AnimeProviderSmartSearchFilter = "batch" | "episodeNumber" | "resolution" | "query" | "bestReleases"
 
declare type AnimeProviderType = "main" | "special"
 
declare interface AnimeProviderSettings {
    // Indicates whether the extension supports smart search.
    canSmartSearch: boolean
    // Filters that can be used in smart search.
    smartSearchFilters: AnimeProviderSmartSearchFilter[]
    // Indicates whether the extension supports adult content.
    supportsAdult: boolean
    // Type of the provider.
    type: AnimeProviderType
}
 
// Media object passed to 'search' and 'smartSearch' methods.
declare interface Media {
    // AniList ID of the media.
    id: number
    // MyAnimeList ID of the media.
    idMal?: number
    // e.g. "FINISHED", "RELEASING", "NOT_YET_RELEASED", "CANCELLED", "HIATUS"
	// This will be set to "NOT_YET_RELEASED" if the status is unknown.
    status?: string
    // e.g. "TV", "TV_SHORT", "MOVIE", "SPECIAL", "OVA", "ONA", "MUSIC"
    // This will be set to "TV" if the format is unknown.
    format?: string
    // e.g. "Attack on Titan"
    englishTitle?: string
    // e.g. "Shingeki no Kyojin"
    romajiTitle?: string
    // TotalEpisodes is total number of episodes of the media.
    // This will be -1 if the total number of episodes is unknown / not applicable.
    episodeCount?: number
    // Absolute offset of the media's season.
    // This will be 0 if the media is not seasonal or the offset is unknown.
    absoluteSeasonOffset?: number
    // All alternative titles of the media.
    synonyms: string[]
    // Whether the media is NSFW.
    isAdult: boolean
    // Start date of the media.
    // This will be undefined if it has no start date.
    startDate?: FuzzyDate
}
 
declare interface FuzzyDate {
    year: number
    month?: number
    day?: number
}
 
declare interface AnimeSearchOptions {
    // The media object.
    media: Media
    // The user search query.
    query: string
}
 
declare interface AnimeSmartSearchOptions {
    // The media object.
    media: Media
    // The user search query.
    // This will be empty if your extension does not support custom queries.
    query: string
    // Indicates whether the user wants to search for batch torrents.
    // This will be false if your extension does not support batch torrents.
    batch: boolean
    // The episode number the user wants to search for.
    // This will be 0 if your extension does not support episode number filtering.
    episodeNumber: number
    // The resolution the user wants to search for.
    // This will be empty if your extension does not support resolution filtering.
    resolution: string
    // AniDB Anime ID of the media.
    anidbAID: number
    // AniDB Episode ID of the media.
    anidbEID: number
    // Indicates whether the user wants to search for the best releases.
    // This will be false if your extension does not support filtering by best releases.
    bestReleases: boolean
}
 
declare interface AnimeTorrent {
    name: string
    // Date of the torrent.
	// The date should have RFC3339 format. e.g. "2006-01-02T15:04:05Z07:00"
    date: string
    // Size of the torrent in bytes.
    size: number
    // Formatted size of the torrent. e.g. "1.2 GB"
    // Leave this empty if you want Seanime to format the size.
    formattedSize: string
    // Number of seeders of the torrent.
    seeders: number
    // Number of leechers of the torrent.
    leechers: number
    // Number of downloads of the torrent.
    downloadCount: number
    // Link to the torrent page.
    link: string
    // Download URL of the torrent.
    // Leave this empty if you cannot provide a direct download URL.
    downloadUrl?: string
    // Magnet link of the torrent.
    // Set this to null if you cannot provide a magnet link without scraping.
    magnetLink?: string
    // Info hash of the torrent.
    // Set this to null if you cannot provide an info hash without scraping.
    infoHash?: string
    // The resolution of the torrent.
    // Leave this empty if you want Seanime to parse the resolution from the name.
    resolution?: string
    // Set this to true if you can confirm that the torrent is a batch.
    // Else, Seanime will parse the torrent name to determine if it's a batch.
    isBatch?: boolean
    // Episode number of the torrent.
    // Return -1 if unknown / unable to determine and Seanime will parse the torrent name.
    episodeNumber: number
    // Release group of the torrent.
    // Leave this empty if you want Seanime to parse the release group from the name.
    releaseGroup?: string
    // Set this to true if you can confirm that the torrent is the best release.
    isBestRelease: boolean
    // Set this to true if you can confirm that the torrent matches the anime the user is searching for.
    // e.g. If the torrent was found using the AniDB anime or episode ID
    confirmed: boolean
}
```



## Code

> **Warning**
>
> Do not change the name of the class. It must be Provider.


```typescript
/// <reference path="./anime-torrent-provider.d.ts" />

class Provider {
    private api = "https://example.com"
		
    // Returns the provider settings.
    async getSettings(): AnimeProviderSettings {
	// TODO: Edit this
         return {
            canSmartSearch: true,
            smartSearchFilters: ["batch", "episodeNumber", "resolution"],
            supportsAdult: false,
            type: "main",
        }
    }
    // Returns the search results depending on the query.
    async search(opts: AnimeSearchOptions): Promise<AnimeTorrent[]> {
	// TODO
        return []
    }
    // Returns the search results depending on the search options.
    async smartSearch(opts: AnimeSmartSearchOptions): Promise<AnimeTorrent[]> {
	// TODO
        return []
    }
    // Scrapes the torrent page to get the info hash.
    // If already present in AnimeTorrent, this should just return the info hash without scraping.
    async getTorrentInfoHash(torrent: AnimeTorrent): Promise<string> {
        return torrent.infoHash
    }
    // Scrapes the torrent page to get the magnet link.
    // If already present in AnimeTorrent, this should just return the magnet link without scraping.
    async getTorrentMagnetLink(torrent: AnimeTorrent): Promise<string> {
        return torrent.magnetLink
    }
    // Returns the latest torrents.
    // Note that this is only used by "main" providers.
    async getLatest(): Promise<AnimeTorrent[]> {
	// TODO
        return []
    }
}
```

### Settings

#### type

* `main`: Your extension can be used as **default provider** for torrent search and the Auto Downloader.
* `special`: Your extension can **ONLY** be used for torrent search.

#### canSmartSearch / smartSearchFilters

<figure><img src="https://266901462-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F7Mat9fLDAotSl6o8o4P3%2Fuploads%2FqGAduQs9DgfN9RhR15yL%2Fimg-2025-03-16-16-13-14.png?alt=media&#x26;token=27690df9-db8d-47ad-814c-151a9cac6116" alt=""><figcaption></figcaption></figure>

* `batch` : Your extension can look for batches
* `episodeNumber` : Your extension can look for specific episode numbers
* `resolution` : Your extension can filter by resolution
* `query`: Allow the user to change the smart search title
* `bestReleases` : Your extension can find highest-quality torrents

## Example

```typescript
/// <reference path="./anime-torrent-provider.d.ts" />
/// <reference path="./core.d.ts" />

class Provider {

    api = "https://feed.animetosho.org/json"

    getSettings(): AnimeProviderSettings {
        return {
            canSmartSearch: true,
            smartSearchFilters: ["batch", "episodeNumber", "resolution"],
            supportsAdult: false,
            type: "main",
        }
    }

    async search(opts: AnimeSearchOptions): Promise<AnimeTorrent[]> {
        const query = `?q=${encodeURIComponent(opts.query)}&only_tor=1`
        console.log(query)
        const torrents = await this.fetchTorrents(query)
        return torrents.map(t => this.toAnimeTorrent(t))
    }

    async smartSearch(opts: AnimeSmartSearchOptions): Promise<AnimeTorrent[]> {
        const ret: AnimeTorrent[] = []

        if (opts.batch) {
            if (!opts.anidbAID) return []

            let torrents = await this.searchByAID(opts.anidbAID, opts.resolution)

            if (!(opts.media.format == "MOVIE" || opts.media.episodeCount == 1)) {
                torrents = torrents.filter(t => t.num_files > 1)
            }

            for (const torrent of torrents) {
                const t = this.toAnimeTorrent(torrent)
                t.isBatch = true
                ret.push(t)
            }

            return ret
        }

        if (!opts.anidbEID) return []

        const torrents = await this.searchByEID(opts.anidbEID, opts.resolution)

        for (const torrent of torrents) {
            ret.push(this.toAnimeTorrent(torrent))
        }

        return ret
    }

    async getTorrentInfoHash(torrent: AnimeTorrent): Promise<string> {
        return torrent.infoHash || ""
    }

    async getTorrentMagnetLink(torrent: AnimeTorrent): Promise<string> {
        return torrent.magnetLink || ""
    }

    async getLatest(): Promise<AnimeTorrent[]> {
        const query = `?q=&only_tor=1`
        const torrents = await this.fetchTorrents(query)
        return torrents.map(t => this.toAnimeTorrent(t))
    }

    async searchByAID(aid: number, quality: string): Promise<ToshoTorrent[]> {
        const q = encodeURIComponent(this.formatQuality(quality))
        const query = `?order=size-d&aid=${aid}&q=${q}`
        return this.fetchTorrents(query)
    }

    async searchByEID(eid: number, quality: string): Promise<ToshoTorrent[]> {
        const q = encodeURIComponent(this.formatQuality(quality))
        const query = `?eid=${eid}&q=${q}`
        return this.fetchTorrents(query)
    }

    async fetchTorrents(url: string): Promise<ToshoTorrent[]> {
        const furl = `${this.api}${url}`

        try {
            const response = await fetch(furl)

            if (!response.ok) {
                throw new Error(`Failed to fetch torrents, ${response.statusText}`)
            }

            const torrents: ToshoTorrent[] = await response.json()

            return torrents.map(t => {
                if (t.seeders > 30000) {
                    t.seeders = 0
                }
                if (t.leechers > 30000) {
                    t.leechers = 0
                }
                return t
            })
        }
        catch (error) {
            throw new Error(`Error fetching torrents: ${error}`)
        }
    }

    formatQuality(quality: string): string {
        return quality.replace(/p$/, "")
    }

    toAnimeTorrent(torrent: ToshoTorrent): AnimeTorrent {
        return {
            name: torrent.title,
            date: new Date(torrent.timestamp * 1000).toISOString(),
            size: torrent.total_size,
            formattedSize: "",
            seeders: torrent.seeders,
            leechers: torrent.leechers,
            downloadCount: torrent.torrent_download_count,
            link: torrent.link,
            downloadUrl: torrent.torrent_url,
            magnetLink: torrent.magnet_uri,
            infoHash: torrent.info_hash,
            resolution: "",
            isBatch: false,
            episodeNumber: -1,
            isBestRelease: false,
            confirmed: true,
        }
    }
}

type ToshoTorrent = {
    id: number
    title: string
    link: string
    timestamp: number
    status: string
    tosho_id?: number
    nyaa_id?: number
    nyaa_subdom?: any
    anidex_id?: number
    torrent_url: string
    info_hash: string
    info_hash_v2?: string
    magnet_uri: string
    seeders: number
    leechers: number
    torrent_download_count: number
    tracker_updated?: any
    nzb_url?: string
    total_size: number
    num_files: number
    anidb_aid: number
    anidb_eid: number
    anidb_fid: number
    article_url: string
    article_title: string
    website_url: string
}

```



<!-- Page: https://seanime.gitbook.io/seanime-extensions/content-providers/manga-provider -->


# Manga Provider

> **Success**
>
> Difficulty: Easy


<details>

<summary>Use bootstrapping command</summary>

You can use this third-party tool to help you quickly bootstrap a folder locally

```bash
npx seanime-tool g-template
```

</details>

## Types

**manga-provider.d.ts**


```typescript
declare type SearchResult = {
    id: string
    title: string
    synonyms?: string[]
    year?: number
    image?: string
}

declare type ChapterDetails = {
    id: string
    url: string
    title: string
    chapter: string
    index: number
    scanlator?: string
    language?: string
    rating?: number
    updatedAt?: string
}

declare type ChapterPage = {
    url: string
    index: number
    headers: { [key: string]: string }
}

declare type QueryOptions = {
    query: string
    year?: number
}

declare type Settings = {
    supportsMultiLanguage?: boolean
    supportsMultiScanlator?: boolean
}

declare abstract class MangaProvider {
    search(opts: QueryOptions): Promise<SearchResult[]>
    findChapters(id: string): Promise<ChapterDetails[]>
    findChapterPages(id: string): Promise<ChapterPage[]>

    getSettings(): Settings
}
```



## Code

> **Warning**
>
> Do not change the name of the class. It must be Provider.


```typescript
/// <reference path="./manga-provider.d.ts" />
 
class Provider {
    private api = "https://example.com"
		
    getSettings(): Settings {
        return {
            supportsMultiLanguage: false,
            supportsMultiScanlator: false,
        }
    }

    // Returns the search results based on the query.
    async search(opts: QueryOptions): Promise<SearchResult[]> {
	// TODO
        return [{
            id: "999",
            title: "Manga Title",
            synonyms: ["Synonym 1", "Synonym 2"],
            year: 2021,
            image: "https://example.com/image.jpg",
        }]
    }
    
    // Returns the chapters based on the manga ID.
    // The chapters should be sorted in ascending order (0, 1, ...).
    async findChapters(mangaId: string): Promise<ChapterDetails[]> {
	// TODO
        return [{
            id: `999-chapter-1`,
            url: "https://example.com/manga/999-chapter-1",
            title: "Chapter 1",
            chapter: "1",
            index: 0,
        }]
    }
    
    // Returns the chapter pages based on the chapter ID.
    // The pages should be sorted in ascending order (0, 1, ...).
    async findChapterPages(chapterId: string): Promise<ChapterPage[]> {
	// TODO
        return [{
            url: "https://example.com/manga/999-chapter-1/page-1.jpg",
            index: 0,
            headers: {
                "Referer": "https://example.com/manga/999/chapter-1",
            },
        }]
    }
}
```

### Workflow

`search` is called twice when the user opens the manga page. Each time with a different manga title as query (English, Romaji).

The best match will automatically be selected and `findChapters` will be called with the manga ID from the search result to get the list of chapters.

<figure><img src="https://266901462-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F7Mat9fLDAotSl6o8o4P3%2Fuploads%2FUF1kuTZ5qQrX9OL247aY%2Fimage.png?alt=media&#x26;token=75ef0a74-5eb2-4c77-936c-c22386727abe" alt="" width="563"><figcaption></figcaption></figure>

`findChapterPages` is called when the user requests to read or download the chapter.

<figure><img src="https://266901462-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F7Mat9fLDAotSl6o8o4P3%2Fuploads%2FLIDGusZF6pPtwd1q3Sxy%2Fimage.png?alt=media&#x26;token=2e50044c-e6c1-4a26-98b4-bd36a7525dc2" alt=""><figcaption></figcaption></figure>

### Manga ID, Chapter ID

Depending on the source website you’re getting the data from, the URLs might get a little complex.

For example, if a manga’s chapter page is: [`https://example.com/manga/999/chapter-1`](https://example.com/manga/999/chapter-1) consisting of 2 URL sections (in this case, the manga ID and the chapter ID), you can construct the Seanime chapter ID by combining the two parts and splitting them in `findChapterPages` .

<figure><img src="https://266901462-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F7Mat9fLDAotSl6o8o4P3%2Fuploads%2FSizqGEViam7MUer2BGdA%2Fimage.png?alt=media&#x26;token=30a9cb41-3c50-4000-8d63-969a85415e24" alt="" width="375"><figcaption></figcaption></figure>

### Settings

* If your manga source supports multiple languages for chapters and you want your extension to give this option to the users, set `supportsMultiLanguage` to `true` and set the `language` property for each of the `ChapterDetails`. Preferably  [ISO 639-1](https://en.wikipedia.org/wiki/ISO_639-1).
* Similarly, you can also give the option to choose a scanlator by setting `supportsMultiScanlator` to `true` and setting the `scanlator` property for each of the `ChapterDetails`.

<figure><img src="https://266901462-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F7Mat9fLDAotSl6o8o4P3%2Fuploads%2FjJGEyZuLpm8gvDUiPDKl%2Fimage.png?alt=media&#x26;token=ed43ab5d-9c3f-47ad-8550-3bac01d08582" alt=""><figcaption></figcaption></figure>

## Example

```typescript
/// <reference path="./manga-provider.d.ts" />

class Provider {

    private api = "https://api.comick.fun"
    
    getSettings(): Settings {
	    return {
		    supportsMultiLanguage: false,
		    supportsMultiScanlator: false,
	    }
    }

    async search(opts: QueryOptions): Promise<SearchResult[]> {
        console.log(this.api, opts.query)

        const requestRes = await fetch(`${this.api}/v1.0/search?q=${encodeURIComponent(opts.query)}&limit=25&page=1`, {
            method: "get",
        })
        const comickRes = await requestRes.json() as ComickSearchResult[]

        const ret: SearchResult[] = []

        for (const res of comickRes) {

            let cover: any = res.md_covers ? res.md_covers[0] : null
            if (cover && cover.b2key != undefined) {
                cover = "https://meo.comick.pictures/" + cover.b2key
            }

            ret.push({
                id: res.hid,
                title: res.title ?? res.slug,
                synonyms: res.md_titles?.map(t => t.title) ?? {},
                year: res.year ?? 0,
                image: cover,
            })
        }

        return ret
    }

    async findChapters(id: string): Promise<ChapterDetails[]> {

        console.log("Fetching chapters", id)

        const chapterList: ChapterDetails[] = []

        const data = (await (await fetch(`${this.api}/comic/${id}/chapters?lang=en&page=0&limit=1000000`))?.json()) as { chapters: ComickChapter[] }

        const chapters: ChapterDetails[] = []

        for (const chapter of data.chapters) {

            if (!chapter.chap) {
                continue
            }

            let title = "Chapter " + this.padNum(chapter.chap, 2) + " "

            if (title.length === 0) {
                if (!chapter.title) {
                    title = "Oneshot"
                } else {
                    title = chapter.title
                }
            }

            let canPush = true
            for (let i = 0; i < chapters.length; i++) {
                if (chapters[i].title?.trim() === title?.trim()) {
                    canPush = false
                }
            }

            if (canPush) {
                if (chapter.lang === "en") {
                    chapters.push({
                        url: `${this.api}/comic/${id}/chapter/${chapter.hid}`,
                        index: 0,
                        id: chapter.hid,
                        title: title?.trim(),
                        chapter: chapter.chap,
                        rating: chapter.up_count - chapter.down_count,
                        updatedAt: chapter.updated_at,
                    })
                }
            }
        }

        chapters.reverse()

        for (let i = 0; i < chapters.length; i++) {
            chapters[i].index = i
        }

        return chapters
    }

    async findChapterPages(id: string): Promise<ChapterPage[]> {

        const data = (await (await fetch(`${this.api}/chapter/${id}`))?.json()) as {
            chapter: { md_images: { vol: any; w: number; h: number; b2key: string }[] }
        }

        const pages: ChapterPage[] = []

        data.chapter.md_images.map((image, index: number) => {
            pages.push({
                url: `https://meo.comick.pictures/${image.b2key}?width=${image.w}`,
                index: index,
                headers: {},
            })
        })

        return pages
    }

    padNum(number: string, places: number): string {
        let range = number.split("-")
        range = range.map((chapter) => {
            chapter = chapter.trim()
            const digits = chapter.split(".")[0].length
            return "0".repeat(Math.max(0, places - digits)) + chapter
        })
        return range.join("-")
    }

}

interface ComickSearchResult {
    title: string;
    id: number;
    hid: string;
    slug: string;
    year?: number;
    rating: string;
    rating_count: number;
    follow_count: number;
    user_follow_count: number;
    content_rating: string;
    created_at: string;
    demographic: number;
    md_titles: { title: string }[];
    md_covers: { vol: any; w: number; h: number; b2key: string }[];
    highlight: string;
}

interface Comic {
    id: number;
    hid: string;
    title: string;
    country: string;
    status: number;
    links: {
        al: string;
        ap: string;
        bw: string;
        kt: string;
        mu: string;
        amz: string;
        cdj: string;
        ebj: string;
        mal: string;
        raw: string;
    };
    last_chapter: any;
    chapter_count: number;
    demographic: number;
    hentai: boolean;
    user_follow_count: number;
    follow_rank: number;
    comment_count: number;
    follow_count: number;
    desc: string;
    parsed: string;
    slug: string;
    mismatch: any;
    year: number;
    bayesian_rating: any;
    rating_count: number;
    content_rating: string;
    translation_completed: boolean;
    relate_from: Array<any>;
    mies: any;
    md_titles: { title: string }[];
    md_comic_md_genres: { md_genres: { name: string; type: string | null; slug: string; group: string } }[];
    mu_comics: {
        licensed_in_english: any;
        mu_comic_categories: {
            mu_categories: { title: string; slug: string };
            positive_vote: number;
            negative_vote: number;
        }[];
    };
    md_covers: { vol: any; w: number; h: number; b2key: string }[];
    iso639_1: string;
    lang_name: string;
    lang_native: string;
}

interface ComickChapter {
    id: number;
    chap: string;
    title: string;
    vol: string | null;
    lang: string;
    created_at: string;
    updated_at: string;
    up_count: number;
    down_count: number;
    group_name: any;
    hid: string;
    identities: any;
    md_chapter_groups: { md_groups: { title: string; slug: string } }[];
}
```



<!-- Page: https://seanime.gitbook.io/seanime-extensions/content-providers/online-streaming-provider -->


# Online Streaming Provider

> **Warning**
>
> Difficulty: Moderate


<details>

<summary>Use bootstrapping command</summary>

You can use this third-party tool to help you quickly bootstrap a folder locally

```bash
npx seanime-tool g-template
```

</details>

## Types

**online-streaming-provider.d.ts**


```typescript
declare type SearchResult = {
    id: string
    title: string
    url: string
    subOrDub: SubOrDub
}

declare type SubOrDub = "sub" | "dub" | "both"

declare type EpisodeDetails = {
    id: string
    number: number
    url: string
    title?: string
}

declare type EpisodeServer = {
    server: string
    headers: { [key: string]: string }
    videoSources: VideoSource[]
}

declare type VideoSourceType = "mp4" | "m3u8" | "unknown"

declare type VideoSource = {
    url: string
    type: VideoSourceType
    // Quality or label of the video source, should be unique (e.g. "1080p", "1080p - English")
    quality: string
    // Secondary label of the video source (e.g. "English")
    label?: string
    subtitles: VideoSubtitle[]
}

declare type VideoSubtitle = {
    id: string
    url: string
    language: string
    isDefault: boolean
}

declare interface Media {
    id: number
    idMal?: number
    status?: string
    format?: string
    englishTitle?: string
    romajiTitle?: string
    episodeCount?: number
    absoluteSeasonOffset?: number
    synonyms: string[]
    isAdult: boolean
    startDate?: FuzzyDate
}

declare interface FuzzyDate {
    year: number
    month?: number
    day?: number
}

declare type SearchOptions = {
    media: Media
    query: string
    dub: boolean
    year?: number
}

declare type Settings = {
    episodeServers: string[]
    supportsDub: boolean
}

declare abstract class AnimeProvider {
    search(opts: SearchOptions): Promise<SearchResult[]>

    findEpisodes(id: string): Promise<EpisodeDetails[]>

    findEpisodeServer(episode: EpisodeDetails, server: string): Promise<EpisodeServer>

    getSettings(): Settings
}

```



## Code

> **Warning**
>
> Do not change the name of the class. It must be Provider.


```typescript
/// <reference path="./online-streaming-provider.d.ts" />
 
class Provider {

    getSettings(): Settings {
        return {
            episodeServers: ["server1", "server2"],
            supportsDub: true,
        }
    }

    async search(query: SearchOptions): Promise<SearchResult[]> {
        return [{
            id: "1",
            title: "Anime Title",
            url: "https://example.com/anime/1",
            subOrDub: "both",
        }]
    }
    async findEpisodes(id: string): Promise<EpisodeDetails[]> {
        return [{
            id: "1",
            number: 1,
            url: "https://example.com/episode/1",
            title: "Episode title",
        }]
    }
    async findEpisodeServer(episode: EpisodeDetails, _server: string): Promise<EpisodeServer> {
        let server = "server1"
        if (_server !== "default") server = _server
        
        return {
            server: server,
            headers: {},
            videoSources: [{
                url: "https://example.com/.../stream.m3u8",
                type: "m3u8",
                quality: "1080p",
                subtitles: [{
                    id: "1",
                    url: "https://example.com/.../subs.vtt",
                    language: "en",
                    isDefault: true,
                }],
            }],
        }
    }
}
```

## Example

```typescript
/// <reference path=".onlinestream-provider.d.ts" />
/// <reference path="./core.d.ts" />

type EpisodeData = {
    id: number; episode: number; title: string; snapshot: string; filler: number; session: string; created_at?: string
}

type AnimeData = {
    id: number; title: string; type: string; year: number; poster: string; session: string
}

class Provider {

    api = "https://example.com"
    headers = { Referer: "https://example.com" }

    getSettings(): Settings {
        return {
            episodeServers: ["kwik"],
            supportsDub: false,
        }
    }

    async search(opts: SearchOptions): Promise<SearchResult[]> {
        const req = await fetch(`${this.api}/api?m=search&q=${encodeURIComponent(opts.query)}`, {
            headers: {
                Cookie: "__ddg1_=;__ddg2_=;",
            },
        })

        if (!req.ok) {
            return []
        }
        const data = (await req.json()) as { data: AnimeData[] }
        const results: SearchResult[] = []

        if (!data?.data) {
            return []
        }

        data.data.map((item: AnimeData) => {
            results.push({
                subOrDub: "sub",
                id: item.session,
                title: item.title,
                url: "",
            })
        })

        return results
    }

    async findEpisodes(id: string): Promise<EpisodeDetails[]> {
        let episodes: EpisodeDetails[] = []

        const req =
            await fetch(
                `${this.api}${id.includes("-") ? `/anime/${id}` : `/a/${id}`}`,
                {
                    headers: {
                        Cookie: "__ddg1_=;__ddg2_=;",
                    },
                },
            )

        const html = await req.text()


        function pushData(data: EpisodeData[]) {
            for (const item of data) {
                episodes.push({
                    id: item.session + "$" + id,
                    number: item.episode,
                    title: item.title && item.title.length > 0 ? item.title : "Episode " + item.episode,
                    url: req.url,
                })
            }
        }

        const $ = LoadDoc(html)

        const tempId = $("head > meta[property='og:url']").attr("content")!.split("/").pop()!

        const { last_page, data } = (await (
            await fetch(`${this.api}/api?m=release&id=${tempId}&sort=episode_asc&page=1`, {
                headers: {
                    Cookie: "__ddg1_=;__ddg2_=;",
                },
            })
        ).json()) as {
            last_page: number;
            data: EpisodeData[]
        }

        pushData(data)

        const pageNumbers = Array.from({ length: last_page - 1 }, (_, i) => i + 2)

        const promises = pageNumbers.map((pageNumber) =>
            fetch(`${this.api}/api?m=release&id=${tempId}&sort=episode_asc&page=${pageNumber}`, {
                headers: {
                    Cookie: "__ddg1_=;__ddg2_=;",
                },
            }).then((res) => res.json()),
        )
        const results = (await Promise.all(promises)) as {
            data: EpisodeData[]
        }[]

        results.forEach((showData) => {
            for (const data of showData.data) {
                if (data) {
                    pushData([data])
                }
            }
        });
        (data as any[]).sort((a, b) => a.number - b.number)

        if (episodes.length === 0) {
            throw new Error("No episodes found.")
        }


        const lowest = episodes[0].number
        if (lowest > 1) {
            for (let i = 0; i < episodes.length; i++) {
                episodes[i].number = episodes[i].number - lowest + 1
            }
        }

        // Remove episode with decimal numbers (those aren't supported)
        episodes = episodes.filter((episode) => Number.isInteger(episode.number))

        return episodes
    }

    async findEpisodeServer(episode: EpisodeDetails, _server: string): Promise<EpisodeServer> {
        const episodeId = episode.id.split("$")[0]
        const animeId = episode.id.split("$")[1]

        console.log(`${this.api}/play/${animeId}/${episodeId}`)

        const req = await fetch(
            `${this.api}/play/${animeId}/${episodeId}`,
            {
                headers: {
                    Cookie: "__ddg1_=;__ddg2_=;",
                },
            },
        )

        const html = await req.text()

        const regex = /https:\/\/kwik\.si\/e\/\w+/g
        const matches = html.match(regex)

        if (matches === null) {
            throw new Error("Failed to fetch episode server.")
        }

        const $ = LoadDoc(html)

        const result: EpisodeServer = {
            videoSources: [],
            headers: this.headers ?? {},
            server: "kwik",
        }

        $("button[data-src]").each(async (_, el) => {
            let videoSource: VideoSource = {
                url: "",
                type: "m3u8",
                quality: "",
                subtitles: [],
            }

            videoSource.url = el.data("src")!
            if (!videoSource.url) {
                return
            }

            const fansub = el.data("fansub")!
            const quality = el.data("resolution")!

            videoSource.quality = `${quality}p - ${fansub}`

            if (el.data("audio") === "eng") {
                videoSource.quality += " (Eng)"
            }

            if (videoSource.url === matches[0]) {
                videoSource.quality += " (default)"
            }

            result.videoSources.push(videoSource)
        })

        const queries = result.videoSources.map(async (videoSource) => {
            try {
                const src_req = await fetch(videoSource.url, {
                    headers: {
                        Referer: this.headers.Referer,
                        "user-agent":
                            "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36 Edg/107.0.1418.56",
                    },
                })

                const src_html = await src_req.text()

                const scripts = src_html.match(/eval\(f.+?\}\)\)/g)
                if (!scripts) {
                    return
                }

                for (const _script of scripts) {
                    const scriptMatch = _script.match(/eval(.+)/)
                    if (!scriptMatch || !scriptMatch[1]) {
                        continue
                    }

                    try {
                        const decoded = eval(scriptMatch[1])
                        const link = decoded.match(/source='(.+?)'/)
                        if (!link || !link[1]) {
                            continue
                        }

                        videoSource.url = link[1]

                    }
                    catch (e) {
                        console.error("Failed to extract kwik link", e)
                    }

                }

            }
            catch (e) {
                console.error("Failed to fetch kwik link", e)
            }
        })

        await Promise.all(queries)

        return result
    }
}
```



<!-- Page: https://seanime.gitbook.io/seanime-extensions/content-providers/custom-source -->


# Custom Source

Note: You cannot test custom sources in the playground. Load them in development mode ([like here](https://seanime.gitbook.io/seanime-extensions/plugins/write-test-share#id-2.-create-a-manifest-file)).

> **Success**
>
> Difficulty: Easy


## Type Definitions

**custom-source.d.ts**


```typescript
/// <reference path="./app.d.ts" />

declare type Settings = {
    supportsAnime: boolean
    supportsManga: boolean
}

declare type ListResponse<T extends $app.AL_BaseAnime | $app.AL_BaseManga> = {
    media: T[]
    page: number
    totalPages: number
    total: number
}

declare abstract class CustomSource {
    getSettings(): Settings

    async getAnime(ids: number[]): Promise<$app.AL_BaseAnime[]>

    async getAnimeMetadata(id: number): Promise<$app.Metadata_AnimeMetadata | null>

    async getAnimeWithRelations(id: number): Promise<$app.AL_CompleteAnime>

    async getAnimeDetails(id: number): Promise<$app.AL_AnimeDetailsById_Media | null>

    async getManga(ids: number[]): Promise<$app.AL_BaseManga[]>

    async listAnime(search: string, page: number, perPage: number): Promise<ListResponse<$app.AL_BaseAnime>>

    async getMangaDetails(id: number): Promise<$app.AL_MangaDetailsById_Media | null>

    async listManga(search: string, page: number, perPage: number): Promise<ListResponse<$app.AL_BaseManga>>
}

```



Keyword search the various $app types used here:

{% embed url="<https://raw.githubusercontent.com/5rahim/seanime/refs/heads/main/internal/extension_repo/goja_plugin_types/app.d.ts>" %}

## Code

> **Warning**
>
> Do not change the name of the class. It must be Provider.


> **Info**
>
> You can define the media objects in an external API and use fetch to retrieve them dynamically.


### Media objects

Under the hood, custom source media are treated like AniList media, which is the reason why you need to return objects following AniList's JSON schemas.

For the media `id`s, you're free to use any number starting from 1. Under the hood, Seanime will automatically convert these IDs to unique numbers to avoid conflicts.

```typescript
/// <reference path="./manga-provider.d.ts" />

const anime: Record<number, $app.AL_CompleteAnime> = {}
const animeMetadata: Record<number, $app.Metadata_AnimeMetadata> = {}
const manga: Record<number, $app.AL_BaseManga> = {}

class Provider implements CustomSource {
    getSettings(): Settings {
        return {
            supportsAnime: true,
            supportsManga: true,
        }
    }

    // Returns all requested anime objects.
    async getAnime(ids: number[]): Promise<$app.AL_BaseAnime[]> {
        let ret: $app.AL_BaseAnime[] = []
        for (const id of ids) {
            if (anime[id]) {
                // Here we make a deep copy and remove the 'relations' attribute
                // this turn AL_CompleteAnime into AL_BaseAnime
                const a = $clone(media[id]) as $app.AL_CompleteAnime
                delete a["relations"]
                ret.push(a)
            }
        }
        return ret
    }

    // Optionally returns the details for an anime (genres, trailer, etc.)
    // Note that not all the fields are used by the client.
    async getAnimeDetails(id: number): Promise<$app.AL_AnimeDetailsById_Media | null> {
        return null
    }

    // Returns the metadata for an anime.
    // This is used for episodes.
    async getAnimeMetadata(id: number): Promise<$app.Metadata_AnimeMetadata | null> {
        return animeMetadata[id]
    }

    // Returns the anime object with its 'relations'.
    // This is only used by the library scanner to build a relation tree.
    async getAnimeWithRelations(id: number): Promise<$app.AL_CompleteAnime> {
        if (media[id]) {
            return media[id] as $app.AL_CompleteAnime
        }
        throw new Error("not found.")
    }

    // Returns all requested manga objects.
    async getManga(ids: number[]): Promise<$app.AL_BaseManga[]> {
        let ret: $app.AL_BaseManga[] = []
        for (const id of ids) {
            if (manga[id]) {
                ret.push(manga[id])
            }
        }
        return ret
    }

    // Optionally returns the manga details.
    // Similarly to getAnimeDetails, not all fields will be used by the client.
    async getMangaDetails(id: number): Promise<$app.AL_MangaDetailsById_Media | null> {
        return null
    }

    // Returns all anime available on the extension.
    async listAnime(search: string, page: number, perPage: number): Promise<ListResponse<$app.AL_BaseAnime>> {
        return {
            media: Object.values(media),
            total: 1,
            page: 1,
            totalPages: 1,
        }
    }

    // Returns all manga available on the extension.
    async listManga(search: string, page: number, perPage: number): Promise<ListResponse<$app.AL_BaseManga>> {
        return {
            media: Object.values(manga),
            total: 1,
            page: 1,
            totalPages: 1,
        }
    }

}
```



<!-- Page: https://seanime.gitbook.io/seanime-extensions/plugins/introduction -->


# Introduction

A plugin is a type of extension that allows for more in-depth customization and addition of new features through multiple APIs.

## What can plugins do?

With the right permissions, a lot. Here is a high overview of what they're capable of:

* Create a tray icon and display dynamic content in the tray
* Add buttons, context menu items, dropdown items to specific places
* Create a dynamic command palette
* Register hooks to modify server-side behavior
* Run commands, access the file system, create, read, edit files etc.
* Communicate with AniList and other APIs
* Fetch and store data
* Manipulate the DOM
* And more

The plugin system is largely inspired by PocketBase.



<!-- Page: https://seanime.gitbook.io/seanime-extensions/plugins/write-test-share -->


# Write, test, share

## 1. Create a JS/TS file

<details>

<summary>Shortcut: Use bootstrapping command</summary>

You can use this third-party tool to help you quickly bootstrap a plugin folder locally

```bash
npx seanime-tool g-template
```

</details>

**my-plugin.ts**


```typescript
function init() {
    // This function is called when the plugin is loaded
    // There is no guarantee as to when exactly the plugin will be loaded at startup
}
```



## 2. Create a manifest file

> **Warning**
>
> The ID should be unique, in case of a conflict with another extension, your plugin might not be loaded.
> 
> The name of the file should be the same as the ID.


Here we're going with Typescript.

We're setting `isDevelopment`to `true` in order to be able to quickly reload it when we make changes. `payloadURI` in this case is the path to the plugin code, it must be an absolute path.

Obviously, before sharing the extension we'll change the `payloadURI` to the URL of the file containing the code and remove `isDevelopment`.

<pre class="language-json" data-title="my-plugin.json"><code class="lang-json">{
    "id": "my-plugin",
    "name": "My Plugin",
    "version": "1.0.0",
    "manifestURI": "",
    "language": "typescript",
<strong>    "type": "plugin",
</strong>    "description": "An example plugin",
    "author": "Seanime",
    "icon": "",
    "website": "",
    "lang": "multi",
<strong>    "payloadURI": "C:/path/to/my-plugin/code.ts",
</strong>    "plugin": {
        "version": "1",
        "permissions": {}
    },
<strong>    "isDevelopment": true
</strong>}
</code></pre>

## 3. Quick overview

### a. Permissions

Some APIs require specific permissions in order to function.

The user of your plugin will need to **grant** them after the installation.

### b. Hooks

You can register hook callbacks to listen to various types of events happening on the server, modify them or execute custom logic. Learn more about hooks in later sections.

> **Info**
>
> **In a nutshell**
> 
> Hooks can be used to listen to or edit server-side events.


For example:

{% code title="Example" overflow="wrap" %}

```typescript
function init() {
   // This hook is triggered before Seanime formats the library data of an anime
   // The event contains the variables that Seanime will use, and you can modify them
   $app.onAnimeEntryLibraryDataRequested((e) => {
      // Setting this to an empty array will cause Seanime to think that the anime
      // has not been downloaded.
      e.entryLocalFiles = []
      
      e.next() // Continue hook chain
   })
}
```



> **Warning**
>
> Each hook handler must call `e.next()` in order for the hook chain listening to that event to proceed. Not calling it will impact other plugins listening to that event.


### c. UI Context

Hooks are great for customizing server-side behavior but most **business logic** and interface interactions will be done in the UI context.

> **Info**
>
> **In a nutshell**
> 
> Think of the UI context as the "main thread" of your plugin and hooks can be thought of as "worker threads".


Hooks and UI Context can be used alongside each other. In the later section you will learn how communication is done between them.

```typescript
// A simple plugin that stores the history of scan durations
function init() {
    $app.onScanCompleted((e) => {
        // Store the scanning duration (in ms)
        $store.set("scan-completed", e.duration)
        
        e.next()
    })
    
    $ui.register((ctx) => {
    
        // Callback is triggered when the value is updated
        $store.watch<number>("scan-completed", (value) => {
            const now = new Date().toISOString().replaceall(".", "_")
            
            // Add the value to the history
            //   Note that this could have been done in the hook callback BUT
            //   the UI context is better suited for business logic
            $storage.set("scan-duration-history."+now, value)
            
            ctx.toast.info(`Scanning took ${value/1000} seconds!`)
        })
    })
}
```

### d. Javascript restrictions

The UI context and each hook callback are run in isolated environments (called runtimes), and thus, cannot share state easily or read global variables.

{% code overflow="wrap" %}

```typescript
const globalVar = 42;

function init() {
    const value = 42;
    
    $app.onGetAnime((e) => {
        console.log(globalVar) // undefined
        console.log(value) // undefined
    })
    
    $ui.register((ctx) => {
        console.log(globalVar) // undefined
        console.log(value) // undefined
    })
}
```



However, you can still share variables between hooks and the UI context using [$store](https://seanime.gitbook.io/seanime-extensions/plugins/apis/store).

{% content-ref url="apis/store" %}
[store](https://seanime.gitbook.io/seanime-extensions/plugins/apis/store)
{% endcontent-ref %}

<figure><img src="https://266901462-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F7Mat9fLDAotSl6o8o4P3%2Fuploads%2F7LbQgNfwe5gmmv7eDKqQ%2Fimg-2025-04-28-10-02-09%402x.png?alt=media&#x26;token=407552a2-42e6-40b3-bb7e-84415c133faf" alt=""><figcaption><p>Diagram of plugin</p></figcaption></figure>

### e. Types

Add the type definition files located here, in addition to `core.d.ts`&#x20;

[<https://raw.githubusercontent.com/5rahim/seanime/refs/heads/main/internal/extension_repo/goja_plugin_types/app.d.ts>](<https://raw.githubusercontent.com/5rahim/seanime/refs/heads/main/internal/extension_repo/goja_plugin_types/app.d.ts>)

[<https://raw.githubusercontent.com/5rahim/seanime/refs/heads/main/internal/extension_repo/goja_plugin_types/plugin.d.ts>](<https://raw.githubusercontent.com/5rahim/seanime/refs/heads/main/internal/extension_repo/goja_plugin_types/plugin.d.ts>)

[<https://raw.githubusercontent.com/5rahim/seanime/refs/heads/main/internal/extension_repo/goja_plugin_types/system.d.ts>](<https://raw.githubusercontent.com/5rahim/seanime/refs/heads/main/internal/extension_repo/goja_plugin_types/system.d.ts>)

**my-plugin.ts**


```typescript
/// <reference path="./plugin.d.ts" />
/// <reference path="./system.d.ts" />
/// <reference path="./app.d.ts" />
/// <reference path="./core.d.ts" />

function init() {
    // Everything is magically typed!
    
    $ui.register((ctx) => {
    
      ctx.dom.onReady(() => {
          console.log("Page loaded!")
      })
      
   })
}
```



## 4. Write and test

You're good to go!

### Code the extension

{% content-ref url="apis" %}
[apis](https://seanime.gitbook.io/seanime-extensions/plugins/apis)
{% endcontent-ref %}

{% content-ref url="ui" %}
[ui](https://seanime.gitbook.io/seanime-extensions/plugins/ui)
{% endcontent-ref %}

{% content-ref url="hooks" %}
[hooks](https://seanime.gitbook.io/seanime-extensions/plugins/hooks)
{% endcontent-ref %}

### Test it live

In order to test your plugin, add the manifest file inside the `extensions` directory which is inside your [data directory](https://seanime.rahim.app/docs/config#data-directory).

Because you've set `isDevelopement` to true in your manifest file, you will be able to manually reload the extension without having to restart the app. It's recommended to test your plugin with the web-app version of Seanime for convenience.

## 5. Share

If you want to share your plugin with others, you can host both the code and manifest file on GitHub and [share](https://seanime.rahim.app/community/extensions) the link to the file.

> **Info**
>
> Make sure to replace `payloadURI` with the URL of the hosted file containing the code.
> 
> Also, remove `isDevelopment` .




<!-- Page: https://seanime.gitbook.io/seanime-extensions/plugins/apis -->


# APIs

- [Helpers](/seanime-extensions/plugins/apis/helpers.md)
- [Store](/seanime-extensions/plugins/apis/store.md): Key-value store.
- [Storage](/seanime-extensions/plugins/apis/storage.md): Persistent storage.
- [Database](/seanime-extensions/plugins/apis/database.md): Interact with parts of Seanime's database.
- [AniList](/seanime-extensions/plugins/apis/anilist.md): Interact with the user's AniList account.
- [System](/seanime-extensions/plugins/apis/system.md): The System APIs give you access to a set of methods for file operations, downloading and more.
- [Permissions](/seanime-extensions/plugins/apis/system/permissions.md): The System APIs give you access to a set of methods for file operations, downloading and more.
- [OS](/seanime-extensions/plugins/apis/system/os.md): OS-agonistic APIs for operating system functionality, such as interacting with the filesystem.
- [Filepath](/seanime-extensions/plugins/apis/system/filepath.md)
- [Commands](/seanime-extensions/plugins/apis/system/commands.md)
- [Buffers, I/O](/seanime-extensions/plugins/apis/system/buffers-i-o.md)
- [MIME](/seanime-extensions/plugins/apis/system/mime.md)



<!-- Page: https://seanime.gitbook.io/seanime-extensions/plugins/ui -->


# UI

- [Basics](/seanime-extensions/plugins/ui/basics.md): Basics of the UI context.
- [User Interface](/seanime-extensions/plugins/ui/user-interface.md)
- [Tray](/seanime-extensions/plugins/ui/user-interface/tray.md)
- [Toast](/seanime-extensions/plugins/ui/user-interface/toast.md): Provide instant feedback in a popup.
- [Screen](/seanime-extensions/plugins/ui/user-interface/screen.md): Observe and control navigation within the app.
- [Command Palette](/seanime-extensions/plugins/ui/user-interface/command-palette.md)
- [Action](/seanime-extensions/plugins/ui/user-interface/action.md)
- [DOM](/seanime-extensions/plugins/ui/user-interface/dom.md): API for DOM manipulation in Seanime plugins. Each DOM operation involves communication between the plugin and the browser, so understanding performance considerations is important.
- [Anime/Library](/seanime-extensions/plugins/ui/anime-library.md)
- [Anime](/seanime-extensions/plugins/ui/anime-library/anime.md)
- [Playback](/seanime-extensions/plugins/ui/anime-library/playback.md): Interact with the playback and auto-tracking system in Seanime.
- [Continuity](/seanime-extensions/plugins/ui/anime-library/continuity.md): Interact with Seanime's watch history system that powers playback resuming.
- [Auto Downloader](/seanime-extensions/plugins/ui/anime-library/auto-downloader.md)
- [Auto Scanner](/seanime-extensions/plugins/ui/anime-library/auto-scanner.md)
- [Filler Manager](/seanime-extensions/plugins/ui/anime-library/filler-manager.md)
- [External Player Link](/seanime-extensions/plugins/ui/anime-library/external-player-link.md)
- [Downloading](/seanime-extensions/plugins/ui/downloading.md)
- [Downloader](/seanime-extensions/plugins/ui/downloading/downloader.md): OS-agnostic API for downloading files asynchronously.
- [Torrent Client](/seanime-extensions/plugins/ui/downloading/torrent-client.md)
- [Other](/seanime-extensions/plugins/ui/other.md)
- [Manga](/seanime-extensions/plugins/ui/other/manga.md)
- [Discord](/seanime-extensions/plugins/ui/other/discord.md)
- [MPV](/seanime-extensions/plugins/ui/other/mpv.md): Interact with the user's MPV instance.



<!-- Page: https://seanime.gitbook.io/seanime-extensions/plugins/hooks -->


# Hooks

> **Danger**
>
> Difficulty: Hard
> 
> * Event-driven understanding required


## List of hooks

<https://seanime.rahim.app/docs/hooks>

## Usage

Hooks should be used carefully as they can introduce undefined behavior and even slow down the app.

Some hooks, like `onGetAnime` , are triggered very often, so it's a good habit to start by logging the event in order to figure out its frequency.&#x20;

You should also avoid expensive calculations or fetch calls in hook handlers unless you can guarantee that the hook is not triggered often.

> **Info**
>
> Any error/exception that happens in a hook handler will result in a server and client error. Test your code carefully.


{% code title="Example" overflow="wrap" %}

```typescript
function init() {
   // This hook is triggered before Seanime formats the library data of an anime
   // The event contains the variables that Seanime will use, and you can modify them
   $app.onAnimeEntryLibraryDataRequested((e) => {
      // Setting this to an empty array will cause Seanime to think that the anime
      // has not been downloaded.
      e.entryLocalFiles = []
      
      e.next() // Continue hook chain
   })
}
```



> **Warning**
>
> Each hook handler must call `e.next()` in order for the hook chain listening to that event to proceed. Not calling it will impact other plugins listening to that event.


## Best Practices

### Editing events

Let's say we want your plugin to change anime banner images based on what custom banner image has been set for that anime. However you want to do it without manipulating the DOM and before the page is even loaded.

We can use `onGetAnimeCollection`  and `onGetRawAnimeCollection` since these are triggered when Seanime fetches the user's anime collection from AniList. Note that this will not change banner images for the same anime if it's fetched using another query (e.g. discover, search).

```typescript
// Triggers the app loads the user's AniList anime collection
$app.onGetAnimeCollection((e) => {
    // 1. Get all the custom banner images
    const bannerImages = $storage.get<Record<string, string | undefined>>('bannerImages');
    if (!bannerImages) {
        e.next()
        return
    }
    if (!e.animeCollection?.mediaListCollection?.lists?.length) {
        e.next()
        return
    }
    
    // 2. Go through all anime in the collection
    for (let i = 0; i < e.animeCollection!.MediaListCollection!.lists!.length; i++) {
        for (let j = 0; j < e.animeCollection!.MediaListCollection!.lists![i]!.entries!.length; j++) {
            const mediaId = e.animeCollection!.MediaListCollection!.lists![i]!.entries![j]!.media!.id
            // 3. If this anime has a custom image, change it
            const bannerImage = bannerImages[mediaId.toString()]
            if (!!bannerImage) {
                e.animeCollection!.MediaListCollection!.lists![i]!.entries![j]!.media!.bannerImage = bannerImage
            }
        }
    }

    // 4. Continue
    e.next()
})

// Do the same with $app.onGetRawAnimeCollection
```

### Listening to events

Let's say we want to make a plugin that stores the history of scanning durations.

```typescript
// ⚠️ Not recommended: Doing unnecessary work in the hook callback
function init() {
    $app.onScanCompleted((e) => {
        const now = new Date().toISOString().replaceall(".", "_")    
        // Add the value to the history
        // NOTE: In reality this operation is very fast
        $storage.set("scan-duration-history."+now, e.duration)
        
        e.next()
    })
    
    $ui.register((ctx) => {
        
    })
}

// ✅ Good practice: Defer business logic to the UI context
function init() {
    $app.onScanCompleted((e) => {
        // Send a copy of the event
        $store.set("scan-completed", $clone(e))
        e.next()
    })
    
    // Let the UI context "listen" to the event and execute business logic
    $ui.register((ctx) => {
        // Callback is triggered anytime 'set' is called on that key
        $store.watch<number>("scan-completed", (e) => {
            const now = new Date().toISOString().replaceall(".", "_")
            
            // Add the value to the history
            $storage.set("scan-duration-history."+now, e.duration)
            
            ctx.toast.info(`Scanning took ${e.duration/1000} seconds!`)
        })
    })
}
```

<figure><img src="https://266901462-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F7Mat9fLDAotSl6o8o4P3%2Fuploads%2FAumoGr35i3qgXCJ8EXOB%2Fimg-2025-04-28-11-15-31%402x.png?alt=media&#x26;token=8b30769c-37f7-4ecf-b3e4-fdce52aa969a" alt=""><figcaption></figcaption></figure>



<!-- Page: https://seanime.gitbook.io/seanime-extensions/plugins/example -->


# Example

Let's create a plugin that allows users to change banner images of anime that are in the AniList collection. We'll use both the UI and hooks APIs.

**my-plugin.ts**


```typescript
/// <reference path="./plugin.d.ts" />
/// <reference path="./hooks.d.ts" />

function init() {
    $ui.register((ctx) => {
        // Create the tray icon
        const tray = ctx.newTray({
            tooltipText: "Anime banner image",
            iconUrl: "https://seanime.rahim.app/logo_2.png",
            withContent: true,
        })
        
        // Keep track of the current media ID
        const currentMediaId = ctx.state(0)

        // Create a field ref for the URL input
        const inputRef = ctx.fieldRef()

        // When the plugin loads, fetch the current screen and set the badge to 0
        ctx.screen.loadCurrent() // Triggers onNavigate
        tray.updateBadge({ number: 0 })
        // Also fetch current screen when tray is open
        tray.onOpen(() => {
            ctx.screen.loadCurrent()
        })

        // Updates the field's value and badge based on the current anime page
        function updateState() {
            // Reset the badge and input if the user currently isn't on an anime page
            if (!currentMediaId.get()) {
                inputRef.setValue("")
                tray.updateBadge({ number: 0 })
            }
            // Get the stored banner image URL for this anime
            const url = $storage.get<string>("bannerImages." + currentMediaId.get())
            if (url) {
                // If there's a URL, set the value of the input 
                inputRef.setValue(url)
                // Add a badge
                tray.updateBadge({ number: 1, intent: "info" })
            } else {
                inputRef.setValue("")
                tray.updateBadge({ number: 0 })
            }
        }
        
        // Run the function when the plugin loads
        updateState()

        // Update currentMediaId when the user navigates
        ctx.screen.onNavigate((e) => {
            // If the user navigates to an anime page
            if (e.pathname === "/entry" && !!e.searchParams.id) {
                // Get the ID from the URL
                const id = parseInt(e.searchParams.id)
                currentMediaId.set(id)
            } else {
                currentMediaId.set(0)
            }
        })

        // This effect will update the state each time currentMediaId changes
        ctx.effect(() => {
            updateState()
        }, [currentMediaId])

        // Create a handler to store the custom banner image URL
        ctx.registerEventHandler("save", () => {
            if (!!inputRef.current) {
                $storage.set(`bannerImages.${currentMediaId.get()}`, inputRef.current)
            } else {
                $storage.remove(`bannerImages.${currentMediaId.get()}`)
            }
            ctx.toast.success("Banner image saved")
            updateState() // Update the state
            
            // Updates the data on the client
            // This is better than calling ctx.screen.reload()
            $anilist.refreshAnimeCollection()
        });
        
        // Tray content
        tray.render(() => {
            return tray.stack([
                currentMediaId.get() === 0 
                    ? tray.text("Open an anime") 
                    : tray.stack([
                        tray.text(`Current media ID: ${currentMediaId.get()}`),
                        tray.input({ fieldRef: inputRef }),
                        tray.button({ label: "Save", onClick: "save" }),
                    ])
            ])
        })
    })

    // Register hook handlers to listen and modify the anime collection.
    
    // Triggers the app fetches the user's AniList anime collection
    $app.onGetAnimeCollection((e) => {
        const bannerImages = $storage.get<Record<string, string | undefined>>('bannerImages');
        if (!bannerImages) {
            e.next()
            return
        }
        if (!e.animeCollection?.mediaListCollection?.lists?.length) {
            e.next()
            return
        }
        
        for (let i = 0; i < e.animeCollection!.mediaListCollection!.lists!.length; i++) {
            for (let j = 0; j < e.animeCollection!.mediaListCollection!.lists![i]!.entries!.length; j++) {
                const mediaId = e.animeCollection!.mediaListCollection!.lists![i]!.entries![j]!.media!.id
                const bannerImage = bannerImages[mediaId.toString()]
                if (!!bannerImage) {
                    e.animeCollection!.mediaListCollection!.lists![i]!.entries![j]!.media!.bannerImage = bannerImage
                }
            }
        }

        e.next()
    })

    // Same as onGetAnimeCollection but also includes custom lists.
    $app.onGetRawAnimeCollection((e) => {
        const bannerImages = $storage.get<Record<string, string | undefined>>('bannerImages');
        if (!bannerImages) {
            e.next()
            return
        }
        if (!e.animeCollection?.mediaListCollection?.lists?.length) {
            e.next()
            return
        }
        
        for (let i = 0; i < e.animeCollection!.mediaListCollection!.lists!.length; i++) {
            for (let j = 0; j < e.animeCollection!.mediaListCollection!.lists![i]!.entries!.length; j++) {
                const mediaId = e.animeCollection!.mediaListCollection!.lists![i]!.entries![j]!.media!.id
                const bannerImage = bannerImages[mediaId.toString()]
                if (!!bannerImage) {
                    e.animeCollection!.mediaListCollection!.lists![i]!.entries![j]!.media!.bannerImage = bannerImage
                }
            }
        }

        e.next()
    })
}

```





<!-- Page: https://seanime.gitbook.io/seanime-extensions/frequently-asked/feature-requests -->


# Feature requests

> **Info**
>
> Feature requests on GitHub pertain to features that will be integrated in the source code **only**.


### Why was this feature request closed?

As of `v2.8.0` , Seanime supports [plugins](https://seanime.gitbook.io/seanime-extensions/plugins "mention"), which can be developed entirely in JavaScript.

A feature request will be closed as not planned with the label `status: plugin-suitable` if:

* The feature can be reasonably added via plugin
* The feature will not benefit a majority of users
* The feature is mostly subjective or cosmetic (e.g. removing elements, changing layout, etc.)

This is done to:

* Reduce development time and update cycles
* Avoid bloat by offloading noncritical features
* Improve contribution

### What if I can't develop a plugin?

Join the Discord server and make a request in the `#extension-proposals` channel, someone might make it for you.



<!-- Page: https://seanime.gitbook.io/seanime-extensions/seanime -->


# Seanime

- [Getting started](/seanime-extensions/seanime/getting-started.md)
- [Core APIs](/seanime-extensions/seanime/core-apis.md): These APIs are shared between all types of extension: content providers and plugins.
- [Changelog](/seanime-extensions/seanime/changelog.md): Important changes are logged here.
