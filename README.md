![](docs/screenshot.png)

# DiscordChatExporter-frontend
View your JSON [DiscordChatExporter](https://github.com/Tyrrrz/DiscordChatExporter) exports as if you were using Discord interface

## Why JSON exports are better than HTML exports?
- better-suited for long-term preservation
- contain more information than HTML exports
- readable by third-party applications like this one or [chat-analytics](https://github.com/mlomb/chat-analytics), not just browsers
- smaller in size

This project aims to provide you with better experience than what you get by just opening HTML exports in your browser. In one place and with many more features, like search, message deduplication, etc. And it will get even better in the future :).

## Features
- View JSON exports in Discord like interface
- Message deduplication - merge multiple JSON exports and view them as one
- Optimized infinite scroll - channels are loaded almost instantly
- Forums and threads support (jump to thread and back)
- Search with filters and autocomplete
- Load assets locally or from Discord servers
- Private messages or guild exports are supported
- Discord Markdown rendering
- Right click message and select "Open in discord" to view message in Discord
- Windows, Linux and Mac support (including M1 Macs)


### System requirements (per guild)
- You will need ~1 GB of RAM to process 100k messages. So if your guild export contains 1 million messages, you will need ~10 GB of RAM.
- The viewer can handle at least ~2-4 million messages (4 GB of ram) in the browser
- Chromium based browsers are recommended (Chrome, Edge, Opera, Brave, Vivaldi, etc.)

Note: Discord servers are known internally as guilds

Planned feature:
- Refactor to client-server architecture is planned to handle larger exports than 4 million messages per guild. And with added bonus of being able to run the viewer on your own server and view your exports from anywhere. But it will take some time to implement.

## Quick start (Windows)
Using prebuilt binaries is the easiest way to use this tool on Windows.
1. Download the latest release from [releases page](https://github.com/slatinsky/DiscordChatExporter-frontend/releases)
2. Extract the archive
3. (optional) Move your [DiscordChatExporter](https://github.com/Tyrrrz/DiscordChatExporter) exports to `/exports/` folder ([supported exports](#supported-exports)). Folder structure inside this folder doesn't matter, script will find everything it needs. **Skip this step if you** don't have any exports yet and **want to try out the viewer** - example exports are included in the release.
4. Run `START_VIEWER.bat` - DiscordChatExporter-frontend will open in your default browser

NOTE: command prompt window ("black rectangle") named "nginx" will open. It provides local web server for the viewer. After you are done with using the viewer, you can close it.

### Beta builds (Windows)
If you want to try out the latest features, you can use [beta builds](https://github.com/slatinsky/DiscordChatExporter-frontend/actions/workflows/windows-build.yml). They are automatically built from the latest commit on `master` branch or from pull requests. They are quite stable, but no guarantees :)

## Docker version (Linux+Mac)
You need docker and git installed. Tested on non-snap version of docker on Ubuntu 22.04. @levithomason [tested it](https://github.com/slatinsky/DiscordChatExporter-frontend/issues/5) on M1 MacBook (Apple Silicon)
1. Build image
```bash
git clone https://github.com/slatinsky/DiscordChatExporter-frontend
cd DiscordChatExporter-frontend
docker build -t dcef .
```
2. Navigate to folder with your exports
```bash
cd [path to your exports]
```

3. Run container
```bash
docker run --volume "$(pwd):/dcef/exports" --volume dcef_cache:/dcef/cache --rm -p 21011:21011 -it dcef
```

4. Open `http://127.0.0.1:21011/` in your browser

<details><summary>Debugging containers</summary>
<p>

To debug running container, run `docker exec -it $(docker ps | grep 'dcef' | awk '{ print $1 }') sh`. This will open a shell inside the container.

</p>
</details>

## Upgrade guide
Want to upgrade from previous version? Follow these steps:

1. Download the latest release from [releases page](https://github.com/slatinsky/DiscordChatExporter-frontend/releases)
2. Extract the archive
3. Move your `/exports` folder to the new release folder.
4. Delete old release folder

Info: since release 1.10.0, exports folder was changed from `/static/input/` to `/exports/`.

<a name="supported-exports"></a>
## Which exports are supported?

Exports are done by [DiscordChatExporter](https://github.com/Tyrrrz/DiscordChatExporter).

- JSON export format (`--format Json`) is required
- Including assets (`--media --reuse-media`) is highly recommended, but not required
- Don't forget to export **threads and forums**, because they **are not included in the main export** (see examples below for a guide on how to export them)

<details><summary>CLI examples</summary>
<p>

Export all accessible channels from guild:
```
DiscordChatExporter.Cli.exe exportguild --token DISCORD_TOKEN -g GUILD_ID --media --reuse-media --format Json --output OUTPUT_FOLDER_PATH
```
Export all dms (sadly, exporting dms can't be done without selfboting):
```
DiscordChatExporter.Cli.exe exportdm --token DISCORD_TOKEN --media --reuse-media --format Json --output OUTPUT_FOLDER_PATH
```
Export channel/thread/forum posts:
```
DiscordChatExporter.Cli export --token DISCORD_TOKEN  --media --reuse-media --output OUTPUT_FOLDER_PATH --format Json --channel CHANNEL_OR_THREAD_ID_OR_FORUM_POST_ID_1 CHANNEL_OR_THREAD_ID_OR_FORUM_POST_ID_2 CHANNEL_OR_THREAD_ID_OR_FORUM_POST_ID_3 CHANNEL_OR_THREAD_ID_OR_FORUM_POST_ID_4
```

Viewer also supports html export with assets + json export without assets - but it's not recommended, because most embeds will be missing.
</p>
</details>

<details><summary>Helper script to export archived threads in a channel</summary>
<p>

Viewing threads is supported by this viewer, but exporting them manually is time consuming. You can use this helper script to generate command to download all archived threads in a channel automatically:

### Steps
1. Open discord in browser (browser needs to be Chromium based - Chrome, Edge, Opera, Brave, Vivaldi, etc., not working in Firefox)
2. Navigate to channel with threads. Do not open thread list (if you opened it, refresh the page)
3. press F12 and paste this script to the console:

```js
len = 0
ids = []
previouseScrollTop = 0

// interceptor  https://stackoverflow.com/a/66564476
if (window.oldXHROpen === undefined) {
    window.oldXHROpen = window.XMLHttpRequest.prototype.open;
    window.XMLHttpRequest.prototype.open = function(method, url, async, user, password) {
    this.addEventListener('load', function() {
        try {
            var json = JSON.parse(this.responseText);
            if (json.hasOwnProperty('threads')) {
                // get ids
                for (const thread of json.threads) {
                    ids.push(thread.id)
                }
            }
        }
        catch (e) {
            console.log(e)
        }
    });
    return window.oldXHROpen.apply(this, arguments);
    }
}
else {
    console.log('OK, interceptor already exists')
}


function getSelector(name) {
    if (name === 'thread-icon')
        return document.querySelectorAll('div[class*="toolbar-"] > div[aria-label="Threads"]')
    if (name === 'scroller-base')
        return document.querySelectorAll('div[id*="popout_"] > div > div > div[class*="scrollerBase-"]')
}

function clickThreadsIcon() {
    if (getSelector('scroller-base').length > 0) {
        console.log('OK, found scroller base (1)')
        mainFunc()
    }
    else if (getSelector('scroller-base').length == 0 && getSelector('thread-icon').length > 0) {
        getSelector('thread-icon')[0].click()
        console.log('OK, clicked Threads Icon')
        setTimeout(() => {
            if (getSelector('scroller-base').length == 0) {
                throw new Error('ERROR, could not find scroller-base')
            }
            else {
                console.log('OK, found scroller base (2)')
                mainFunc()
            }
        }, 1000)
    }
    else {
        throw new Error('ERROR, could not find threads icon')
    }
}
clickThreadsIcon()

function scrollToPosition(offset) {
    scrollDiv = getSelector('scroller-base')[0]
    scrollDiv.scroll(0, offset)
}

function captureIds() {
    ids = [...new Set(ids)]
    if (ids.length > len) {
        len = ids.length
        console.log('Found', len, 'IDs')
    }
}

function printIds() {
    console.log('DiscordChatExporter.Cli.exe export --token TOKEN --output "exports/threads" --format Json --media --reuse-media --channel',ids.join(' '))
}

function mainFunc() {
    scrollToPosition(0)
    interval = setInterval(() => {
        scrollToPosition(scrollDiv.scrollTop + window.innerHeight / 3)
        setTimeout(() => {
            captureIds()
            if (previouseScrollTop === scrollDiv.scrollTop) {
                clearInterval(interval)
                printIds()
            }
            previouseScrollTop = scrollDiv.scrollTop
        }, 500)
    }, 742)
}
```

4. script will scroll thread list. At the the end, it will print command to the console, which allows you to export all archived threads in the channel
5. edit command printed in the console (`--format`, `--output` and `--token`) and export with CLI version of DiscordChatExporter
</p>
</details>

<details><summary>Helper script to export forum posts in a channel</summary>
<p>

Viewing forums is supported by this viewer, but exporting them manually is time consuming. You can use this helper script to generate command to download all forum posts in a forum channel automatically:

### Steps
1. Open discord in browser (browser needs to be Chromium based - Chrome, Edge, Opera, Brave, Vivaldi, etc., not working in Firefox)
2. Navigate to channel with forum post list
3. press F12 and paste this script to the console:

```js
len = 0
ids = []
previouseScrollTop = 0

function scrollToPosition(offset) {
    scrollDiv = document.querySelector('div[class*="chat-"] > div > div > div[class*="scrollerBase-"]')
    scrollDiv.scroll(0, offset)
}

function captureIds() {
    document.querySelectorAll('div[data-item-id]').forEach((e) => ids.push(e.dataset.itemId))
    ids = [...new Set(ids)]
    if (ids.length > len) {
        len = ids.length
        console.log('Found', len, 'IDs')
    }
}

function printIds() {
    console.log('DiscordChatExporter.Cli.exe export --token TOKEN --output "exports/forums" --format Json --media --reuse-media --channel',ids.join(' '))
}

scrollToPosition(0)
interval = setInterval(() => {
    scrollToPosition(scrollDiv.scrollTop + window.innerHeight / 3)
    setTimeout(() => {
        captureIds()
        if (previouseScrollTop === scrollDiv.scrollTop) {
            clearInterval(interval)
            printIds()
        }
        previouseScrollTop = scrollDiv.scrollTop
    }, 1000)
}, 1542)
```

4. script will scroll the page. At the the end, it will print command to the console, which allows you to export all forum posts in the forum channel
5. edit command printed in the console (`--format`, `--output` and `--token`) and export with CLI version of DiscordChatExporter
</p>
</details>

### Settings for GUI version of DiscordChatExporter
Make sure that export format is set to `JSON`. Optionally, you can enable `Download assets` option to download images, videos and other types of assets.

![](docs/dce-export-more.png)

**NOTE**: You can't export threads and forums using GUI version of DCE. You need to use CLI version for that.

## Development
You don't need to follow development steps if you don't intend to modify the code.

<details><summary>Show development steps</summary>
<p>

Then make sure you use node 16.16.0 and have nodemon installed globally (used for python3 hot reloading)
```bash
nvm use 16.16.0
npm install -g nodemon
```

Then install python3 dependencies
```bash
cd backend/preprocess
py -m pip install imagesize
cd ..
```

Now you just need to run the development helper script to start all needed processes
```bash
DEV.bat
```

If everything was done correctly, DiscordChatExporter-frontend will open in your browser with working hot reloading.

(There is no dev version for Linux. You have to use docker version)

</p>
</details>


<details><summary>Show steps to build release binaries from source (Windows)</summary>
<p>

## Requirements
- Node.js 16
- Python 3.9+
- pyinstaller (installled globally)
```
py -m pip install pyinstaller
```

## Steps
1. Clone this repository
```bash
git clone https://github.com/slatinsky/DiscordChatExporter-frontend
```
2. Install dependencies
```bash
npm install
cd server
npm install
cd ..
```
3. Make sure you have Python3.9+ Node.js 16 and pyinstaller installed:
```
>py --version
Python 3.10.2
```
```
>node --version
v16.14.2
```
```
>pyinstaller --version
5.5
```

4. Kill `npm run dev` if it is running

5. Run the build script
```bash
BUILD_RELEASE.bat
```

6. Release binaries will be in `/release/` folder

</p>
</details>

<details><summary>Tested on</summary>
<p>

```
>winver
Windows 10, 21H2
Os build: 19044.1766

>py --version
Python 3.10.2

>node --version
v16.14.2

>pyinstaller --version
5.5

nginx/Windows-1.23.2

DiscordChatExporter version:
v2.36.1

Processor:
AMD Ryzen™ 7 5800H

400k messages with 18GB of media files
```


But binary release should work on any Windows 10 / Windows 11 x64 computer.

Docker release should work on Linux x64 and Mac M1 (arm64) computers.

</p>
</details>

## Thanks
- [Tyrrrz/DiscordChatExporter](https://github.com/Tyrrrz/DiscordChatExporter) - for a great tool. This project is based on some parts of it's code.
- Discord - for a great chat app
- [brussell98/discord-markdown](https://github.com/brussell98/discord-markdown) - for discord markdown rendering

And for other technologies used in this project - sveltekit, docker, nodejs, nvm, pyinstaller, nginx.

## License
GNU GENERAL PUBLIC LICENSE

This product contains software provided by NGINX and its contributors.

DiscordChatExporter-frontend is not affiliated with Discord. Discord is a registered trademark of Discord Inc.

## Contributing
Feel free to open issues and pull requests.
### Short guide, how to contribute
- Fork the repository
- Create a new branch
- Implement your changes
- Commit and push the changes
- Create a pull request

If you find this project useful, please consider starring it here on GitHub :)