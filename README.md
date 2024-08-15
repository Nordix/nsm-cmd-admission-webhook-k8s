# Build

## Build cmd binary locally

You can build the locally by executing

```bash
go build ./...
```

## Build Docker container

You can build the docker container by running:

```bash
docker build .
```

# Usage

## Environment config

* `NSM_NAME`                    - Name of current admission webhook instance (default: "admission-webhook-k8s")
* `NSM_SERVICE_NAME`            - Name of service that related to this admission webhook instance (default: "default")
* `NSM_NAMESPACE`               - Namespace where admission webhook is deployed (default: "default")
* `NSM_ANNOTATION`              - Name of annotation that means that the resource can be handled by admission-webhook (default: "networkservicemesh.io")
* `NSM_LABELS`                  - Map of labels and their values that should be appended for each deployment that has Config.Annotation
* `NSM_NSURL_ENV_NAME`          - Name of env that contains NSURL in initContainers/Containers
* `NSM_INIT_CONTAINER_IMAGES`   - List of init containers that should be appended for each deployment that has Config.Annotation
* `NSM_CONTAINER_IMAGES`        - List of containers that should be appended for each deployment that has Config.Annotation
* `NSM_ENVS`                    - Additional Envs that should be appended for each Config.ContainerImages and Config.InitContainerImages
* `NSM_WEBHOOK_MODE`            - Default 'spire' mode uses spire certificates and external webhook configuration. Set to 'selfregister' to use the automatically generated webhook configuration (default: "spire")
* `NSM_CERT_FILE_PATH`          - Path to certificate. Preferred use if specified
* `NSM_KEY_FILE_PATH`           - Path to RSA/Ed25519 related to Config.CertFilePath. Preferred use if specified
* `NSM_CA_BUNDLE_FILE_PATH`     - Path to cabundle file related to Config.CertFilePath. Preferred use if specified
* `NSM_OPEN_TELEMETRY_ENDPOINT` - OpenTelemetry Collector Endpoint (default: "otel-collector.observability.svc.cluster.local:4317")
* `NSM_METRICS_EXPORT_INTERVAL` - interval between mertics exports (default: "10s")
* `NSM_SIDECAR_LIMITS_MEMORY`   - Lower bound of the NSM sidecar memory limit (in k8s resource management units) (default: "80Mi")
* `NSM_SIDECAR_LIMITS_CPU`      - Lower bound of the NSM sidecar CPU limit (in k8s resource management units) (default: "200m")
* `NSM_SIDECAR_REQUESTS_MEMORY` - Lower bound of the NSM sidecar requests memory limits (in k8s resource management units) (default: "40Mi")
* `NSM_SIDECAR_REQUESTS_CPU`    - Lower bound of the NSM sidecar requests CPU limits (in k8s resource management units) (default: "100m")
* `NSM_KUBELET_QPS`             - kubelet QPS config (default: "50")
* `NSM_PPROF_ENABLED`           - is pprof enabled (default: "false")
* `NSM_PPROF_LISTEN_ON`         - pprof URL to ListenAndServe (default: "localhost:6060")

# Testing

## Testing Docker container

Testing is run via a Docker container.  To run testing run:

```bash
docker run --privileged --rm $(docker build -q --target test .)
```

# Debugging

## Debugging the tests
If you wish to debug the test code itself, that can be acheived by running:

```bash
docker run --privileged --rm -p 40000:40000 $(docker build -q --target debug .)
```

This will result in the tests running under dlv.  Connecting your debugger to localhost:40000 will allow you to debug.

```bash
-p 40000:40000
```
forwards port 40000 in the container to localhost:40000 where you can attach with your debugger.

```bash
--target debug
```

Runs the debug target, which is just like the test target, but starts tests with dlv listening on port 40000 inside the container.

## Debugging the cmd

When you run 'cmd' you will see an early line of output that tells you:

```Setting env variable DLV_LISTEN_FORWARDER to a valid dlv '--listen' value will cause the dlv debugger to execute this binary and listen as directed.```

If you follow those instructions when running the Docker container:
```bash
docker run --privileged -e DLV_LISTEN_FORWARDER=:50000 -p 50000:50000 --rm $(docker build -q --target test .)
```

```-e DLV_LISTEN_FORWARDER=:50000``` tells docker to set the environment variable DLV_LISTEN_FORWARDER to :50000 telling
dlv to listen on port 50000.

```-p 50000:50000``` tells docker to forward port 50000 in the container to port 50000 in the host.  From there, you can
just connect dlv using your favorite IDE and debug cmd.

## Debugging the tests and the cmd

```bash
docker run --privileged -e DLV_LISTEN_FORWARDER=:50000 -p 40000:40000 -p 50000:50000 --rm $(docker build -q --target debug .)
```

Please note, the tests **start** the cmd, so until you connect to port 40000 with your debugger and walk the tests
through to the point of running cmd, you will not be able to attach a debugger on port 50000 to the cmd.
