# docker-rsync-cron

This is a cronjob docker container which is used to regularly sync two
docker volumes using rsync.

This image is based off of
[dkruger/cron](https://hub.docker.com/r/dkruger/cron/) which provides a simple
cron implementation. The cron daemon is used to execute `rsync` which will
copy the contents of the `rsync_src` volume to the `rsync_dst` volume. The
specific options are configurable via environment variables.

Checkout the [example docker-compose.yml](example/docker-compose.yml) for an
exmaple of setting up the container with named NFS volumes using `Netshare`.

## Using the image

The image is configurable via environment variables for configuring the rsync
command settings:

* `RSYNC_CRONTAB`: The crontab time entry, defaults to nightly at midnight
* `RSYNC_OPTIONS`: Flags passed to `rsync`, defaults to
`--archive --timeout=3600`
* `RSYNC_UID`: The UID to use when calling rsync, defaults to 0
* `RSYNC_GID`: The GID to use when calling rsync, defaults to 0

The image defines two volumes: `/rsync_src`, and `/rsync_dst`. The contents of
`/rsync_src` will be copied to `/rsync_dst` on the interval defined by the
crontab entry.

The `rsync` command is called using the format `rsync /rsync_src/ /rsync_dst`,
so that the *contents* of `/rsync_src` will be copied no the directory itself.

Here is an example command for executing the container two rsync two NFS mounts
using the `Netshare` volume plugin:
```bash
docker run \
    --name my-nfs-sync \
    --volume-driver=nfs \
    -v master-svr/volume1/master:/rsync_src \
    -v local-svr/export:/rsync_dst \
    -e RSYNC_OPTIONS="--archive --timeout=3600 --delete"
    dkruger/rsync-cron:latest
```

## About permissions

Depending on your volume store, permissions might be an issue. For example some
NAS implementations are very picky when it comes to the UID and GID of the
process reading/writing. As such you may need to specify a UID and GID that has
the correct permissions for the volumes. Note that these are the ID numbers,
not the names.
