# Zibbit for Android

The Android app for [Zibbit](https://ghcr.io/jamiecrout/zibbit), a self-hosted, touch-first streaming player that you run on your own server. The app is a thin full-screen wrapper: it opens your own Zibbit page, and nothing else. It does not include any channels, playlists or provider logins.

**Download:** [Zibbit.apk](Zibbit.apk) (Android 7.0 or newer)

SHA-256: `20ff334ec73884d009e520b00b6f5ef4cab698dcb70d847e7e10436d854fd28f`
Signing certificate SHA-256: `a7a9fd99926ce22ffa3d8325fcd254c0b907b3bc9069bb98de2fed6c39b3464a`

## Install the app

1. Download `Zibbit.apk` on the phone or tablet.
2. Open it. Android will ask you to allow installs from that app (your browser or file manager). Allow it for this install.
3. Open **Zibbit**. On first launch it asks for the address of your Zibbit server, for example `zibbit.example.com` or `192.168.1.20:8080`, then saves it to a list so you can switch servers later.

Plain `http` addresses are only accepted for home-network hosts (192.168.x.x, 10.x.x.x, 172.16-31.x.x and local names). Anything reached over the internet needs `https`.

## You need a Zibbit server first

Zibbit runs as a Docker container. With Docker installed:

```
docker run -d --name zibbit --restart unless-stopped -p 8080:8080 -v /path/to/zibbit-config:/config ghcr.io/jamiecrout/zibbit:latest
```

Then open `http://<server-address>:8080` in a browser. The first sign-in is `admin` / `admin`, and you must change the password straight away. After that, open **Settings > Admin settings** to add your own sources. Settings and sign-ins are kept in the folder you mounted at `/config`.

The same page works in any browser, and it can be added to a phone's home screen if you would rather not install the app.

## Notes

- Unofficial and provided as is, with no warranty. For personal use.
- Zibbit supplies no content. You bring your own sources and are responsible for having the right to use them.
- The server's source code is not published yet.
