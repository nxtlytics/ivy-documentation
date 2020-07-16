# 1. Software Engineering On Boarding

If you are just coming onboard there are a number of things you will want to do and learn to better understand how to work within Ivy.

# Table of Contents

-   1 [Software Development On Boarding](#id-1.SoftwareEngineeringOnBoarding-SoftwareDevelopmentOnBoarding)
-   2 [Common Tools](#id-1.SoftwareEngineeringOnBoarding-CommonTools)

# Children

-   [New Developer Setup](New_Developer_Setup)
-   [VPN Setup (Mac OS)](VPN_Setup_Mac_OS_)
-   [VPN Troubleshooting](VPN_Troubleshooting)


# Software Development On Boarding

## Homebrew

[Homebrew](https://brew.sh/) is a package manager, the following
software development tools will require brew to install them correctly.
Follow the direction on their site to install homebrew onto your
computer.

To ensure everything is installed as expected run the following and make
sure there are no errors:

```
brew doctor
```

## SSH

GitHub will require an ssh key pair. To generate and use the key pair do
the following in a terminal.

``` bash
# NOTE: be sure to replace <G Suite email> in the following command.
ssh-keygen -t ed25519 -C "<user>@example.com"
eval "$(ssh-agent -s)" > /dev/null
echo $'\neval "$(ssh-agent -s)" > /dev/null\n' >> ~/.bash_profile
echo "$(
  cat <<EOF
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519

EOF
  )" >> ~/.ssh/config
```

## Git

OSX comes with git installed, but upgrading to the latest is a good idea.

``` bash
brew install git
git config --global url.ssh:url.ssh://git@github.com/.insteadOf https://github.com/
git config --global user.name '<first_name> <last_name>'
git config --global user.email '<user>@example.com'
```

## GPG

GPG is a tool to cryptographically sign and encrypt data. We use it for
git and GitHub to sign commits and tags with your unique cryptographic
signature which helps ensure we know who is committing code.

Setup GPG on your box with:

``` bash
brew install gpg
gpg --full-generate-key
# Select RSA/RSA
# Choose a 4096 bit key
# User your G Suite email address and real name in the key.

# NOTE: be sure to replace <G Suite email> in the following command.
GPG_KEY="$(gpg --list-secret-keys | grep -B 1 '<G Suite email>' | head -n 1 | tr -d '[[:space:]]')" && [ -z "${GPG_KEY}" ] && echo $'\n#####\nFAILURE\n#####\n' || echo $'\n#####\nSUCCESS\n#####'

# Check to make sure the previous result does not indicate a failure before proceeding
git config --global user.signingkey "${GPG_KEY}"
git config --global commit.gpgsign true
git config --global tag.gpgsign true
```

The following will make commits done through the web interface in GitHub
trusted signers as well. Adding this key will clean up git log entries
if you enable displaying signatures.

``` bash
curl https://github.com/web-flow.gpg | gpg --import
```

This is a copy of githubs PGP public key retrieved on 2019-09-05:

``` java
-----BEGIN PGP PUBLIC KEY BLOCK-----

xsBNBFmUaEEBCACzXTDt6ZnyaVtueZASBzgnAmK13q9Urgch+sKYeIhdymjuMQta
x15OklctmrZtqre5kwPUosG3/B2/ikuPYElcHgGPL4uL5Em6S5C/oozfkYzhwRrT
SQzvYjsE4I34To4UdE9KA97wrQjGoz2Bx72WDLyWwctD3DKQtYeHXswXXtXwKfjQ
7Fy4+Bf5IPh76dA8NJ6UtjjLIDlKqdxLW4atHe6xWFaJ+XdLUtsAroZcXBeWDCPa
buXCDscJcLJRKZVc62gOZXXtPfoHqvUPp3nuLA4YjH9bphbrMWMf810Wxz9JTd3v
yWgGqNY0zbBqeZoGv+TuExlRHT8ASGFS9SVDABEBAAHNNUdpdEh1YiAod2ViLWZs
b3cgY29tbWl0IHNpZ25pbmcpIDxub3JlcGx5QGdpdGh1Yi5jb20+wsBiBBMBCAAW
BQJZlGhBCRBK7hj4Ov3rIwIbAwIZAQAAmQEIACATWFmi2oxlBh3wAsySNCNV4IPf
DDMeh6j80WT7cgoX7V7xqJOxrfrqPEthQ3hgHIm7b5MPQlUr2q+UPL22t/I+ESF6
9b0QWLFSMJbMSk+BXkvSjH9q8jAO0986/pShPV5DU2sMxnx4LfLfHNhTzjXKokws
+8ptJ8uhMNIDXfXuzkZHIxoXk3rNcjDN5c5X+sK8UBRH092BIJWCOfaQt7v7wig5
4Ra28pM9GbHKXVNxmdLpCFyzvyMuCmINYYADsC848QQFFwnd4EQnupo6QvhEVx1O
j7wDwvuH5dCrLuLwtwXaQh0onG4583p0LGms2Mf5F+Ick6o/4peOlBoZz48=
=HXDP
-----END PGP PUBLIC KEY BLOCK-----
```

You may want to set the following in your `~/.bash_profile`

```bash
# git can now ask for gpg key passphrase
export GPG_TTY=$(tty)
```


## GitHub

GitHub is the tool we use for all code related sharing and storage.

TODO: More on our GitHub process.

Ensure you have a GitHub account.

Login to your GitHub. After logging in do the following to ensure you are using GitHub the best way possible:

1.  Ensure 2FA/MFA are enabled on [the security settings for your account](https://github.com/settings/security)

2.  Add your SSH public key to [the SSH and GPG keys section of your account settings](https://github.com/settings/keys)

```bash
# To get your public key onto your keyboard:
cat ~/.ssh/id_ed25519.pub | pbcopy
```

3.  Add you PGP public key to [the SSH and GPG keys section of your
    account settings](https://github.com/settings/keys)

```bash
# To get your public key for uploading to your GitHub account:
GPG_KEY="$(gpg --list-secret-keys | grep -B 1 '<G Suite email>' | head -n 1 | tr -d '[[:space:]]')" && [ -z "${GPG_KEY}" ] && echo $'\n#####\nFAILURE\n#####\n' || echo $'\n#####\nSUCCESS\n#####'

# Check to make sure the previous result does not indicate a failure before proceeding
gpg --armor --export "${GPG_KEY}" | pbcopy
```

When cloning, prefer to clone using ssh or `ssh://git@github.com/your-org/...`.

## Python

If you are planning on doing python development, the preferred toolchain
is to use:

-   pyenv
-   pipenv

### pyenv:

Installing:

``` bash
brew install pyenv
```

To initialize pyenv for shell integration, add the following to your `~/.bash_profile`:

```bash
if command -v pyenv &> /dev/null; then
    eval "$(pyenv init -)"
fi
```

Installing new versions of python to `pyenv`:

``` bash
pyenv install 3.7.4
```

To setup a project using a set version of python:

``` bash
cd new-project/
echo '3.7.4' > .python-version
```

### pipenv

Installing:

``` bash
brew install pipenv
```

Add the following to your `~/.bash_profile`:

``` bash
if command -v pipenv &> /dev/null; then
    eval "$(pipenv --completion)"
fi
```

### Docker

```
brew cask install docker
```

# Common Tools

## iTerm

[iTerm is a terminal emulator](https://www.iterm2.com/) that extends the functionality of the built in OSX terminal.

## bash-autocomplete

``` bash
brew install bash-completion
echo $'\n[[ -r "/usr/local/etc/profile.d/bash_completion.sh" ]] && . "/usr/local/etc/profile.d/bash_completion.sh"\n' >> ~/.bash_profile
```
