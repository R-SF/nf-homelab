# Todo
- [ ] Tdarr setup
  - [ ] Convert all torrents before running anything. 
  - [ ] DoVi converter.
  - [ ] 2.1 auto converter.
  - [ ] x256 converter.
- [ ] qBittorrent set up torrent seed limit + removal.
  - [ ] Tdarr - See if there's a way to only process once removed.
- [ ] NzbGet + qBittorrent - Daytime download limits
  - [ ] See if there's a way to pause download when someone is streaming.
- [ ] Add ntfy alert on login.
- [ ] Automated config backups.
- [ ] Add container health checks.
- [ ] Change HDD file format to ext4.
- [ ] Change source for TV shows in plex.

# Done
- [x] Change `ntfy.sh` topic for OOM drive monitor.
- [x] Add `.env.example` files for each docker stack.
- [x] Maybe add ntfy to the watchdog.

# Potential Bugs
- None

# Confirmed Bugs
## `core-nord-vpn-watchdog`
### Potential failure point
Priority: Low
Description:
If this container is restarted independently, while `core-nord-vpn-meshnet` is already up, it loses track of the "first" start of `core-nord-vpn-meshnet`.
This then means that it will not restart other containers unless `core-nord-vpn-meshnet` triggers the `start` event 2 times.
- The first time, it incorrectly tracks the "first" start event.
- The second time, it starts to restart all containers.