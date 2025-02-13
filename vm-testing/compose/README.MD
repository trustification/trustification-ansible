# External Services locally With Docker compose

### compose

Requires `podman-compose` to run dependent services.

For Linux systems only:

```shell
export SELINUX_VOLUME_OPTIONS=':Z'
```
Start:
```shell
cd vm-testing/compose
podman-compose -f compose.yaml up
```
Shutdown:
To stop or to clean the previous running compose containers
```shell
cd vm-testing/compose
podman-compose -f compose.yaml down
```
