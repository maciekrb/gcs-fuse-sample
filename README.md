# GCSFuse storage for containers

This directory contains a sample deployment using Google Cloud Storage as a multi RW file system that can be mounted from different containers.

The deployment file contains the required configuration for a container to start with a Google Cloud Storage bucket mounted in a given path.

The most note worthy parts of the configuration are the following:

```yaml
securityContext:
  privileged: true
  capabilities:
    add:
      - SYS_ADMIN

```
For the container to have access to `/dev/fuse` it has to run with `SYS_ADMIN` capabilities.

```yaml
lifecycle:
  postStart:
    exec:
      command: ["gcsfuse", "-o", "nonempty", "test-bucket", "/mnt/test-bucket"]
  preStop:
    exec:
      command: ["fusermount", "-u", "/mnt/test-bucket"]
```
As no real Kubernetes volumes are really involved, the whole thing can be implemented by using `lifecycle` directives, a `postStart` will mount the `gcsfuse` volume and a `preStop` will unmount it.

The big catch is that for this to work, the container has to be built with `gcsfuse`. The `Dockerfile` includes a base build for debian jessie.

Unfortunately as the the `gcsfuse` does not sync the files, it is not possible to share the file system with other containers in the pod via a `volumes[].emptyDir.{}` directive.

## References

https://cloud.google.com/storage/docs/gcs-fuse
https://github.com/GoogleCloudPlatform/gcsfuse
https://karlstoney.com/2017/03/01/fuse-mount-in-kubernetes/
