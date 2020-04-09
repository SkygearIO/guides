# Deployment Image

Skygear Cluster supports building Docker image for your micro-services before deployment. Currently we support build image using two methods: from template, and from Dockerfile.

Developer can also specify a pre-built image, which Skygear Cluster would use it directly.

### Build Image from template

To build image from template, the `context` and `template` key must be specified in the micro-service configuration.

The `context` key specifies the location of the micro-service code, it will be copied into the image at build time.

The `template` key specifies the image template to use. Currently, we support three templates:

* `nodejs:8`
* `nodejs:10`
* `nodejs:12`

### Build Image from Dockerfile

To build image from Dockerfile, the `context` key must be specified in the micro-service configuration.

The `context` key specifies the context of image build. `Dockerfile` must exists at the root of context, or otherwise specified using `dockerfile` key.

### Using pre-built image

To use pre-built image, the `image` key must be specified in the micro-service configuration.

The image pull policy is [IfNotPresent](https://kubernetes.io/docs/concepts/containers/images/#updating-images). So you should use a different tag if you want to pull a newer image. Using a constant tag like `:latest` always resolve to the first pulled image.

To pull image from private image registry, the `image_pull_secret` key must be specified. The value should a secret name, refer to a secret created with type `dockerconfigjson`:

```text
$ skycli secret create --type=dockerconfigjson --name="<secret name>" --file="<docker_json_file>"
```

