Container Development Guide
===========================

This document contains instructions for OSG Technology Team members, including:

- How to to develop container images that adhere to our [container release policy](/policy/container-release)
- How to build a new version of an existing image
- How to manage tags for images in the [OSG DockerHub organization](https://hub.docker.com/r/opensciencegrid/)
- Tips for container image development

Creating a New Container Image
------------------------------

### Prepare the GitHub repository ###

1. Create a Git repository whose name is prefixed with `docker-`, e.g. `docker-frontier-squid`
1. Create a `README.md` file describing the software provided by the image
1. Create a `LICENSE` file containing the [Apache 2.0 license text](https://www.apache.org/licenses/LICENSE-2.0.txt)
1. Create a `Dockerfile` based off of the OSG Software Base image:

        FROM opensciencegrid/software-base:fresh

        LABEL maintainer OSG Software <help@opensciencegrid.org>
        
        RUN yum update -y && \
            yum clean all && \
            rm -rf /var/cache/yum/*

        RUN yum install -y <PACKAGE> --enablerepo=osg-minefield && \
            yum clean all && \
            rm -rf /var/cache/yum/*


    Replacing `<PACKAGE>` with the name of the RPM you'd like to provide in this container image

### Prepare the DockerHub repository ###

1. Create a Docker Hub repo in the OSG organization.
1. Go to the permissions tab and give the `robots` and `technology` teams `Read & Write` access
1. Copy over `.travis.yml` and `travis/` from a previous docker build (e.g., <https://github.com/opensciencegrid/docker-frontier-squid>).
   Update the `docker_repos` in `build_docker.sh` to the name of the Docker Hub repo
1. Set the `+x` bit on `travis/build_docker.sh`
1. Send the `opensciencegrid` GitHub repository URL to the Software Manager and ask them to do the following:
    1. Enable the repository in Travis-CI
    1. Add the OSG Software Docker Hub account credentials to the Travis CI repository as secure variables
       `DOCKER_USERNAME` and `DOCKER_PASSWORD` only available to the `master` branch.
       Ensure that `Display value in build log` remains unset.
       Escape any special characters with `\`.
    1. Enable weekly cron builds from `master` and set `Always run`

Triggering Container Image Builds
---------------------------------

To build a new version of an [existing container image](#creating-a-new-container-image), e.g. for a new RPM version of
software in the container, you can kick off a new build in one of two ways:

- **If there are no changes necessary to the container packaging:** go to the repository's latest Travis-CI build off of
  `master`, e.g. <https://travis-ci.org/opensciencegrid/docker-xcache>, and click "Restart build"
- **If changes need to be made to the container packaging:** submit a pull request with your changes to the relevant
  GitHub repository and request that another team member review it.
  Once merged into `master`, a Travis-CI build should start automatically.

If the Travis-CI build completes successfully, you should shortly see new `fresh` and timestamp tags appear in the
DockerHub repository.

!!! info "Automatic weekly rebuilds"
    If the repo's Travis-CI is configured as above, container images will automatically rebuild, and therefore pick up
    new packages available in minefield once per week.

Managing Tags in DockerHub
--------------------------

### Adding tags ###

Images that have passed acceptance testing should be tagged as `stable`:

1. Install the `jq` utility:

        yum install jq
        
1. Get the sha256 repo digest of the image that the user has tested.
   All you need is the part that starts with `sha256:...` (aka the `<DIGEST>`).

1. (Optional) If you are tagging multiple images, you can enter your Docker Hub username and password into environment
   variables, to avoid having to re-type them.
   Otherwise the script will prompt for them.

        read user     # enter dockerhub username
        read -s pass  # enter dockerhub password
        export user pass

1. Run the Docker container image tagging command from [release-tools](https://github.com/opensciencegrid/release-tools/):

        ./dockerhub-tag-fresh-to-stable.sh <IMAGE NAME> <DIGEST>

### Removing tags ###

Run the Docker container image pruning command from [release-tools](https://github.com/opensciencegrid/release-tools/):

    ./dockerhub-prune-tags.py <IMAGE NAME>

Making Slim Containers
----------------------

Here are some resources for creating slim, efficient containers:

- <https://developers.redhat.com/blog/2016/03/09/more-about-docker-images-size/>
- <https://github.com/opensciencegrid/topology/pull/399>
- <https://docs.docker.com/develop/develop-images/multistage-build/>