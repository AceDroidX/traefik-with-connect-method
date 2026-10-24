## Allow NaiveProxy CONNECT Requests (Custom Patch)

This fork of Traefik Proxy v3.7.15 permits CONNECT requests to reach HTTP routing when either of these process environment variables has a nonempty value that exactly matches its request header:

| Environment Variable | Request Header |
| --- | --- |
| `TRAEFIK_NAIVE_CONNECT_SECRET` | `naive` |
| `TRAEFIK_NAIVE_PROXY_AUTH` | `Proxy-Authorization` (the complete value, including `Basic `) |

Set the variables on the Traefik container, for example with Docker Compose:

```yaml
environment:
  TRAEFIK_NAIVE_CONNECT_SECRET: ${NAIVE_CONNECT_SECRET}
  TRAEFIK_NAIVE_PROXY_AUTH: ${NAIVE_PROXY_AUTH}
```

An unset or empty variable disables its matching condition. If neither condition matches, Traefik returns `501 Not Implemented` before HTTP routing. Other HTTP methods are unaffected. Values are read when the HTTP server handler is created; restart Traefik after changing the environment.

Keep the existing HTTP routers that exactly match these headers and select the NaiveProxy service on port 8080. Set each router's header value to the corresponding environment variable value. The backend can continue to run `naive --listen=http://0.0.0.0:8080 --log`. Passing this check only permits HTTP routing; it does not select a service or replace router matching.

CONNECT tunneling remains unsupported upstream. This patch does not establish correct HTTP/2 or HTTP/3 tunnel shutdown, flow control, or resource cleanup. Review the routing, middleware, proxy, and transport code and retest tunneling before every version upgrade.

### Publish the Docker Image

The [Build Docker workflow](.github/workflows/build-docker.yaml) runs on pushes and manual dispatches. It builds the WebUI and binary with the repository's build settings and publishes only `linux/amd64` to `acedroidx/traefik_with_connect_method:<version>`.

Set the repository Actions secrets `DOCKER_USERNAME` and `DOCKER_PASSWORD` (a Docker Hub access token with write permission). The workflow reads the branch name as the build version and image tag, for example `v3.7.15`. For a manual run, select the version branch to build. The branch name must be a valid Docker image tag. The same value is embedded in the binary and used as the image tag.

---

<p align="center">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="docs/content/assets/img/traefik.logo-dark.png">
      <source media="(prefers-color-scheme: light)" srcset="docs/content/assets/img/traefik.logo.png">
      <img alt="Traefik" title="Traefik" src="docs/content/assets/img/traefik.logo.png">
    </picture>
</p>

[![Docs](https://img.shields.io/badge/docs-current-brightgreen.svg)](https://doc.traefik.io/traefik)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/traefik/traefik/blob/master/LICENSE.md)
[![Join the community support forum at https://community.traefik.io/](https://img.shields.io/badge/style-register-green.svg?style=social&label=Discourse)](https://community.traefik.io/)
[![Twitter](https://img.shields.io/twitter/follow/traefik.svg?style=social)](https://twitter.com/intent/follow?screen_name=traefik)

Traefik (pronounced _traffic_) is a modern HTTP reverse proxy and load balancer that makes deploying microservices easy.
Traefik integrates with your existing infrastructure components ([Docker](https://www.docker.com/), [Swarm mode](https://docs.docker.com/engine/swarm/), [Kubernetes](https://kubernetes.io), [Consul](https://www.consul.io/), [Etcd](https://coreos.com/etcd/), [Rancher v2](https://rancher.com), [Amazon ECS](https://aws.amazon.com/ecs), ...) and configures itself automatically and dynamically.
Pointing Traefik at your orchestrator should be the _only_ configuration step you need.

---

. **[Overview](#overview)** .
**[Features](#features)** .
**[Supported backends](#supported-backends)** .
**[Quickstart](#quickstart)** .
**[Web UI](#web-ui)** .
**[Documentation](#documentation)** .

. **[Support](#support)** .
**[Release cycle](#release-cycle)** .
**[Contributing](#contributing)** .
**[Maintainers](#maintainers)** .
**[Credits](#credits)** .

---

:warning: When migrating to a new major version of Traefik, please refer to the [migration guide](https://doc.traefik.io/traefik/migrate/v2-to-v3/) to ensure a smooth transition and to be aware of any breaking changes.


## Overview

Imagine that you have deployed a bunch of microservices with the help of an orchestrator (like Swarm or Kubernetes) or a service registry (like etcd or consul).
Now you want users to access these microservices, and you need a reverse proxy.

Traditional reverse-proxies require that you configure _each_ route that will connect paths and subdomains to _each_ microservice. 
In an environment where you add, remove, kill, upgrade, or scale your services _many_ times a day, the task of keeping the routes up to date becomes tedious. 

**This is when Traefik can help you!**

Traefik listens to your service registry/orchestrator API and instantly generates the routes so your microservices are connected to the outside world -- without further intervention from your part. 

**Run Traefik and let it do the work for you!** 
_(But if you'd rather configure some of your routes manually, Traefik supports that too!)_

![Architecture](docs/content/assets/img/traefik-architecture.png)

## Features

- Continuously updates its configuration (No restarts!)
- Supports multiple load balancing algorithms
- Provides HTTPS to your microservices by leveraging [Let's Encrypt](https://letsencrypt.org) (wildcard certificates support)
- Circuit breakers, retry
- See the magic through its clean web UI
- WebSocket, HTTP/2, gRPC ready
- Provides metrics (Rest, Prometheus, Datadog, Statsd, InfluxDB 2.X)
- Keeps access logs (JSON, CLF)
- Fast
- Exposes a Rest API
- Packaged as a single binary file (made with :heart: with go) and available as an [official](https://hub.docker.com/r/_/traefik/) docker image

## Supported Backends

- [Docker](https://doc.traefik.io/traefik/providers/docker/) / [Swarm mode](https://doc.traefik.io/traefik/providers/docker/)
- [Kubernetes](https://doc.traefik.io/traefik/providers/kubernetes-crd/)
- [ECS](https://doc.traefik.io/traefik/providers/ecs/)
- [File](https://doc.traefik.io/traefik/providers/file/)

## Quickstart

To get your hands on Traefik, you can use the [5-Minute Quickstart](https://doc.traefik.io/traefik/getting-started/quick-start/) in our documentation (you will need Docker).

## Web UI

You can access the simple HTML frontend of Traefik.

![Web UI Providers](docs/content/assets/img/webui-dashboard.png)

## Documentation

You can find the complete documentation of Traefik v3 at [https://doc.traefik.io/traefik/](https://doc.traefik.io/traefik/).

## Support

To get community support, you can:

- join the Traefik community forum: [![Join the chat at https://community.traefik.io/](https://img.shields.io/badge/style-register-green.svg?style=social&label=Discourse)](https://community.traefik.io/)

If you need commercial support, please contact [Traefik.io](https://traefik.io) by mail: <mailto:support@traefik.io>.

## Download

- Grab the latest binary from the [releases](https://github.com/traefik/traefik/releases) page and run it with the [sample configuration file](https://raw.githubusercontent.com/traefik/traefik/master/traefik.sample.toml):

```shell
./traefik --configFile=traefik.toml
```

- Or use the official tiny Docker image and run it with the [sample configuration file](https://raw.githubusercontent.com/traefik/traefik/master/traefik.sample.toml):

```shell
docker run -d -p 8080:8080 -p 80:80 -v $PWD/traefik.toml:/etc/traefik/traefik.toml traefik
```

- Or get the sources:

```shell
git clone https://github.com/traefik/traefik
```

## Introductory Videos

You can find high level and deep dive videos on [videos.traefik.io](https://videos.traefik.io).

## Maintainers

We are strongly promoting a philosophy of openness and sharing, and firmly standing against the elitist closed approach. Being part of the core team should be accessible to anyone who is motivated and want to be part of that journey!
This [document](docs/content/contributing/maintainers-guidelines.md) describes how to be part of the [maintainers' team](docs/content/contributing/maintainers.md) as well as various responsibilities and guidelines for Traefik maintainers.
You can also find more information on our process to review pull requests and manage issues [in this document](https://github.com/traefik/contributors-guide/blob/master/issue_triage.md).

## Contributing

If you'd like to contribute to the project, refer to the [contributing documentation](CONTRIBUTING.md).

Please note that this project is released with a [Contributor Code of Conduct](CODE_OF_CONDUCT.md).
By participating in this project, you agree to abide by its terms.

## Release Cycle

- We usually release 3/4 new versions (e.g. 1.1.0, 1.2.0, 1.3.0) per year.
- Release Candidates are available before the release (e.g. 1.1.0-rc1, 1.1.0-rc2, 1.1.0-rc3, 1.1.0-rc4, before 1.1.0).
- Bug-fixes (e.g. 1.1.1, 1.1.2, 1.2.1, 1.2.3) are released as needed (no additional features are delivered in those versions, bug-fixes only).

Each version is supported until the next one is released (e.g. 1.1.x will be supported until 1.2.0 is out).

We use [Semantic Versioning](https://semver.org/).

## Mailing Lists

- General announcements, new releases: mail at news+subscribe@traefik.io or on [the online viewer](https://groups.google.com/a/traefik.io/forum/#!forum/news).
- Security announcements: mail at security+subscribe@traefik.io or on [the online viewer](https://groups.google.com/a/traefik.io/forum/#!forum/security).

## Credits

Kudos to [Peka](https://www.instagram.com/pierroks/) for his awesome work on the gopher's logo!

The gopher's logo of Traefik is licensed under the Creative Commons 3.0 Attributions license.

The gopher's logo of Traefik was inspired by the gopher stickers made by [Takuya Ueda](https://twitter.com/tenntenn).
The original Go gopher was designed by [Renee French](https://reneefrench.blogspot.com/).
