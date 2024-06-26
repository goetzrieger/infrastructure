# Rebuild execution environment

```bash
podman login registry.redhat.io
export VERSION=$(date +%Y%m%d%H%M)

ansible-builder build \
    --verbosity 3 \
    --container-runtime podman \
    --file execution-environment-fedora-based.yml \
    --tag quay.io/cloud-native-robotz-hackathon/infrastructure-robot-execution-environment:$VERSION

podman login quay.io
podman push quay.io/cloud-native-robotz-hackathon/infrastructure-robot-execution-environment:$VERSION
```
