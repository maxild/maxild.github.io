+++
title = "Private Github submodules"
date = 2021-02-06
[taxonomies]
tags = ["git"]
+++

Some workflows require use of an SSH private key during build - for instance
when getting sources for private submodules during a build.

## The quick rundown (TL;DR)

- create SSH keypair on your computer
- add public part as user key on Github
- add private key as secret in (container) repo where action is triggered (such that `$HOME/.ssh/id_rsa` can be synthesized as a preliminary build step)

**Github Actions**:
```yaml
env:
  SSH_SUBMODULES_KEY: ${{ secrets.SSH_SUBMODULES_KEY }}
```

**AppVeyor**
```yaml
env:
  SSH_SUBMODULES_KEY:
    secure: DF3lCBl.......G5+p/ # NOTE: Very long!!!
```

- do the following before pulling the submodules in the run configuration

```bash
mkdir -p "$HOME/.ssh" && \
echo "$SSH_SUBMODULES_KEY" > "$HOME/.ssh/id_rsa" && \
chmod 600 "$HOME/.ssh/id_rsa"
```

<!-- more -->

## Intro

In my normal workflow I use HTTPS to access Github repos, but when adding submodules
via HTTPS the repo has to be public. The below submodule configuration does not work
on AppVeyor or Github Actions, because `maxild/Lofus` is accessed via HTTPS

```ini
[submodule "src/submodules/aspnetcore"]
    path = src/submodules/aspnetcore
    url = https://github.com/dotnet/aspnetcore.git
    branch = master
[submodule "src/submodules/Lofus"]
    path = src/submodules/Lofus
    url = https://github.com/maxild/Lofus.git
    branch = dev
```

First step is to use SSH for the private submodule repo:

```ini
[submodule "src/submodules/aspnetcore"]
    path = src/submodules/aspnetcore
    url = https://github.com/maxild/aspnetcore.git
    branch = internals-Visible-To
[submodule "src/submodules/Lofus"]
    path = src/submodules/Lofus
    url = git@github.com:maxild/Lofus.git
    branch = dev
```

> Tip: Public submodules (dotnet/aspnetcore) should use HTTPS, and private submodules (maxild/Lofus) should use SSH.

> Tip: Submodule repositories linked via SSH (i.e. all private submodule repositories) require an SSH key called a `deploy/user key` [in GitHub](https://docs.github.com/en/developers/overview/managing-deploy-keys#deploy-keys) or [in GitLab](https://docs.gitlab.com/ee/ssh/#per-repository-deploy-keys) or an `access key` [in Bitbucket](https://confluence.atlassian.com/bitbucketserver/ssh-access-keys-for-system-use-776639781.html).

## Create SSH public/private keypair

First, create an ssh key and add the public key (shorter one) to your service (e.g. on your GitHub repo settings, or in ~/.ssh/authorized_keys on your server). [Run the `ssh-keygen` in your terminal](https://docs.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key)

```bash
ssh-keygen -t rsa
```

I choose to create `submodules` and `submodules.pub` files (when the tool asks for a filename, and use an empty passphrase).

## Add the public key to your account settings on Github (`user key`, _not_ `deploy key`)

In the [user account settings page](https://github.com/settings/keys) on GitHub add a new SSH key. Use the content of the `submodules.pub` file.

It is the following (public, non-secret) content

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDiAj88GBUtxT0seyYZ7xXUZmYShD/ISwjByYjmp+S7fx7sqA2ws5yLHyueYksCBS1tZOo+jyBAJxzpEGiPirLnMvZwK0BDdwJNQ2vWv6JCCSnUtb8pfckXmTGOxDbhkNZNdm7lIVVOm0W2odyVkIqGB2VF3MLnqQqAcaXtu/iOk91AimYoVV8IKu7WCrXqNqNLqOrlPBvqg9Ys82En64FryDDhmRx+B7j+qvX11TWlqQ3/Cxp9ejULn8reZQrNGlEI37gbPxfgHl0+dI27jdiUwkpjTcfSyT6DV8agrVh1m6DVMv8sglzmLqQ3OscWB5RMXuyyJ//FfIADrZqrg70qj1nHzC5ddCgXpZThIVtXpdFIoQUKtqMMmchcuQGrr6GjURZ0UMsYfvaxzTpejulOs6Qd6jiZoSl0tdUUqfQQmcFAb9rvsmJMzA2U05yuK8Cwd2CdxLiIForQha3wZYtvzYj+z2mw5XAgQypIxC/DdRmkruwpsl9eZWWWx6uqgV0= maxfire@pop-os
```

NOTE: It is much smaller than the private key.

## Add the private key as a secret to your CI provider (Github or AppVeyor)

The CI build is executing git to not only get the main (container) repo, but also to recursively get all submodules. This does
not work out of the box on either Github or AppVeyor. We have to configure the SSH infrastructure

### On Github

Copy the private key to a secret named "SSH_SUBMODULES_KEY". It is the following content that must be copied to the value of the secret

```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
.
.
.
.
SIpPw+HALdX737AAAADm1heGZpcmVAcG9wLW9zAQIDBA==
-----END OPENSSH PRIVATE KEY-----
```

### On AppVeyor

Using the "Encrypt YAML" page generate an encrypted environment variable called "SSH_SUBMODULES_KEY" and paste it into `appveyor.yml`. See this [guide](https://www.appveyor.com/docs/how-to/private-git-sub-modules/) for more information.

## Setup the CI YAML Pipeline

After having setup the secret/private keys on the CI cloud providers, its time to use the secrets to setup the CI pipeline/environment:

### On Github

Here we use Windows and Linux (MacOS is way too expensive, and who will use that in production?)

```
- name: Install SSH key on Linux/Windows (via GIT bash shell on win)
  env:
    SSH_SUBMODULES_KEY: ${{ secrets.SSH_SUBMODULES_KEY }}
  run: |
    mkdir -p ~/.ssh && chmod 700 ~/.ssh
    # See https://linuxize.com/post/using-the-ssh-config-file/
    (cat << EOF
    Host github.com
      User git
      Hostname github.com
      PreferredAuthentications publickey
      IdentityFile $HOME/.ssh/id_rsa
    EOF
    ) > ~/.ssh/config
    chmod 600 ~/.ssh/config
    echo "${SSH_SUBMODULES_KEY//_/\\n}" > "$HOME/.ssh/id_rsa"
    chmod 600 "$HOME/.ssh/id_rsa"
    SSH_KNOWN_HOSTS="$(ssh-keyscan -t rsa github.com)"
    echo "$SSH_KNOWN_HOSTS" > "$HOME/.ssh/known_hosts"
  shell: bash
```

### On AppVeyor

Here we use Windows (`image: `)

```yaml
install:
  - ps: $fileContent = "-----BEGIN OPENSSH PRIVATE KEY-----`n"
  - ps: $fileContent += $env:SSH_SUBMODULES_KEY.Replace(' ', "`n")
  - ps: $fileContent += "`n-----END OPENSSH PRIVATE KEY-----`n"
  - ps: Set-Content $env:userprofile\.ssh\id_rsa $fileContent
```

## SSH keys

SSH keys on Github can be either repo specific (aka deploy keys) or user specific (aka user keys):

Authentication | Protocol | Dependency URL format | Gives access to	| Notes
|---|---|---|---|---|
Deploy Key | SSH | `git@github.com:<owner>/<repo>.git` | single repository | used by default for main repository
User Key | SSH | `git@github.com:<owner>/<repo>.git` | all repos user has access to | recommended for dependencies

## Deploy Key

GitHub allows you to set up SSH keys for a repository. These deploy keys have some great advantages:

- They are not bound to a user account, so they will not get invalidated by removing users from a repository.
- They do not give access to other, unrelated repositories.
- The same key can be used for dependencies not stored on GitHub.

However, using deploy keys is complicated by the fact that GitHub does not allow you to reuse keys. So a single private key cannot access multiple GitHub repositories.

You could include a different private key for every dependency in the repository, possibly encrypting them. Maintaining complex dependency graphs this way can be complex and hard to maintain. For that reason, we recommend using a user key instead.

## User Key (preferred IMO)

You can add SSH keys to user accounts on GitHub. Most users have probably already done this to be able to clone the repositories locally.

This way, a single key can access multiple repositories. To limit the list of repositories and type of access, it is recommended to create a dedicated CI user account.

