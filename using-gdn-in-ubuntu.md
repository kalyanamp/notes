# How to use the `gdn` container engine in Ubuntu 16.04

## What is `gdn`

`gdn` is the standalone version of [Garden
runC](https://github.com/cloudfoundry/garden-runc-release), the container
engine behind the popular [Cloud Foundry](https://www.cloudfoundry.org/) PaaS.

Garden runC has been powering public Cloud Foundry installations since early
2016. However, the standalone version is quite new and still very much work in
progress.

It is preferred to use `gdn` on an Ubuntu 16.04 machine since this short guide
is only tested on that platform.

In case you run into deployment/usage issues, feel free to reach out to the
`#garden` channel of the [Cloud Foundry](http://slack.cloudfoundry.org/) Slack
for support.

## What you will need

You will need to download the latest `gdn` binary from
https://github.com/cloudfoundry/garden-runc-release/releases, that is
[1.2.0](https://github.com/cloudfoundry/garden-runc-release/releases/download/v1.2.0/gdn-1.2.0)
at the time of writing.

You will also need to download the Garden CLI, `gaol` from
https://github.com/contraband/gaol/releases, that is
[2016-8-22](https://github.com/contraband/gaol/releases/download/2016-8-22/gaol_linux)
at the time of writing.

## How to start `gdn`

We recommend running the following commands in a VM that you can later throw
away. In an Ubuntu 16.04 machine, run the following as user `root`:

```bash
# download gdn
curl https://github.com/cloudfoundry/garden-runc-release/releases/download/v1.2.0/gdn-1.2.0 -o gaol

# prepare your system for gdn
./gdn setup

# start gdn
./gdn server --bind-port 7777 --bind-ip 0.0.0.0
```

## How to use `gdn`

In the same machine, run the following as any user:

```bash
# download the gdn CLI, gaol
curl https://github.com/contraband/gaol/releases/download/2016-8-22/gaol_linux -o gaol

# create a container
./gaol create -n test -r docker:///busybox
# this is a container based on the Busybox Docker image, which will be
#   downloaded from DockerHub.

# run commands inside the container
./gaol run -a -c "ls -la" test
# replace the contents of the -c flag with the command you want to run

# delete a container
./goal destroy test
```
