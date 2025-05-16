You should probably grab the latest version of the [immich docker-compose file](https://github.com/immich-app/immich/blob/main/docker/docker-compose.yml) and add the tailscale container config found here to it. 

Note that `serve-config.json` requires magicDNS and HTTPS Certificates to both be enabled in your tailscale admin console. You can find both options in the DNS pane.