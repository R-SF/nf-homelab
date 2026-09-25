# Todo
- [ ] Change `ntfy.sh` topic for OOM drive monitor.
- [ ] Tdarr setup
  - [ ] Convert all torrents before running anything. 
  - [ ] DoVi converter.
  - [ ] 2.1 auto converter.
  - [ ] x256 converter.
- [ ] qBittorrent set up torrent seed limit + removal.
  - [ ] Tdarr - See if there's a way to only process once removed.
- [ ] NzbGet + qBittorrent - Daytime download limits
  - [ ] See if there's a way to pause download when someone is streaming.
- [ ] Maybe add ntfy to the watchdog.
  - Code can be found below.
- [ ] Add ntfy alert on login.
- [ ] Automated config backups.
- [ ] Add container health checks.
- [ ] Change HDD file format to ext4.
- [ ] Change source for TV shows in plex.

# Watchdog ntfy implementation.
```
nord-vpn-watchdog:
    container_name: core-nord-vpn-watchdog
    image: docker:cli
    restart: ${GLOBAL_RESTART_STRATEGY}

    volumes:
      - ${GLOBAL_LOCALTIME}
      - ${GLOBAL_DOCKER_SOCK}

    environment:
      - TZ=${GLOBAL_TIMEZONE}
      - NTFY_URL=https://ntfy.sh
      - NTFY_TOPIC=${NTFY_TOPIC_WATCHDOG}

    entrypoint: >
      sh -c '
        apk add --no-cache curl >/dev/null 2>&1;
        FIRST=1;
        docker events --filter "container=core-nord-vpn-meshnet" --filter "event=start" |
        while read line; do
          if [ "$$FIRST" = "1" ]; then
            FIRST=0;
            echo "$$(date "+%Y-%m-%d %H:%M:%S") - Skipping initial start event";
            continue;
          fi;
          echo "[$$(date "+%Y-%m-%d %H:%M:%S")] VPN restarted, restarting dependent containers...";
          curl -fsS -d "VPN meshnet restarted, restarting dependent containers" "$${NTFY_URL}/$${NTFY_TOPIC}" >/dev/null 2>&1;
          RESTARTED="";
          for c in $$(docker ps -q --filter "label=watchdog-reboot"); do
            name=$$(docker inspect --format "{{.Name}}" "$$c" | sed "s#^/##");
            echo "[$$(date "+%Y-%m-%d %H:%M:%S")] >> Restarting $$name";
            docker restart "$$c";
            RESTARTED="$$RESTARTED $$name";
          done;
          curl -fsS -d "Finished restarting:$$RESTARTED" "$${NTFY_URL}/$${NTFY_TOPIC}" >/dev/null 2>&1;
        done'
```