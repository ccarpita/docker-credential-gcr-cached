# docker-credential-gcr-cached

A small POSIX script that provides alternative fast "get" queries for Google Container Registry (GCR) credentials requests, resulting in a cascading large performance impact on docker build initialization.

# The Problem

Based on the author's syscall analysis of docker builds (version 17.12.0-ce), it seems that multiple requests are made in a serial fashion to all of the `credHelpers` entries in the `$HOME/.docker/config.json` file, including domains that will not end up being used.  One improvement is to use "gcr" in place of "gcloud", as the former is a compiled binary.  However, the docker build operations can be delayed by more than an eye-popping ten seconds due to the many repeated requests made to retrieve credentials for gcr.io.

Significant improvements can be made by removing the regional cname entries from the docker configuration, but there is still a noticeable 1-2 second delay even with the single gcr.io entry in place.  This script can help remove this delay by using a simple file cache.

# A Solution

This script acts as a proxy for `docker-credential-gcr` credential-helper, intercepting `get` requests and returning the `.gcr-token` cached token if the file is valid and less than an hour old.  Otherwise, the script will perform a manual token refresh using `gcloud config`, and then subsequent requests will be cached for an hour.  The end result is a 10-100x improvement in startup performance for docker build operations.

# Requirements

- You are using Docker with Google Container Registry, and you can use your active gcloud account for pulling/pushing docker images to GCR.
- You have `gcloud` CLI tools installed and available on your environment PATH.

# Installation

1. Clone the repo:

    ```
    git clone git@github.com:ccarpita/docker-credential-gcr-cached
    ```

2. Add the repository path to your shell's `PATH` variable.

3. Change "gcr" to "gcr-cached" in your `$HOME/.docker/config.json` file.

4. Enjoy faster docker builds!


# Usage

If you have permissions issues when running docker builds, this could result from having the wrong gcloud account enabled during the first time the script is run.  If you think this has occurred, you can try setting `GCR_AUTH_REFRESH=1` before your `docker` command to force refresh of the token.

Example:

```
GCR_AUTH_REFRESH=1 docker build -f Dockerfile .
```
