# Automation

```bash
cd infrastructure/automation/
export KUBECONFIG=kubeconfig-data-center
oc login -u admin --insecure-skip-tls-verify https://api.cluster-...

export KUBECONFIG=kubeconfig-edge-gateway
oc login -u admin --insecure-skip-tls-verify https://edge-gateway.lan:6443
```

## Rebuild execution environment

```bash
podman login registry.redhat.io
export VERSION=$(date +%Y%m%d%H%M)

ansible-builder build \
    --verbosity 3 \
    --container-runtime podman \
    --file execution-environment-fedora-based.yml \
    --extra-build-cli-args "--platform linux/amd64,linux/arm64  --manifest quay.io/cloud-native-robotz-hackathon/infrastructure-robot-execution-environment:$VERSION"

    --tag quay.io/cloud-native-robotz-hackathon/infrastructure-robot-execution-environment:$VERSION \
podman login quay.io
podman push quay.io/cloud-native-robotz-hackathon/infrastructure-robot-execution-environment:$VERSION
```

