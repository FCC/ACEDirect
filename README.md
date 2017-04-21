![](images/adsmall.png)

# ACE Direct v1.1

Accessible Communications for Everyone (ACE) Direct is a Direct Video Calling (DVC) platform that enables direct calling from deaf and hard-of-hearing individuals to an American Sign Language (ASL)-trained agent within an organization's call center. The agent handles the call using a video-capable phone with real-time video connection. To demonstrate the capabilities of DVC, the FCC and CAMH have further advanced the original auto-routing POC into a call center platform for two to ten customer service representatives.

## Release v1.1

The latest release of ACE Direct is **Release v1.1**. See the RELEASENOTES.md file for the changes included in this release.

## Getting the Latest Software

The latest Release v1.1 includes several Git submodules in this repo. Please clone the *Master* branches and checkout the *v1.1* tag of each of these Git submodules:

* acedirect
* acr-cdr
* aserver
* userver
* managementportal
* esb
* asterisk
* hashconfig
* fendesk
* autoinstall

## Documentation
Please see the *docs* folder for complete documentation on the project overview, installation, and configuration. See the README.md files in the Git submodules for more information.

## Single Server Installation (example)

### Prerequisites

1. Install Node.js
1. Install MySQL
1. Install Asterisk server
1. Install STUN server

### Installation Instructions

```
[centos@yourmachine ~]$  cd /home/centos
[centos@yourmachine ~]$  git clone git@github.com:mitrefccace/autoinstall.git
[centos@yourmachine ~]$  cp autoinstall/installer.py .
[centos@yourmachine ~]$  python ./installer.py 
```

This will start the installation menu. Install all servers (Options 1-7). For each server:

* Specify the location of the customized *_TEMPLATE configuration file
* Enter 'y' to edit the configuration file
* Modify or accept the default value for each parameter and press <Enter>

After installing all servers, selection option *0. Finish installation process*. This will quit the menu and start up all servers. 

## Submodule Summary

### ACE Direct
ACE Direct is the main application of the ACE Direct system.

### ACR-CDR
ACR-CDR is a standalone application that provides call detail information.
 
### Management Portal
The Management Portal is a standalone application that provides management information.
 
### Asterisk
The Asterisk application is a standalone component that is the Virtual PBX.
 
### ASERVER
ASERVER is a standalone application that provides agent information to ACE Direct. The interface is a RESTful API.
 
### USERVER
USERVER is a standalone application that provides user information to ACE Direct. The interface is a RESTful API. 
 
### ESB
ESB is a standalone application is an enterprise service broker for ACE Direct.
 
### HashConfig
HashConfig is an open-source, standalone Node.js application that automates the process of editing a JavaScript Object Notation (JSON) file. This tool facilitates the editing of the encrypted configuration files.

### Fendesk
Fendesk is a simple ticketing system that creates and maintains tickets as local JSON files. It emulates real-world ticketing systems. 

### Autoinstall
This repo contains an installation script that automates ACE Direct software installation.