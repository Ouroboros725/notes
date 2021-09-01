http://hub.docker.com is the public docker image registry. Here you’ll find both official images as well as images that are uploaded by anyone wishing to share what they’ve built. There’s also provisions for you to upload images you’d like to keep private for a fee. It’s a SaaS platform.

registry.docker.io is actually a CNAME pointing to registry-1.docker.io, which is utilized if you would like to setup a self-hosted registry as a pull-through cache. Say you don’t want multiple engines headed out to grab the same NGINX base image every time there’s a pull…that’s where a cache comes into play.

Here’s more info on the topic Registry as a pull through cache: https://docs.docker.com/registry/recipes/mirror/#configure-the-cache

---

http://hub.docker.com is the DNS name for Docker Hub - the web UI for Docker’s container image registry for either public or private container images.

registry.docker.io does indeed exist; it is a DNS alias to registry-1.docker.io. This is where the actual registry that images are pulled from. When you issue a command to download an image from Docker Hub (say docker pull ubuntu:latest), the command makes an API call to registry-1 to initiate the download (list the layers and then download each layer).