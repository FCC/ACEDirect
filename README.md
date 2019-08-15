![](images/adsmall.png)

# ACE Direct

Accessible Communications for Everyone (ACE) Direct is a Direct Video Calling (DVC) platform that enables direct calling from deaf and hard-of-hearing individuals to an American Sign Language (ASL)-trained agent within an organization's call center. The agent handles the call using a video-capable phone with real-time video connection. To demonstrate the capabilities of DVC, the FCC and CAMH have further advanced the original auto-routing POC into a call center platform for two to ten customer service representatives.

## Release v3.2.0

The latest release of ACE Direct is **Release v3.2.0**. See the [RELEASENOTES.md](RELEASENOTES.md) file for the changes included in this release.

## Clean Installation

To install ACE Direct from scratch, see the [CHECKLISTS.md](CHECKLISTS.md) file. This file describes the installation and configuration process for the ACE Direct system.

## Getting the Latest Software

If you only want to download the latest software, start here. The latest release includes several Git **submodules** in this repo. Please pull the Git submodules linked here, for example:

```sh
user@yourmachine:~$  git clone git@github.com:mitrefccace/acedirect-public.git  # get top repo
user@yourmachine:~$  cd acedirect-public
user@yourmachine:~$  git submodule init  # initialize local config file
user@yourmachine:~$  git submodule update # get individual repos
```

Next, check out the **v3.2.0** tag (or the most current tag) of each of the submodule repos. Some repos may not have been updated for the most current release. This will ensure that all submodules are in sync.

## Documentation

Please see the *docs* folder for official documentation on the project overview, installation, and configuration. See the README.md files in the Git submodules for installation and configuration details of those individual repos.
