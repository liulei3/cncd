+++
date = "2017-02-06T22:57:35+07:00"
title = "Intermediate Representation Specification"
version = "1.0-draft"

[params]
  [[editors]]
    name = "bradrydzewski"
    link = "https://github.com/bradrydzewski"

  [versions]
    current = "https://github.com/cncd/runtime-spec"
    previous = [
      "https://github.com/cncd/runtime-spec/tree/1.0.0",
    ]

  [participate]
    issues = "https://github.com/cncd/runtime-spec/issues"
+++

# Abstract

This specification introduces an intermediate representation (IR) for defining continuous delivery pipelines and their container execution environments. A continuous delivery pipeline is an automated manifestation of your process for getting software from version control to release.

# Introduction

_This section is non-normative._

This specification introduces an intermediate representation (IR) for defining continuous delivery pipelines and their container execution environments. The intermediate representation should be machine writable, machine executable, and platform agnostic.

## Frontends

_This section is non-normative._

The intermediate representation is not intended to be written by humans. Instead higher-level file formats are compiled to the intermediate representation. These compilers are known as frontends. Example frontend compilers may include:

* Travis Yaml
* GitLab Yaml
* Bitbucket Pipeline Yaml

The Cloud Native Continuous Delivery working group is also authoring a [specification]({{< relref "yaml.md" >}}) for a YAML representation of a continuous delivery pipeline. This will include a reference implementation for compiling to the intermediate representation.

## Backends

_This section is non-normative._

The intermediate representation should be platform agnostic. This means it can be consumed by different container engines and orchestration platforms. These engines and platforms are known as backends. Example backends may include:

* Docker
* Docker Swarm
* Kubernetes
* Rocket

# Conformance

As well as sections marked as non-normative, all authoring guidelines, diagrams, examples, and notes in this specification are non-normative. Everything else in this specification is normative.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119](https://tools.ietf.org/html/rfc2119).

# Format

The intermediate representation is a JSON document that defines the pipeline execution environment and execution steps. The intermediate representation consists of a top-level object that contains both required and optional members. Each of the members are defined below, as well as how their values are processed.

```typescript
interface Spec {
  pipeline: Stage[]
  networks: Network[]
  volumes: Volume[]
}
```

The `pipeline` attribute must contain a collection of one or more `stage` objects. The stages defined in the pipeline must be executed sequentially in the oder in which they are defined.

The `networks` attribute should contain a collection of one or more `network` objects. Networks may be referenced by individual steps and must be created prior to executing steps where a reference exists.

The `volumes` attribute should contain a collection of one or more `volume` objects. Volumes may be referenced by individual steps and must be created prior to executing steps where a reference exists.

## The `Stage` interface

The `stage` object represents a set of processes that are grouped together and executed in parallel. The stage does not complete until all processes have exited.

```typescript
interface Stage {
  name: string
  steps: Step[]
}
```

The `name` attribute is required and must match `[a-zA-Z0-9_-]`

The `steps` attribute is required and must contain at least one `step`. If the stage contains multiple steps, each step is executed in parallel. The runtime agent should wait until all steps complete prior before it continues to the next stage in the pipeline.

## The `Step` interface

The `step` object defines a container process in the pipeline. The step attributes define how a container is created and started. Each of these attributes are defined below, as well as how their values are processed.

```typescript
class Step {
  name: string
  alias: string
  image: string
  pull: boolean
  detached: boolean
  privileged: boolean
  working_dir: string
  environment: [string, string]
  entrypoint: string[]
  command: string[]
  devices: string[]
  extra_hosts: string[]
  dns: string[]
  dns_search: string[]
  shm_size: number
  tmpfs: string[]
  volumes: string[]
  networks: Network[]
  resources: Resources
  auth_config: AuthConfig
  on_failure: boolean
  on_success: boolean
}
```

### The `name` attribute

The name of the container. This should be globally unique and must match `[a-zA-Z0-9_-]`.

### The `alias` attribute

The name of the container as defined by the user (i.e. user-friendly name). Containers in the pipeline will be able to access the container user the alias as the hostname. This value must be unique within the pipeline, and must match `[a-zA-Z0-9_-]`.

### The `image` attribute

The fully qualified image to start the container from. When defining a docker image the full name, including the tag, must be provided.

```
image: redis:latest
image: library/redis:latest
image: index.docker.io/library/redis:latest
```

### The `pull` attribute

Pull the latest version of the image from the remote image registry.

### The `detached` attribute

Start the container and run in the background. The parent stage must not wait for the container to complete execution. The container exit code must be ignored and must not cause the pipeline to exit on failure.

### The `privileged` attribute

Start the container with extended privileges.

### The `working_dir` attribute

Start the container in the specified working directory. This must be the absolute path of the directory inside the container. The container engine (e.g. Docker) may create the working directory if it does not exist, however, this behavior may vary.

### The `environment` attribute

Set environment variables in the container.

```json
{
  "GOPATH": "/go",
  "DOCKER_HOST": "unix:///var/run/docker.sock"
}
```

### The `entryptoint` attribute

Override the default entrypoint.

### The `command` attribute

todo

### The `devices` attribute

Expose devices to a container. For example, a specific block storage device or loop device or audio device can be added to an otherwise unprivileged container.

```json
[
  "/dev/sdc:/dev/xvdc"
]
```

### The `dns` attribute

Sets the IP addresses added as server lines to the container's `/etc/resolv.conf` file.

```json
[
  "8.8.8.8",
  "9.9.9.9"
]
```

### The `dns_search` attribute

Sets the domain names that are searched when a bare unqualified hostname is used inside of the container, by writing search lines into the container's `/etc/resolv.conf`.

```json
[
  "dc1.example.com",
  "dc2.example.com"
]
```

### The `extra_hosts` attribute

Add additional lines to the container's `/etc/hosts` file.

```json
[
  "somehost:162.242.195.82",
  "otherhost:50.31.209.229"
]
```

### The `shm_size` attribute

Sets the size of `/dev/shm` in bytes:

```json
{
  "shm_size": 64000000
}
```


### The `tmpfs` attribute

Mount an empty temporary file system inside of the container.

```json
[
  "/run",
  "/tmp"
]
```

### The `volumes` attribute

Mount paths or named volumes, optionally specifying a path on the host machine

### The `networks` section

```typescript
interface Network {
  name: string
  aliases: string[]
}
```

### The `resources` section

```typescript
interface Resources {
  limits: Limits
  reservations: Reservations
}

interface Limits {
  cpus: string
  memory: number
}

interface Reservations {
  cpus: string
  memory: number
}
```

### The `auth_config` section

```typescript
interface AuthConfig {
  username: string
  password: string
  email: string
}
```

### The `on_success` attribute

todo

### The `on_failure` attribute

todo

## The `Networks` section

```typescript
interface Volume {
  name: string
  driver: string
  driver_opts: [string, string]
}
```

### The `name` attribute

todo

### The `driver` attribute

todo

### The `driver_opts` attribute

todo

## The `Volumes` section

```typescript
interface Volume {
  name: string
  driver: string
  driver_opts: [string, string]
}
```

### The `name` attribute

todo

### The `driver` attribute

todo

### The `driver_opts` attribute

todo

# Security

_This section is non-normative._

Backends should audit the use of privileged features and capabilities because hostile authors could otherwise use these settings to compromise the host machine.

# Disk Space

_This section is non-normative._

Backends should limit the total amount of space allowed for volume storage, because hostile authors could otherwise use this feature to exhaust the system's available disk space.

Backends should also limit the total amount of space allowed for caching images (e.g. Docker images), and should regularly purge the cache to remove unused or stale images.

# Examples

_This section is non-normative._

This section shows how frontends can make use of the various features of this specification.

## Example Pipeline

_This section is non-normative._

In the following example the intermediate representation defines two stages. The first stage clones the github project to a shared volume. The second stage executes the test suite for the project.

```json
{
  "pipeline": [
    {
      "name": "clone_stage",
      "steps": [
        {
          "name": "git_clone_step",
          "image": "git:latest",
          "entrypoint": [
            "/bin/sh",
            "-c"
          ],
          "command": [
            "git clone git://github.com/foo/bar.git /go/src/github.com/foo/bar"
          ],
          "volumes": [
            "default:/go"
          ],
          "on_success": true,
          "on_failure": false
        }
      ],
    },
    {
      "name": "test_stage",
      "steps": [
        {
          "name": "go_test_step",
          "image": "golang:latest",
          "entrypoint": [
            "/bin/sh",
            "-c"
          ],
          "command": [
            "go test -v github.com/foo/bar"
          ],
          "volumes": [
            "default:/go"
          ],
          "on_success": true,
          "on_failure": false
        }
      ],
    }
  ],

  "volumes": [
    {
      "name": "default",
      "driver": "local"
    }
  ]
}
```

## Example Pipeline with Services

_This section is non-normative._

In the following example the intermediate representation defines a service container (redis) that runs in detached mode, which is non-blocking. Subsequent steps in the pipeline are able to access the service container using its alias hostname.

```json
{
  "pipeline": [
    {
      "name": "clone_stage",
      "steps": [
        {
          "name": "clone",
          "image": "git:latest",
          "entrypoint": [
            "/bin/sh",
            "-c"
          ],
          "command": [
            "git clone git://github.com/foo/bar.git /go/src/github.com/foo/bar"
          ],
          "volumes": [
            "default:/go"
          ],
          "on_success": true,
          "on_failure": false
        }
      ],
    },
    {
      "name": "service_stage",
      "steps": [
        {
          "name": "redis_step",
          "alias": "redis",
          "image": "redis:latest",
          "detach": true,
          "networks": [
            {
              "name": "default",
              "aliases": [ "redis" ]
            }
          ],
          "volumes": [
            "default:/go"
          ],
          "on_success": true,
          "on_failure": false
        }
      ],
    },
    {
      "name": "test_stage",
      "steps": [
        {
          "name": "test_step",
          "image": "golang:latest",
          "entrypoint": [
            "/bin/sh",
            "-c"
          ],
          "command": [
            "go test -v github.com/foo/bar"
          ],
          "networks": [
            {
              "name": "default",
              "aliases": [ "redis" ]
            }
          ],
          "volumes": [
            "default:/go"
          ],
          "on_success": true,
          "on_failure": false
        }
      ],
    }
  ],

  "networks": [
    {
      "name": "default",
      "driver": "bridge"
    }
  ],

  "volumes": [
    {
      "name": "default",
      "driver": "local"
    }
  ]
}
```

# References

## Normative References

[JSON]
: A. Barth. HTTP State Management Mechanism. April 2011. https://tools.ietf.org/html/rfc6265

## Informative References

[DOCKER]
: [Docker Run Reference](https://docs.docker.com/engine/reference/run/), Docker Inc.

[DOCKER COMPOSE]
: [Compose File Reference](https://docs.docker.com/compose/compose-file/), Docker Inc.