https://github.com/coreos/docs/blob/master/os/registry-authentication.md

## Using a Quay robot for registry auth

The recommended way to authenticate container manager software with [quay.io][quay-site] is via a [Quay Robot][quay-robot]. The robot account acts as an authentication token with some nice features, including:

* Readymade repository authentication configuration files
* Credentials are limited to specific repositories
* Choose from read, write, or admin privileges
* Token regeneration

![Quay Robot settings][quay-bot-img]

Quay robots provide config files for Kubernetes, Docker, Mesos, and rkt, along with intructions for using each. Find this information in the **Robot Accounts** tab under your Quay user settings. For more information, see the [Quay robot documentation][quay-robot].