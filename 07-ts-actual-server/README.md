# tailscale-dev/docker-guide-code-examples/07-ts-actual-server

This has been tested on Synology DSM 7.2.2 and running Synology Container Manager.

Steps to get this working:

1. Create a new `Project` in Container Manager. Name it `actual-server` or something identifiable.
2. Under the `Yaml Configurations` tab, paste this `compose.yaml` file as is. 
3. Modify the yaml and insert your own Tailscale AuthKey.
4. Using the Synology File Station (or similar) make sure to create the correct folders so that the Docker files can save their data in the mounted volumes.
5. Copy up the `serve-config.json` into the `/config` folder volume mount as well.
6. If you haven't done so already, save and build the project. After a few minutes it should build and start the Actual Server container.
7. Open up your Actual Server client and add the `https://<FQDN.TS.NET>` of your container and it should see it and start syncing!

Go and budget away! Please file a bug issue if you run into any problems with this example!

Actual Server Docker info:

- https://actualbudget.com/docs/install/docker
- Docker Compose file - https://github.com/actualbudget/actual/blob/master/packages/sync-server/docker-compose.yml

