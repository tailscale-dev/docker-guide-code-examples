# Podman Compatibility

This document provides information on using Podman as an alternative to Docker for the Tailscale examples in this repository.

## Compatibility Assessment

After testing, we've confirmed that **Podman is fully compatible** with all the Tailscale Docker examples in this repository. Podman 5.4.1+ can be used as a drop-in replacement for Docker with minimal to no modifications.

## Key Compatibility Points

1. **Docker Compose Files**: All Docker Compose files work with `podman compose` without modification.
2. **Network Features**: Podman supports all network features required by Tailscale:
   - `net_admin` capability
   - `/dev/net/tun` device access
   - Shared network namespaces via `network_mode: service:container_name`
3. **Volumes & Environment Variables**: All volume mounts and environment variables work as expected.
4. **Container Orchestration**: Multi-container setups work correctly with proper dependency management.

## Using Podman Instead of Docker

To use these examples with Podman:

1. Replace `docker` with `podman` in commands:
   ```
   podman run ...
   podman pull ...
   podman exec ...
   ```

2. Replace `docker compose` with `podman compose`:
   ```
   podman compose up -d
   podman compose down
   podman compose logs
   ```

3. All other Docker CLI commands have direct Podman equivalents with the same syntax.

## Notes

- Podman's rootless mode can be used with these examples for enhanced security.
- The `podman compose` command may display a message about using an external Docker Compose provider, which is normal and can be ignored.
- macOS users will need to install Podman Desktop or use Podman Machine to create a Linux VM for running containers.
- If you see a warning about the `version` attribute being obsolete in compose files, this can be safely ignored.

## Known Differences

- Podman's network implementation is slightly different than Docker's, but for these examples, the functionality remains the same.
- Podman uses a different default filesystem driver, but this doesn't affect these examples.
- Some advanced Docker-specific features may not be available in Podman, but none of these are used in the examples in this repository.
