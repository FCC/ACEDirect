![](images/adsmall.png)

# ACE Direct

Accessible Communications for Everyone (ACE) Direct is a Direct Video Calling (DVC) platform that enables direct calling from deaf and hard-of-hearing individuals to an American Sign Language (ASL)-trained agent within an organization's call center. The agent handles the call using a video-capable phone with real-time video connection. To demonstrate the capabilities of DVC, the FCC and CAMH have further advanced the original auto-routing POC into a call center platform for two to ten customer service representatives.

## Release v2.x

The latest release of ACE Direct is **Release v2.x**. See the RELEASENOTES.md file for the changes included in this release. Four new repositories were added to this release: *busylightapi*, *iam*, *nginx*, and *virtualagent*.

## Getting the Latest Software

The latest release includes several Git **submodules** in this repo. Please pull the Git submodules linked here, for example:

```sh
user@yourmachine:~$  git clone git@github.com:mitrefccace/acedirect-public.git  # get top repo
user@yourmachine:~$  cd acedirect-public
user@yourmachine:~$  git submodule init  # initialize local config file
user@yourmachine:~$  git submodule update # get individual repos
```

Next, check out the **v2.x** tag (i.e., highest *v2.x* number) in each of the submodule repos. This will ensure that all submodules are in sync.

## Documentation
Please see the *docs* folder for official documentation on the project overview, installation, and configuration. See the README.md files in the Git submodules for installation and configuration details of those individual repos.

## Installation Script

The *installer.py* installation script in the *autoinstall* repo is an overall installation script that installs most of the components of the system. The submodules have further information on direct installation and configuration.

