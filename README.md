# Zibbit

Zibbit is a self-hosted, touch-first streaming player that you run on your own server. It gives you a real guide grid, your own channel groups and favorites, and one place to manage several sources: IPTV playlists and Xtream providers, HDHomeRun antenna tuners, and (experimental) Tablo 4th Gen. It works in any browser, and this repository has the Android app.

It carries no channels, playlists or logins. You bring your own sources and are responsible for having the right to use them.

There are two parts, and you need both:

1. **A Zibbit server** (a small Docker container you run at home or on a server).
2. **A way to watch**: any web browser, or the Android app below.

## 1. Run the server

The image is public, so no account or login is needed to pull it:

```
docker run -d --name zibbit --restart unless-stopped \
  -p 8080:8080 \
  -v /path/to/zibbit-config:/config \
  ghcr.io/jamiecrout/zibbit:latest
```

Use any folder in place of `/path/to/zibbit-config`. Your settings, sign-ins and favorites live there, so the container can be updated or recreated without losing them. The image runs on `amd64` and `arm64` (Raspberry Pi 4/5 included).

Then open `http://<server-address>:8080`. Sign in as **admin** with the temporary password the container prints in its log on first start (`docker logs zibbit`, or the log button in the Unraid Docker tab). You are asked to choose a new password straight away. Open **Settings > Admin settings** to add your own sources and the people who will watch.

Step-by-step guides, including Docker Compose and Unraid:

- [Setup](docs/SETUP.md): install, first sign-in, reaching it from other devices, updating
- [Using it](docs/USING.md): a tour of the screens for the people who watch
- [Configuration](docs/CONFIGURATION.md): roles, settings, environment variables, admin commands
- [Upgrading](docs/UPGRADING.md): moving from v1 or from 2.x
- [What's new](#whats-new)

## 2. Watch

**In a browser:** open the server address on any phone, tablet or computer. You can add the page to a phone's home screen.

**With the Android app:** [Zibbit.apk](Zibbit.apk) (Android 7.0 or newer).

SHA-256: `b45199131cdca157d5bef99c90946a6134fbbbde9debebf27352b58537415771`
Signing certificate SHA-256: `a7a9fd99926ce22ffa3d8325fcd254c0b907b3bc9069bb98de2fed6c39b3464a`

1. Download `Zibbit.apk` on the phone or tablet.
2. Open it. Android will ask you to allow installs from that app (your browser or file manager). Allow it for this install.
3. Open **Zibbit**. On first launch it asks for the address of your server, for example `zibbit.example.com` or `192.168.1.20:8080`, then saves it. From then on it opens the last server straight away. To switch servers, open Settings in the page and choose **Change server** (needs server 2.1 or newer); Back returns to where you were.

**Installing with Downloader:** if you use the Downloader app, enter the code **5635714** (or the address `http://aftv.news/5635714`) to fetch the same `Zibbit.apk`. Downloader needs permission to install unknown apps, the same as in step 2.

On a big screen the app switches to big-screen mode: large type, d-pad navigation, and a menu on long-press. It signs in with a code you approve on your phone.

Plain `http` addresses are only accepted for home-network hosts (192.168.x.x, 10.x.x.x, 172.16-31.x.x and local names). Anything reached over the internet needs `https`; see the Setup guide.

## What's new

**Server 3.0.0 and Android app 3.0** (November 2026)

Zibbit started as a live-TV player. Now it also rewinds and records channels, plays your provider's movies and shows, and runs on your phone, tablet and big screen.

- Rewind live TV up to an hour and press Go live to return. Record a show from the rewind buffer, or from now until you stop.
- Movies & Shows: browse a provider's movies and series, search, choose your categories, and Continue watching. Download a movie, episode or season to your own folder.
- A PIN-based content lock for groups you want out of sight. Admin settings > Locks.
- Provider logins and guide addresses are now stored encrypted. Admins get a recovery key and encrypted server backups under Security.
- Sign out everywhere, All channels, E-AC-3 audio converted for browsers that can't play it, Refresh channel list, and install as an app from the browser.
- Android app: big-screen mode, sign in with a code from your phone, and the app can update itself.
- Numerous security enhancements and bug fixes since 2.x.

**Upgrading from 2.x:** back up your config folder first. Once 3.0 has started, saved logins are encrypted and 2.x can't read them. See [Upgrading](docs/UPGRADING.md).

**Server 2.1.0 and Android app 2.1** (November 2026)

- HDHomeRun antenna tuners as a source, with the guide filled in automatically if you have SiliconDust's guide subscription.
- Tablo 4th Gen as a source (experimental), with its own guide.
- The server version shows at the bottom of Settings, and image tags now follow releases: `latest`, `2.1` and `2.1.0`.
- The Android app opens your last server on launch; switching is under Settings > Change server.
- The player says why the browser refused to start playback instead of staying on "Ready".

## Notes

- Unofficial and provided as is, with no warranty. For personal use.
- The server's source code is at [github.com/jamiecrout/zibbit](https://github.com/jamiecrout/zibbit) (MIT license).
