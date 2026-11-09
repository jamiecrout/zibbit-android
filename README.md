# Zibbit

Zibbit is a self-hosted, touch-first streaming player that you run on your own server. It gives you a real guide grid, your own channel groups and favorites, and one place to manage several sources. It works in any browser, and this repository has the Android app.

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

Then open `http://<server-address>:8080`. The first sign-in is `admin` / `admin`, and you are asked to choose a new password straight away. Open **Settings > Admin settings** to add your own sources and the people who will watch.

Step-by-step guides, including Docker Compose and Unraid:

- [Setup](docs/SETUP.md): install, first sign-in, reaching it from other devices, updating
- [Using it](docs/USING.md): a tour of the screens for the people who watch
- [Configuration](docs/CONFIGURATION.md): roles, settings, environment variables, admin commands
- [Upgrading](docs/UPGRADING.md): moving from the earlier single-file version

## 2. Watch

**In a browser:** open the server address on any phone, tablet or computer. You can add the page to a phone's home screen.

**With the Android app:** [Zibbit.apk](Zibbit.apk) (Android 7.0 or newer).

SHA-256: `20ff334ec73884d009e520b00b6f5ef4cab698dcb70d847e7e10436d854fd28f`
Signing certificate SHA-256: `a7a9fd99926ce22ffa3d8325fcd254c0b907b3bc9069bb98de2fed6c39b3464a`

1. Download `Zibbit.apk` on the phone or tablet.
2. Open it. Android will ask you to allow installs from that app (your browser or file manager). Allow it for this install.
3. Open **Zibbit**. On first launch it asks for the address of your server, for example `zibbit.example.com` or `192.168.1.20:8080`, then saves it to a list so you can switch servers later.

Plain `http` addresses are only accepted for home-network hosts (192.168.x.x, 10.x.x.x, 172.16-31.x.x and local names). Anything reached over the internet needs `https`; see the Setup guide.

## Notes

- Unofficial and provided as is, with no warranty. For personal use.
- The server's source code is not published yet.
