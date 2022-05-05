# actions

### kube-do-action

[![test-kube](https://github.com/hckops/actions/actions/workflows/test-kube.yml/badge.svg)](https://github.com/hckops/actions/actions/workflows/test-kube.yml)

> Manage DigitalOcean Kubernetes cluster lifecycle

Create or delete clusters based on a config definition
```diff
# examples/kube-do-sample.yaml
version: 1
name: do-sample
provider: digitalocean
+ status: UP
- status: DOWN

config:
  region: lon1
  size: s-1vcpu-2gb
  count: 1
```

For a working example see [kube-template](https://github.com/hckops/kube-template/blob/main/.github/workflows/kube-do.yml)
```bash
- name: Provision
  uses: hckops/actions/kube-do-action@main
  with:
    access-token: ${{ secrets.DIGITALOCEAN_ACCESS_TOKEN }}
    config-path: examples/kube-do-sample.yaml
    wait: true
```

Requires `DIGITALOCEAN_ACCESS_TOKEN` secret
* [How to Create a Personal Access Token](https://docs.digitalocean.com/reference/api/create-personal-access-token)
* Create a [Personal Access Token](https://cloud.digitalocean.com/account/api/tokens)

How to test it locally
```bash
# build image
docker build -t hckops/kube-do-action ./kube-do-action

# run action
docker run --rm \
  -e GITHUB_REPOSITORY="INVALID_GITHUB_REPOSITORY" \
  hckops/kube-do-action \
  "INVALID_GITHUB_TOKEN" "INVALID_ACCESS_TOKEN" "./examples/kube-do-sample.yaml" "false" "false" "false"
```

TODOs
- [ ] make it properly configurable: cluster definition `ClusterConfig`
- [ ] make it schedulable: run action to reconcile cluster drift status
- [ ] try to remove `github-token` from inputs
- [ ] implementation: shell vs ???

### bootstrap-action

[![test-bootstrap](https://github.com/hckops/actions/actions/workflows/test-bootstrap.yml/badge.svg)](https://github.com/hckops/actions/actions/workflows/test-bootstrap.yml)

> TODO

```bash
TODO
```

Requires
* `GITOPS_SSH_KEY` secret
    - [Generate a new SSH key pair](https://help.github.com/en/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key)
    ```bash
    # generate ssh key pair
    ssh-keygen -t rsa -b 4096 -C "gitops@example.com" -N '' -f /tmp/id_rsa_gitops

    # add public key to a github user account with access to the repo
    cat /tmp/id_rsa_gitops.pub | xclip -selection clipboard

    # create secret with private key
    cat /tmp/id_rsa_gitops | xclip -selection clipboard

    # cleanup
    rm /tmp/id_rsa_gitops*
    ```
* `ARGOCD_ADMIN_PASSWORD` secret
    - [User Management](https://argo-cd.readthedocs.io/en/stable/operator-manual/user-management)
    - [How to change admin password](https://argo-cd.readthedocs.io/en/stable/faq/#i-forgot-the-admin-password-how-do-i-reset-it)
    - Example to ***do NOT use it in production***
        * user `admin`
        * password `argocd`
        * secret `$2a$04$qj3hWU1Id.l.4e/8JN4Kr.ecQDuf3hhyG0TbsLeDcZV2kRG/AizY2`

How to test it locally
```bash
docker build -t hckops/bootstrap-action ./bootstrap-action
# TODO
docker run --rm hckops/bootstrap-action
```

### discord-action

[![test-discord](https://github.com/hckops/actions/actions/workflows/test-discord.yml/badge.svg)](https://github.com/hckops/actions/actions/workflows/test-discord.yml)

> Interact with Discord API

*Create message*
```bash
- name: Notification
  uses: hckops/actions/discord-action@main
  with:
    action: create-message
    webhook-url: ${{ secrets.DISCORD_WEBHOOK_URL }}
    message: "Hello World"
```

Requires `DISCORD_WEBHOOK_URL` secret
* [Intro to Webhooks](https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks)

How to test it locally
```bash
DISCORD_WEBHOOK_URL="INVALID_URL"
make discord-create webhook=${DISCORD_WEBHOOK_URL} message=test

docker build -t hckops/discord-action ./discord-action
docker run --rm hckops/discord-action "create-message" ${DISCORD_WEBHOOK_URL} "docker"
```

## Development

### Docker images

[![docker-ci](https://github.com/hckops/actions/actions/workflows/docker-ci.yml/badge.svg)](https://github.com/hckops/actions/actions/workflows/docker-ci.yml)

> Actions base images

* [DockerHub](https://hub.docker.com/u/hckops)

```bash
# run command
docker run --rm hckops/kube-base /bin/bash -c <kubectl|helm|argocd>
# start temporary container
docker run --rm --name hck-kube -it hckops/kube-<base|aws|do>
```

How to build and publish images manually
```bash
make docker-build
make docker-publish version=v0.1.0 token=<ACCESS_TOKEN>
make docker-clean
```
