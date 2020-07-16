# New Developer Setup

# Terminology

|     Term | Definition                                                                                         |
|      --- | ---                                                                                                |
|      VPN | Network tunnel to secure Cloud presence</br>The VPN is necessary to access any protected resources |
| Artifact | Docker container, Python module (Pipenv/pip), Node module (NPM)                                    |
|    Nexus | Artifact storage server                                                                            |

# Prerequisites

Before you're able to follow this guide, you'll need the following
things:

-   G Suite account (@example.com email)
-   Setup your Datadog Account
-   [Nexus](https://www.sonatype.com/nexus-repository-oss) username and password 
-   Tools:
    -   A package manager:
        -   MacOS [homebrew](https://brew.sh/)
        -   Windows, not supported but
            recommend [scoop](https://scoop.sh/)
    -   [npm](https://www.npmjs.com/get-npm)
        -   `brew install node`
    -   [pip](https://pip.pypa.io/en/stable/installing/)
        -   `brew install python` 
    -   [docker](https://docs.docker.com/install/)
        -   `brew cask install docker` 

This process should only take **5 minutes** of time to complete. You
should **follow it in order** if you've never completed this guide
before.

# VPN Setup

**VPN Setup (Mac OS)**

Let's install a VPN client on your Mac.

Don't worry, our VPN is lightweight and will not slow your computer
down.

1.  **Install Tunnelblick** - this allows you to connect to the VPN  
    Go to <https://tunnelblick.net/downloads.html> and download the
    latest **Stable** version of Tunnelblick  
      

2.  **Sign into the VPN server**  
    Go to <https://vpn.example.com> and click "**Log in with Google**".  
      
    Once prompted, ensure you click on your
    @**[example.com](http://example.com)** Google
    account  
      

3.  **Download your VPN profiles** - these will be imported by
    Tunnelblick allow your Mac to connect to the VPN

    Click "**Show More**"  
      
    Then click "**Download Profiles**"  

    ...and then **save the file**.  
      

4.  **Import the VPN profiles**  
    Find the file that was downloaded in step 3 in your Downloads folder
    and extract (double click) it.  
    You'll now see a folder with your Google Username containing one or
    more files ending with the name ".ovpn".  
      
    **Double-click each of these files** and Tunnelblick will ask you to
    import the configurations. Select "**For me only**".  
    For all profiles, except **fullencryptonly** (previously
    **govtools-full**), change the `Set DNS/WINS`  value to
    `Set nameserver (3.0b10)`   
      

5.  **Test the VPN**  
    You can now connect to the VPN by using the new icon in your Mac's
    menu bar at the top.  
      
    Don't be concerned if you see only one "Connect" entry.  
    Nexus uses the "**govtools**" profile. Select that and it will
    connect after a few seconds.

# Troubleshoot VPN

## Tunnelblick was not able to load system extension

**VPN Troubleshooting**

In your keyboard, press Command key + Space bar → enter "Security &
Privacy" → Press Enter → Go to "General" tab and press "Allow"


# Nexus Setup

Now that you have the VPN set up, you can configure your Mac to use
Ivy's artifact repository system, Nexus, instead of the public NPM
and PyPi servers.

1.  In a terminal window (iTerm, Terminal.app), navigate to your
    favorite code folder (I suggest **`~/code`**) and run the following
    code snippet:  
    (ensure you're cloning with git@ instead of https!)

    ``` bash
    git clone git@github.com:nxtlytics/ivy-new-developer.git
    cd new-developer-setup/
    ```

2.  Now, we'll run the setup script for Nexus

    ``` bash
    ./reposetup.sh
    ```

    Please read the output of the script carefully.  
      
    You'll need to enter your email address, Nexus username,
    and Nexus password.  
    The script will ask you to confirm the details and then will run a
    self-test to ensure your Nexus access is set up properly.

# Party!

That's it. You're ready to use the VPN and Nexus for development!

# saml2aws (version 2.26.0 and up)

saml2aws is only for people who will need to interact with AWS using
awscli or any of their APIs

## Install saml2aws

**Install on MacOS using homebrew**

``` java
brew tap versent/homebrew-taps
brew install saml2aws
```

## Configure saml2aws

**\~/.saml2aws**

``` java
[dev]
app_id               = <You'll get this from the SAML Apps page on admin.google.com>
url                  = <Google URL>
username             = <Your Username>@example.com
provider             = GoogleApps
mfa                  = Auto
skip_verify          = false
timeout              = 0
aws_urn              = urn:amazon:webservices
aws_session_duration = 28800
aws_profile          = dev
resource_id          =
subdomain            =
role_arn             = arn:aws:iam::<account id>:role/<your role>
region               = us-west-2
```

  

# VPN Setup Ubuntu 18.04

## Setup

1.  Follow steps 2 and 3 from VPN Setup above

2.  By default, Ubuntu does not come with openvpn support installed,
    install it first.

    **Install openvpn support on Ubuntu 18.04**

    ``` bash
    $ sudo apt install -y network-manager-openvpn-gnome openvpn-systemd-resolved
    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    The following package was automatically installed and is no longer required:
      libllvm8
    Use 'apt autoremove' to remove it.
    The following additional packages will be installed:
      libnss-resolve libpkcs11-helper1 network-manager-openvpn openvpn
    Suggested packages:
      easy-rsa resolvconf
    The following NEW packages will be installed:
      libnss-resolve libpkcs11-helper1 network-manager-openvpn network-manager-openvpn-gnome openvpn openvpn-systemd-resolved
    0 upgraded, 6 newly installed, 0 to remove and 0 not upgraded.
    ```

3.  Import `.ovpn`  files

    **Import OpenVPN files**

    ``` bash
    $ cd /path/to/where/ovpn/files/where/downloaded/to
    $ sudo nmcli connection import type openvpn file ./sso-gsuite_user@example.com_profile.ovpn 
    [sudo] password for user: 
    Connection 'sso-gsuite_user@example.com_profile' (XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX) successfully added.
    $ # Repeat `sudo nmcli connection import type openvpn file` per file
    ```

4.  The following step applies to all VPN profiles except for
    fullencryptonly (previously govtools-full), Open the VPN profile
    configuration → `IPv4 Settings`  tab → click `Routes`  in this tab →
    Check the box for
    `Use this connection only for resources on its network` . (Example
    below is in Xubuntu, Gnome may have the VPN profile configuration in
    a different place)  

## DNS Setup (Systemd-resolve only)

1.  Confirm you are connected to the VPN where the tools are installed on

2.  Confirm `/etc/systemd/resolved.conf.d`  exist if it doesn't
    run `mkdir -p /etc/systemd/resolved.conf.d` (you may have to run it
    with `sudo` )

3.  Copy the following files into /etc/systemd/resolved.conf.d

    **dev.conf**

    ```bash
    [Resolve]
    DNS=10.255.20.1
    Domains=example.com
    ```

4.  `systemctl restart systemd-resolved` 

5.  `ping master.mesos.service.dev.example.com` 

    **Example:**

    ``` java
    $ ping master.mesos.service.dev.example.com
    PING master.mesos.service.dev.example.com (10.20.144.10): 56 data bytes
    64 bytes from 10.20.144.10: icmp_seq=0 ttl=254 time=58.004 ms
    64 bytes from 10.20.144.10: icmp_seq=1 ttl=254 time=63.079 ms
    ^C
    --- master.mesos.service.dev.example.com ping statistics ---
    2 packets transmitted, 2 packets received, 0.0% packet loss
    round-trip min/avg/max/stddev = 58.004/60.541/63.079/2.538 ms
    ```

6.  This is it!
