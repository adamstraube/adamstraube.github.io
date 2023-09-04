---
layout: post
title: Using gitlab-runner locally with Docker in Docker on Windows 10 and WSL
date: 2020-01-18 00:04 +1100
comments: true
tags: Bash Docker Gitlab WSL
---

### Environment:
- Windows 10
- WSL (Windows Subsystem for Linux) with Ubuntu 18.04 installed
- Docker Toolbox for Windows
- Docker installed on WSL1

### Quickstart - Bash script that does it for you

Keep scrolling below for a step by step and instructions.
If you need to get up and running fast then I have created a bash script that handles the starting and running of the containers. [You can download it from here](https://gist.github.com/adamstraube/b5c8eae3034f3d8d5561cfb143751d7e)

#### Features:
- Run single or multiple stages at once
- Retains cache when running multiple stages

#### Commands to get you up and running:
Change into your project/working directory in WSL (directory must be accessible to Windows, not only to the WSL container)

```shell
> rungit.sh up
```

Start gitlab-runner container

```shell
> rungit.sh runStages {stage(s)}
```

Run gitlab-runner a specified stage or group of stages in your .gitlab-ci.yml. Multiple stages should be comma separated without spaces eg. `rungit.sh runStages build,test`. Giving no stage will run the`test` stage by default.

```shell
> rungit.sh down
```

Stop the gitlab-runner container and remove

### Overview and caveats:

Running gitlab-runner locally means utilising the `exec` option. Looking into this I came across some issues:
- Running locally means you can only run one stage at a time (eg. stage, test, build)
- The `exec` functionality may be [deprecated/removed sometime in the future](https://gitlab.com/gitlab-org/gitlab-runner/issues/2710) but a [replacement function will likely be implemented before this happens](https://gitlab.com/gitlab-org/gitlab-runner/issues/2797)
- Running gitlab-runner directly in Windows [has path resolution issues](https://gitlab.com/gitlab-org/gitlab-runner/issues/3809)


### Step 1: Create and start a gitlab-runner docker instance

It is easier to keep gitlab-runner up to date and be sure it has everything it needs by running this in a container.

Assuming you are in your working directory and most importantly it is accessible via Windows, run the following command to start the Gitlab-runner container:

```
docker run -d --name gitlab-runner --restart always \
                  -v $(pwd -P):$(pwd -P) \
                  -v /var/run/docker.sock:/var/run/docker.sock \
                  --workdir $(pwd -P) \
                  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
                  gitlab/gitlab-runner:latest
```

The reason we use `pwd -P` is due to how the variance in directory structure between WSL, Docker Desktop and Windows. When you run a docker command in WSL with a host mount that feeds to Docker toolbox for Windows then it will try and translate the location. This will generally work but you will run into issues with this if:
- You use symlinks to your working directory in WSL
- You utilise Docker in Docker and try to pass volume bindings down the nested containers

By passing the full host path through all the nested containers we ensure we are in the correct directory. The reason is that even within Docker in Docker, when we talk to Docker we are talking to the daemon on the host (Docker toolbox). When we pass a host directory location it will check on the host machine (after translation from WSL). This means mounting a directory from one container into another will not work.


### Step 2: Test a stage from your .gitlab-ci.yml configuration

To test a stage within your .gitlab-ci.yml run:

```
		docker exec -w $(pwd -P) -it gitlab-runner \
 		  gitlab-runner exec docker test \
		  --docker-privileged \
		  --docker-volumes '/var/run/docker.sock:/var/run/docker.sock' \
		  --env ROOT_PWD=$PWD_RESOLVED
```

This will run the test stage. To run another stage change `test` to something else.

### Script

Congratulations for reading this far.
I have written a shell script that makes this simpler. Check out the Quickstart heading above and here is a [link to the script](https://gist.github.com/adamstraube/b5c8eae3034f3d8d5561cfb143751d7e)

