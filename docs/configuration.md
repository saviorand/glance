# Configuration

- [Intro](#intro)
- [Preconfigured page](#preconfigured-page)
- [Server](#server)
- [Theme](#theme)
  - [Themes](#themes)
- [Pages & Columns](#pages--columns)
- [Widgets](#widgets)
  - [RSS](#rss)
  - [Videos](#videos)
  - [Hacker News](#hacker-news)
  - [Reddit](#reddit)
  - [Weather](#weather)
  - [Monitor](#monitor)
  - [Releases](#releases)
  - [Bookmarks](#bookmarks)
  - [Calendar](#calendar)
  - [Stocks](#stocks)
  - [Twitch Channels](#twitch-channels)
  - [Twitch Top Games](#twitch-top-games)
  - [iframe](#iframe)

## Intro
Configuration is done via a single YAML file and a server restart is required in order for any changes to take effect. Trying to start the server with an invalid config file will result in an error.

## Preconfigured page
If you don't want to spend time reading through all the available configuration options and just want something to get you going quickly you can use the following `glance.yml` and make changes as you see fit:

```yaml
pages:
  - name: Home
    columns:
      - size: small
        widgets:
          - type: calendar

          - type: rss
            limit: 10
            collapse-after: 3
            cache: 3h
            feeds:
              - url: https://ciechanow.ski/atom.xml
              - url: https://www.joshwcomeau.com/rss.xml
                title: Josh Comeau
              - url: https://samwho.dev/rss.xml
              - url: https://awesomekling.github.io/feed.xml
              - url: https://ishadeed.com/feed.xml
                title: Ahmad Shadeed

          - type: twitch-channels
            channels:
              - theprimeagen
              - cohhcarnage
              - christitustech
              - blurbs
              - asmongold
              - jembawls

      - size: full
        widgets:
          - type: hacker-news

          - type: videos
            channels:
              - UCR-DXc1voovS8nhAvccRZhg # Jeff Geerling
              - UCv6J_jJa8GJqFwQNgNrMuww # ServeTheHome
              - UCOk-gHyjcWZNj3Br4oxwh0A # Techno Tim

          - type: reddit
            subreddit: selfhosted

      - size: small
        widgets:
          - type: weather
            location: London, United Kingdom

          - type: stocks
            stocks:
              - symbol: SPY
                name: S&P 500
              - symbol: BTC-USD
                name: Bitcoin
              - symbol: NVDA
                name: NVIDIA
              - symbol: AAPL
                name: Apple
              - symbol: MSFT
                name: Microsoft
              - symbol: GOOGL
                name: Google
              - symbol: AMD
                name: AMD
              - symbol: RDDT
                name: Reddit
```

This will give you a page that looks like the following:

![](images/preconfigured-page-preview.png)

Configure the widgets, add more of them, add extra pages, etc. Make it your own!

## Server
Server configuration is done through a top level `server` property. Example:

```yaml
server:
  port: 8080
  assets-path: /home/user/glance-assets
```

### Properties

| Name | Type | Required | Default |
| ---- | ---- | -------- | ------- |
| host | string | no |  |
| port | number | no | 8080 |
| assets-path | string | no |  |

#### `host`
The address which the server will listen on. Setting it to `localhost` means that only the machine that the server is running on will be able to access the dashboard. By default it will listen on all interfaces.

#### `port`
A number between 1 and 65,535, so long as that port isn't already used by anything else.

#### `assets-path`
The path to a directory that will be served by the server under the `/assets/` path. This is handy for widgets like the Monitor where you have to specify an icon URL and you want to self host all the icons rather than pointing to an external source.

> [!IMPORTANT]
>
> When installing through docker the path will point to the files inside the container. Don't forget to mount your assets path to the same path inside the container.
> Example:
>
> If your assets are in:
> ```
> /home/user/glance-assets
> ```
>
> You should mount:
> ```
> /home/user/glance-assets:/app/assets
> ```
>
> And your config should contain:
> ```
> assets-path: /app/assets
> ```

##### Examples

Say you have a directory `glance-assets` with a file `gitea-icon.png` in it and you specify your assets path like:

```yaml
assets-path: /home/user/glance-assets
```

To be able to point to an asset from your assets path, use the the `/assets/` path like such:

```yaml
icon: /assets/gitea-icon.png
```

## Theme
Theming is done through a top level `theme` property. Values for the colors are in [HSL](https://giggster.com/guide/basics/hue-saturation-lightness/) (hue, saturation, lightness) format. You can use a color picker [like this one](https://hslpicker.com/) to convert colors from other formats to HSL. The values are separated by a space and `%` is not required for any of the numbers.

Example:

```yaml
theme:
  background-color: 100 20 10
  primary-color: 40 90 40
  contrast-multiplier: 1.1
```

### Themes
If you don't want to spend time configuring your own theme, there are [several available themes](themes.md) which you can simply copy the values for.

### Properties
| Name | Type | Required | Default |
| ---- | ---- | -------- | ------- |
| light | bool | no | false |
| background-color | HSL | no | 240 8 9 |
| primary-color | HSL | no | 43 50 70 |
| positive-color | HSL | no | same as `primary-color` |
| negative-color | HSL | no | 0 70 70 |
| contrast-multiplier | number | no | 1 |
| text-saturation-multiplier | number | no | 1 |
| custom-css-file | string | no | |

#### `light`
Whether the scheme is light or dark. This does not change the background color, it inverts the text colors so that they look appropriately on a light background.

#### `background-color`
Color of the page and widgets.

#### `primary-color`
Color used across the page, largely to indicate unvisited links.

#### `positive-color`
Used to indicate that something is positive, such as stock price being up, twitch channel being live or a monitored site being online. If not set, the value of `primary-color` will be used.

#### `negative-color`
Oppposite of `positive-color`.

#### `contrast-multiplier`
Used to increase or decrease the contrast (in other words visibility) of the text. A value of `1.3` means that the text will be 30% lighter/darker depending on the scheme. Use this if you think that some of the text on the page is too dark and hard to read. Example:

![difference between 1 and 1.3 contrast](images/contrast-multiplier-example.png)

#### `text-saturation-multiplier`
Used to increase or decrease the saturation of text, useful when using a custom background color with a high amount of saturation and needing the text to have a more neutral color. `0.5` means that the saturation will be 50% lower and `1.5` means that it'll be 50% higher.

#### `custom-css-file`
Path to a custom CSS file, either external or one from within the server configured assets path. Example:

```yaml
theme:
  custom-css-file: /assets/my-style.css
```

> [!TIP]
>
> Because Glance uses a lot of utility classes it might be difficult to target some elements. To make it easier to style specific widgets, each widget has a `widget-type-{name}` class, so for example if you wanted to make the links inside just the RSS widget bigger you could use the following selector:
>
> ```css
> .widget-type-rss a {
>     font-size: 1.5rem;
> }


## Pages & Columns
![illustration of pages and columns](images/pages-and-columns-illustration.png)

Using pages and columns is how widgets are organized. Each page contains up to 3 columns and each column can have any number of widgets.

### Pages
Pages are defined through a top level `pages` property. The page defined first becomes the home page and all pages get automatically added to the navigation bar in the order that they were defined. Example:

```yaml
pages:
  - name: Home
    columns: ...

  - name: Videos
    columns: ...

  - name: Homelab
    columns: ...
```

### Properties
| Name | Type | Required |
| ---- | ---- | -------- |
| title | string | yes |
| slug | string | no |
| columns | array | yes |

#### `title`
The name of the page which gets shown in the navigation bar.

#### `slug`
The URL friendly version of the title which is used to access the page. For example if the title of the page is "RSS Feeds" you can make the page accessible via `localhost:8080/feeds` by setting the slug to `feeds`. If not defined, it will automatically be generated from the title.

### Columns
Columns are defined for each page using a `columns` property. There are two types of columns - `full` and `small`, which refers to their width. A small column takes up a fixed amount of width (300px) and a full column takes up the all of the remaining width. You can have up to 3 columns per page and you must have either 1 or 2 full columns. Example:

```yaml
pages:
  - name: Home
    columns:
      - size: small
        widgets: ...
      - size: full
        widgets: ...
      - size: small
        widgets: ...
```

### Properties
| Name | Type | Required |
| ---- | ---- | -------- |
| size | string | yes |
| widgets | array | no |

Here are some of the possible column configurations:

![column configuration small-full-small](images/column-configuration-1.png)

```yaml
columns:
  - size: small
    widgets: ...
  - size: full
    widgets: ...
  - size: small
    widgets: ...
```

![column configuration small-full-small](images/column-configuration-2.png)

```yaml
columns:
  - size: full
    widgets: ...
  - size: small
    widgets: ...
```

![column configuration small-full-small](images/column-configuration-3.png)

```yaml
columns:
  - size: full
    widgets: ...
  - size: full
    widgets: ...
```

## Widgets
Widgets are defined for each column using a `widgets` property. Example:

```yaml
pages:
  - name: Home
    columns:
      - size: small
        widgets:
          - type: weather
            location: London, United Kingdom
```

> [!NOTE]
>
> Currently not all widgets are designed to fit every column size, however some widgets offer different "styles" that help alleviate this limitation.

### Shared Properties
| Name | Type | Required |
| ---- | ---- | -------- |
| type | string | yes |
| title | string | no |
| cache | string | no |

#### `type`
Used to specify the widget.

#### `title`
The title of the widget. If left blank it will be defined by the widget.

#### `cache`
How long to keep the fetched data in memory. The value is a string and must be a number followed by one of s, m, h, d. Examples:

```yaml
cache: 30s # 30 seconds
cache: 5m  # 5 minutes
cache: 2h  # 2 hours
cache: 1d  # 1 day
```

> [!NOTE]
>
> Not all widgets can have their cache duration modified. The calendar and weather widgets update on the hour and this cannot be changed.

### RSS
Display a list of articles from multiple RSS feeds.

Example:

```yaml
- type: rss
  title: News
  style: horizontal-cards
  feeds:
    - url: https://feeds.bloomberg.com/markets/news.rss
      title: Bloomberg
    - url: https://moxie.foxbusiness.com/google-publisher/markets.xml
      title: Fox Business
    - url: https://moxie.foxbusiness.com/google-publisher/technology.xml
      title: Fox Business
```

#### Properties
| Name | Type | Required | Default |
| ---- | ---- | -------- | ------- |
| style | string | no | vertical-list |
| feeds | array | yes |
| limit | integer | no | 25 |
| collapse-after | integer | no | 5 |

##### `style`
Used to change the appearance of the widget. Possible values are `vertical-list` and `horizontal-cards` where the former is intended to be used within a small column and the latter a full column. Below are previews of each style.

`vertical-list`

![preview of vertical-list style for RSS widget](images/rss-feed-vertical-list-preview.png)

`horizontal-cards`

![preview of horizontal-cards style for RSS widget](images/rss-feed-horizontal-cards-preview.png)

##### `feeds`
An array of RSS/atom feeds. The title can optionally be changed.

###### Properties for each feed
| Name | Type | Required | Default |
| ---- | ---- | -------- | ------- |
| url | string | yes | |
| title | string | no | the title provided by the feed |

##### `limit`
The maximum number of articles to show.

##### `collapse-after`
How many articles are visible before the "SHOW MORE" button appears. Set to `-1` to never collapse.

### Videos
Display a list of the latest videos from specific YouTube channels.

Example:

```yaml
- type: videos
  channels:
    - UCXuqSBlHAE6Xw-yeJA0Tunw
    - UCBJycsmduvYEL83R_U4JriQ
    - UCHnyfMqiRRG1u-2MsSQLbXA
```

Preview:
![](images/videos-widget-preview.png)

#### Properties
| Name | Type | Required | Default |
| ---- | ---- | -------- | ------- |
| channels | array | yes | |
| limit | integer | no | 25 |

##### `channels`
A list of channel IDs. One way of getting the ID of a channel is going to the channel's page and clicking on its description:

![](images/videos-channel-description-example.png)

Then scroll down and click on "Share channel", then "Copy channel ID":

![](images/videos-copy-channel-id-example.png)

##### `limit`
The maximum number of videos to show.

### Hacker News
Display a list of posts from [Hacker News](https://news.ycombinator.com/).

Example:

```yaml
- type: hacker-news
  limit: 15
  collapse-after: 5
```

Preview:
![](images/hacker-news-widget-preview.png)

#### Properties
| Name | Type | Required | Default |
| ---- | ---- | -------- | ------- |
| limit | integer | no | 15 |
| collapse-after | integer | no | 5 |

### Reddit
Display a list of posts from a specific subreddit.

> [!WARNING]
>
> Reddit does not allow unauthorized API access from VPS IPs, if you're hosting Glance on a VPS you will get a 403 response. As a workaround you can route the traffic from Glance through a VPN.

Example:

```yaml
- type: reddit
  subreddit: technology
```

#### Properties
| Name | Type | Required | Default |
| ---- | ---- | -------- | ------- |
| subreddit | string | yes |  |
| style | string | no | vertical-list |
| limit | integer | no | 15 |
| collapse-after | integer | no | 5 |

##### `subreddit`
The subreddit for which to fetch the posts from.

##### `style`
Used to change the appearance of the widget. Possible values are `vertical-list`, `horizontal-cards` and `vertical-cards`. The first two were designed for full columns and the last for small columns.

`vertical-list`

![](images/reddit-widget-preview.png)

`horizontal-cards`

![](images/reddit-widget-horizontal-cards-preview.png)

`vertical-cards`

![](images/reddit-widget-vertical-cards-preview.png)

##### `limit`
The maximum number of posts to show.

##### `collapse-after`
How many posts are visible before the "SHOW MORE" button appears. Set to `-1` to never collapse. Not available when using the `vertical-cards` and `horizontal-cards` styles.

### Weather
Display weather information for a specific location. The data is provided by https://open-meteo.com/.

Example:

```yaml
- type: weather
  units: metric
  location: London, United Kingdom
```

Preview:

![](images/weather-widget-preview.png)

Each bar represents a 2 hour interval. The yellow background represents sunrise and sunset. The blue dots represent the times of the day where there is a high chance for precipitation. You can hover over the bars to view the exact temperature for that time.

#### Properties

| Name | Type | Required | Default |
| ---- | ---- | -------- | ------- |
| location | string | yes |  |
| units | string | no | metric |
| hide-location | boolean | no | false |

##### `location`
The name of the city and country to fetch weather information for. Attempting to launch the applcation with an invalid location will result in an error. You can use the [gecoding API page](https://open-meteo.com/en/docs/geocoding-api) to search for your specific location. Glance will use the first result from the list if there are multiple.

##### `units`
Whether to show the temperature in celsius or fahrenheit, possible values are `metric` or `imperial`.

##### `hide-location`
Optionally don't display the location name on the widget.

### Monitor
Display a list of sites and whether they are reachable (online) or not. This is determined by sending a HEAD request to the specified URL, if the response is 200 then the site is OK. The time it took to receive a response is also shown in milliseconds.

Example:

```yaml
- type: monitor
  cache: 1m
  title: Services
  sites:
    - title: Jellyfin
      url: https://jellyfin.yourdomain.com
      icon: /assets/jellyfin-logo.png
    - title: Gitea
      url: https://gitea.yourdomain.com
      icon: /assets/gitea-logo.png
    - title: Immich
      url: https://immich.yourdomain.com
      icon: /assets/immich-logo.png
    - title: AdGuard Home
      url: https://adguard.yourdomain.com
      icon: /assets/adguard-logo.png
    - title: Vaultwarden
      url: https://vault.yourdomain.com
      icon: /assets/vaultwarden-logo.png

```

Preview:

![](images/monitor-widget-preview.png)

You can hover over the "ERROR" text to view more information.

#### Properties

| Name | Type | Required |
| ---- | ---- | -------- |
| sites | array | yes |  |

##### `sites`

Properties for each site:

| Name | Type | Required |
| ---- | ---- | -------- |
| title | string | yes |
| url | string | yes |
| icon | string | no |

`title`

The title used to indicate the site.

`url`

The URL which will be requested and its response will determine the status of the site.

`icon`

Optional URL to an image which will be used as the icon for the site. Can be an external URL or internal via [server configured assets](#assets-path).

### Releases
Display a list of releases for specific repositories on Github. Draft releases and prereleases will not be shown.

Example:

```yaml
- type: releases
  repositories:
    - immich-app/immich
    - go-gitea/gitea
    - dani-garcia/vaultwarden
    - jellyfin/jellyfin
```

Preview:

![](images/releases-widget-preview.png)

#### Properties

| Name | Type | Required | Default |
| ---- | ---- | -------- | ------- |
| repositories | array | yes |  |
| token | string | no | |
| limit | integer | no | 10 |
| collapse-after | integer | no | 5 |

##### `repositories`
A list of repositores for which to fetch the latest release for. Only the name/repo is required, not the full URL.

##### `token`
Without authentication Github allows for up to 60 requests per hour. You can easily exceed this limit and start seeing errors if you're tracking lots of repositories or your cache time is low. To circumvent this you can [create a read only token from your Github account](https://github.com/settings/personal-access-tokens/new) and provide it here.

You can also specify the value for this token through an ENV variable using the syntax `${GITHUB_TOKEN}` where `GITHUB_TOKEN` is the name of the variable that holds the token. If you've installed Glance through docker you can specify the token in your docker-compose:

```yaml
services:
  glance:
    image: glanceapp/glance
    environment:
      - GITHUB_TOKEN: <your token>
```

and then use it in your `glance.yml` like this:

```yaml
- type: releases
  token: ${GITHUB_TOKEN}
  repositories: ...
```

This way you can safely check your `glance.yml` in version control without exposing the token.

##### `limit`
The maximum number of releases to show.

#### `collapse-after`
How many releases are visible before the "SHOW MORE" button appears. Set to `-1` to never collapse.

### Bookmarks
Display a list of links which can be grouped.

Example:

```yaml
- type: bookmarks
  groups:
    - links:
        - title: Gmail
          url: https://mail.google.com/mail/u/0/
        - title: Amazon
          url: https://www.amazon.com/
        - title: Github
          url: https://github.com/
        - title: Wikipedia
          url: https://en.wikipedia.org/
    - title: Entertainment
      color: 10 70 50
      links:
        - title: Netflix
          url: https://www.netflix.com/
        - title: Disney+
          url: https://www.disneyplus.com/
        - title: YouTube
          url: https://www.youtube.com/
        - title: Prime Video
          url: https://www.primevideo.com/
    - title: Social
      color: 200 50 50
      links:
        - title: Reddit
          url: https://www.reddit.com/
        - title: Twitter
          url: https://twitter.com/
        - title: Instagram
          url: https://www.instagram.com/
```

Preview:

![](images/bookmarks-widget-preview.png)


#### Properties

| Name | Type | Required |
| ---- | ---- | -------- |
| groups | array | yes |

##### `groups`
An array of groups which can optionally have a title and a custom color.

###### Properties for each group
| Name | Type | Required | Default |
| ---- | ---- | -------- | ------- |
| title | string | no | |
| color | HSL | no | the primary theme color |
| links | array | yes | |

###### Properties for each link
| Name | Type | Required |
| ---- | ---- | -------- |
| title | string | yes |
| url | string | yes |

### Calendar
Display a calendar.

Example:

```yaml
- type: calendar
```

Preview:

![](images/calendar-widget-preview.png)

> [!NOTE]
>
> There is currently no customizability available for the calendar. Extra features will be added in the future.

### Stocks
Display a list of stocks, their current value, change for the day and a small 21d chart. Data is taken from Yahoo Finance.

Example:

```yaml
- type: stocks
  stocks:
    - symbol: SPY
      name: S&P 500
    - symbol: BTC-USD
      name: Bitcoin
    - symbol: NVDA
      name: NVIDIA
    - symbol: AAPL
      name: Apple
    - symbol: MSFT
      name: Microsoft
    - symbol: GOOGL
      name: Google
    - symbol: AMD
      name: AMD
    - symbol: RDDT
      name: Reddit
```

Preview:

![](images/stocks-widget-preview.png)

#### Properties

| Name | Type | Required |
| ---- | ---- | -------- |
| stocks | array | yes |

##### `stocks`
An array of stocks for which to display information about.

###### Properties for each stock
| Name | Type | Required |
| ---- | ---- | -------- |
| symbol | string | yes |
| name | string | no |

`symbol`

The symbol, as seen in Yahoo Finance.

`name`

The name that will be displayed under the symbol.

### Twitch Channels
Display a list of channels from Twitch.

Example:

```yaml
- type: twitch-channels
  channels:
    - jembawls
    - giantwaffle
    - asmongold
    - cohhcarnage
    - j_blow
    - xQc
```

Preview:

![](images/twitch-channels-widget-preview.png)

#### Properties
| Name | Type | Required | Default |
| ---- | ---- | -------- | ------- |
| channels | array | yes | |
| collapse-after | integer | no | 5 |

##### `channels`
A list of channels to display.

##### `collapse-after`
How many channels are visible before the "SHOW MORE" button appears. Set to `-1` to never collapse.

### Twitch top games
Display a list of games with the most viewers on Twitch.

Example:

```yaml
- type: twitch-top-games
  exclude:
    - just-chatting
    - pools-hot-tubs-and-beaches
    - music
    - art
    - asmr
```

Preview:

![](images/twitch-top-games-widget-preview.png)

#### Properties
| Name | Type | Required | Default |
| ---- | ---- | -------- | ------- |
| exclude | array | no | |
| limit | integer | no | 10 |
| collapse-after | integer | no | 5 |

##### `exclude`
A list of categories that will never be shown. You must provide the slug found by clicking on the category and looking at the URL:

```
https://www.twitch.tv/directory/category/grand-theft-auto-v
                                         ^^^^^^^^^^^^^^^^^^
```

##### `limit`
The maximum number of games to show.

##### `collapse-after`
How many games are visible before the "SHOW MORE" button appears. Set to `-1` to never collapse.

### iframe
Embed an iframe as a widget.

Example:

```yaml
- type: iframe
  source: <url>
  height: 400
```

#### Properties
| Name | Type | Required | Default |
| ---- | ---- | -------- | ------- |
| source | string | yes | |
| height | integer | no | 300 |

##### `source`
The source of the iframe.

##### `height`
The height of the iframe. The minimum allowed height is 50.
