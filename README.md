# docker-credential-gcr-cached

A tiny one-off POSIX script to intercept and speed-up Google Container Registry (GCR) credentials requests, which may have a vast cascading performance impact on docker build initialization.

# The Problem

Based on the author's syscall analysis of docker builds (version 17.12.0-ce), it seems that multiple requests are made in a serial fashion to the `credHelpers` entries in the `$HOME/.docker/config.json` file.  Even if "gcr" is used in place of "gcloud", build operations can be delayed by more than ten seconds (!) due to the many un-cached blocking requests made to retrieve credentials for gcr.io and other regional cnames.  A lot of gains can be made by removing the regional cname entries (anything not "gcr.io") from the docker configuration, but there is still a noticeable multi-second delay.  That's where this script comes in handy!

# A Solution

This script acts as a proxy for `docker-credential-gcr` credential-helper, intercepting `get` requests and returning the token found in `$HOME/.gcloud/docker_credentials.json` if it is not expired.  This results in order-of-magnitude docker build startup time performance, as long as your environment allows the gcloud active account token to be used.

# Requirements

- You are using Docker with Google Container Registry, and you can use your active gcloud account for pulling/pushing docker images to GCR.
- [jq](https://stedolan.github.io/jq/) must be installed and available on your environment PATH.

# Installation

1. Clone the repo:

    ```
    git clone git@github.com:ccarpita/docker-credential-gcr-cached
    ```

2. Add the repository path to your shell's `PATH` variable.

3. Change "gcr" to "gcr-cached" in your `$HOME/.docker/config.json` file.

4. Enjoy faster docker builds!
