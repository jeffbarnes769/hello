# Docker Images

The Go blog <https://blog.golang.org/docker> and 'Hello World' <https://golang.org/doc/install> are great introductions to Docker and Go.

FROM **golang:onbuild** make building images simple, but produces large images containing vulnerabilities.  Vulnerability scanning, policy checking through container security brokers and multi-stage docker builds, help address image challenges.

This example uses hosted AquaSec in an Azure tenant and scanner-cli both from the command line and Jenkins.  Image scanning is now included in many CSP image repositories and opensource, such as https://github.com/aquasecurity/microscanner.

Running scanner-cli against an AquaSec SaaS tenant, containing corporate polices, produces https://jeffbarnes769.github.io/files/hello0.html
```
$ docker run --rm -v /var/run/docker.sock:/var/run/docker.sock --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/scanner-cli:3.0.1 scan --user username --password password --host https://tenantx-saas.aquasec.com --register --local --registry remote hello:v0 --html >> hello0.html
```
Simple ‘hello world’ built using onbuild is over 700MB, with vulnerabilities:
```
$ docker build -t hello:v0 .
```
<img src="img/onbuild.jpg" width="600">

We then build various images by modifying the Dockerfile and running docker build
```
$ docker run --rm -v /var/run/docker.sock:/var/run/docker.sock --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/scanner-cli:3.0.1 scan --user username --password password --host https://tenantx-saas.aquasec.com --register --local --registry remote hello:v0
```
Hello:v0 has fewer vulnerabilities (but still too many) and is now nearly 800MB.  As well our CSB policy is to not run containers as root
```
$ docker run --rm -v /var/run/docker.sock:/var/run/docker.sock --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/scanner-cli:3.0.1 scan --user username --password password --host https://tenantx-saas.aquasec.com --register --local --registry remote hello:v0
```
<img src="img/hello0.jpg" width="600">

Hello:v2 passes our scan but at 378MB is still larger than necessary
```
$ docker run --rm -v /var/run/docker.sock:/var/run/docker.sock --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/scanner-cli:3.0.1 scan --user username --password password --host https://tenantx-saas.aquasec.com --register --local --registry remote hello:v2
```
<img src="img/hello2.jpg" width="400">

<img src="img/repo.jpg" width="200">

# Multi-Stage Docker Builds

Modify the Dockerfile for multi-stage docker build to reduce image size and vulnerabilities, with no change to 'hello world'.
```
$ docker build -t hello:v2 .
```
Output from the AquaSec scanner-cli
```
$ docker run --rm -v /var/run/docker.sock:/var/run/docker.sock --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/scanner-cli:3.0.1 scan --user username --password password --host https://tenantx-saas.aquasec.com --register --local --registry remote hello:v2
```
<img src="img/hello3.jpg" width="400">

# Running from Jenkins
The AquaSec scanner-cli can also be run from Jenkins using the AquaSec plugin.  Retrieve and docker load scanner-cli:
```
$ docker load -i aquasec-scanner-cli-2.5.3.tar
```
Add to Jenkins config: 

<img src="img/aqua3.jpg" width="600">

Run during image build:

<img src="img/aqua2.jpg" width="600">

Resources for securing containers, such as the CIS Benchmark <https://www.cisecurity.org/benchmark/docker/>, Understanding and Hardening Linux Containers <https://www.nccgroup.trust/us/our-research/understanding-and-hardening-linux-containers> and others
