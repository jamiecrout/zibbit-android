# Upgrading from v1

v2 reads the v1 config as it is: `providers.json`, `users.json`, `settings.json`, `secret.key`,
`epgmap.json`, per-person `state-*.json` and the cache. Nothing is converted, and a v1 install tested
this way keeps its sign-ins, groups and favorites (this is part of `npm test`).

What differs:

- **Nobody is an admin yet.** Existing profiles become regular users, with no forced password change, and
  **no default `admin/admin` account is created** on an install that already has profiles. Make one person
  an admin from the command line, then everything else is done in the browser:

  ```
  docker exec -it zibbit node server.js promote <name>
  ```

  (`demote <name>` reverses it. The last admin can't be demoted from the browser.)
- **Installs with no profiles at all** (open, or `APP_PASSWORD`) now get the first-run `admin` account.
  `APP_PASSWORD`, if set, becomes the first password of that account, which must still be changed at first
  sign-in. The shared-password mode itself is gone.
- Provider and settings edits made in the browser are written to `providers.json` and `settings.json`
  (the same files and formats; unknown keys are preserved). Changes take effect immediately, with no
  restart.
- `providers.example.json` is no longer written.

Rolling back to v1 is just running the v1 code against the same config. v1 ignores the extra fields v2 adds
(`role`, `mustChange`, `type`, the new settings keys). Providers added as M3U playlists are skipped by v1.
