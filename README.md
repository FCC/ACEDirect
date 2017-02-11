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

## Documentation
Please see the *docs* folder for complete documentation on the project overview, installation, and configuration. See the README.md files in the Git submodules for more information.

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
 
 



