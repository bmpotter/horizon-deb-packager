# horizon-deb-packager

(formerly `horizon-pkg`)

## Introduction

A project used to create and publish Debian packages of the Horizon platform. Current supported architectures and distributions include:

 * Architectures: amd64, armhf, arm64 ppc64el
 * Distributions: Ubuntu Trusty, Xenial, and Yakkety; Raspbian Jessie, Stretch and Sid; and Debian Jessie and Sid

Related Projects:

 * `anax` (http://github.com/open-horizon/anax): The client control application in the Horizon system
 * `anax-ui` (http://github.com/open-horizon/anax-ui): The source for the Anax web UI
 * `raspbian-image` (http://github.com/open-horizon/raspbian-image): The Raspbian image builder for Raspberry Pi 2 and 3 models dedicated to Horizon

## Manual use

### Publishing new versions using remote builders

Steps:

1. Check out this repository using the `ssh` clone method
2. Update the `VERSION` string in this repository's root directory. Use the following criteria:
    * Increment the micro version (*z* in x.y.z) if there are changes in the source repositores that are limited to bug fixes
    * Increment the minor version (*y* in x.y.z) if source changes only include new features or other non-breaking changes to the system
    * Increment the major version (*x* in x.y.z) if there are any source changes  that change external interfaces or configuration files such that old clients will fail to behave with the new version as they could with older versions and configuration *or* if the behavior of the system changes substantially such that existing users cannot expect the same behavior of the new version as they could from older versions
3. Execute `make meta[-distribution_name]`
4. Review the changes to the local repository as instructed by cli output
5. If satisfied with the proposed changes, execute `make publish-meta`

### Creating a local version

(Shown below is use of the optional cross compilation feature).

    cd build_support/ && make artifacts arch=arm64

This command will build all packages for all distributions supported for the current architecture.

## Running your own build agent

Docker build agent container creation command examples:

    cd build_support/ && make $(make show-flag)

**Note**: You must have appropriate SSH keys added to the agent to: 1) pull code from the repository, and 2) push built packages to the apt signing system.

Docker start command example:

    docker run --rm --name hzn-build --hostname=$(hostname) -v /root/.ssh-aptrepo-signer:/root/.ssh:ro -v /root/go/.cache:/root/go/.cache:rw -v /root/.package_cache:/root/.package_cache:rw -it summit.hovitos.engineering/amd64/horizon-deb-packager-i:latest /bin/bash -c '/prj/build_support/bin/watch-build 1 "/horizon-deb-packager" "http://pkg.bluehorizon.network/linux/##DIST##/dists/##RELEASE##-testing/main" "aptsigner:/incoming" "https://hooks.slack.com/services/..." [restrict_to_distro1,restrict_to_distro2]'

### Manual build with a build agent

In order to execute `make meta` inside a container, you need ssh credentials in-container and/or the ssh agent's authorization socket mounted into the build container. You can execute it this way:

    docker run --rm --name hzn-build-manual -v $PWD:/horizon-deb-packager -it hzn-build /bin/bash

Inside the container, change the `VERSION` and then execute the build steps (constraining the packages to `xenial` in the example below):

    make verbose=y git_repo_prefix=https://github.com/open-horizon skip-precheck=y $(make show-packages | xargs -n1 | grep xenial | xargs)
