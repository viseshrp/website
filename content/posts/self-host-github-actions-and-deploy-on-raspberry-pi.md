---
title: "Self-host Github Actions and deploy on Raspberry Pi"
date: 2023-01-25T09:33:56-05:00
tags: [blog, github, actions, self-host, pi, website, rsync, systemd, systemctl]
draft: false
---

Following are the notes for automating the deployment of this blog on my Raspberry Pi.

After exploring different options, I considered `rsync`-ing over the source code
after some [basic minification](https://github.com/viseshrp/website/blob/main/.github/workflows/publish.yml#L9)
through GitHub Actions with Hugo. This required opening
 up ports on my router and port-forwarding. A couple of ports were already opened up for
HTTP/HTTPS (to self-host this blog) and I did not want to expose any more of them. Definitely not for
`rsync` or `ssh`.

Enter [self-hosted](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners) GitHub Actions.
This is a really nice way to extend the power of Actions and lets you run jobs on your own
machine, avoiding any throttling or wait time. You install the [runner](https://github.com/actions/runner) application
and that listens to GitHub for jobs. When jobs are available, the source is pulled automatically
and actions are run on your server, so all this needs is an internet connection. The runner also auto-updates itself to the latest version whenever a job is assigned.

There are multiple packages for different operating systems and CPU architectures, but I chose Linux/ARM64 for
my Pi running Raspberry Pi OS. All you need to do is go to *your-github-repo-url/settings/actions/runners/new*
and follow the instructions. So, for this website which is open source, it would be *github.com/viseshrp/website/settings/actions/runners/new*.
You could also go to your repo and then: *Settings -> Actions -> Runners -> New self-hosted runner*.

Be aware you may not be able to install or run this application directly as the root user but will have to run `sudo`
actions as a different user.

`./run.sh` will start and run the runner app immediately but if you want to run it as a daemon, you must install the systemd service and start it.

```shell
sudo ./svc.sh install
sudo ./svc.sh start
```

`sudo` is necessary here because it will create a *systemd* controlled service
for you and run it as a daemon. This should print out the name and/or path of the
service which should be something like `actions.runner.user-repo.hostname.service`.

Mine looked something like below, and I enabled it so that it started itself if
I ever restarted my Pi.

```shell
sudo systemctl status actions.runner.viseshrp-website.rpiweb.service
sudo systemctl enable actions.runner.viseshrp-website.rpiweb.service
```

If you check the runners page again, it must show up as connected and idle.
Now, you just have to include it with the `runs-on: self-hosted` directive in your Actions YAML.

My runner and this website run on the same Pi so the hack I made was to include a local `rsync` [command that copied the code](https://github.com/viseshrp/website/blob/ce3423523812e007b1f02a5c3e435aa4c251004d/.github/workflows/publish.yml#L30) over
to the blog's installation path on the server once the build was complete.

Now, every time I push something to my repo, it will auto-trigger a build through Actions on my Pi
and in the end copy the artifacts over. The website is served through a [NGINX Docker container](https://github.com/viseshrp/website/tree/main/nginx) that
runs on the same host and since the source is just static files, no restarts of the container or NGINX are required.
Changes will immediately go live.

#### Update (2/2/23)

Encountered a problem when I was setting up the runner again from scratch.

```bash
Error: Input 'submodules' not supported when falling back to download using the GitHub REST API
```

Reinstalling `apt install git` solved it for me.

#### Update (9/26/23)

This has now been made a lot easier through a Docker image. Checkout my compose file [here](https://github.com/viseshrp/homelab/blob/main/docker-compose/gh-runner/docker-compose.yml), add your secrets and just do: `docker compose up -d`.
