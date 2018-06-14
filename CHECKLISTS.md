![](images/adsmall.png)

# ACE Direct Installation/Configuration Checklist

* Acquire registered domain and subdomain names.
* Add fully-qualified domain names (FQDNs) to the provider peering lists; this request may take up to two weeks to fulfill.
* Create the AWS instances for the node, OpenAM, stun, and Asterisk servers. Do not use special characters for the server names.
  * Domain names *must* be two-level domain names; this is a requirement by OpenAM
  * _nodeace.domain.com_, _openamace.domain.com_, _stunace.domain.com_, and _sipace.domain.computer_
  * On _nodeace_, modify SE Linux: ```sudo setsebool -P httpd_can_network_connect 1``` . You will need to do this after any reboot.
  * Set all servers to the UTC timezone.

* Create the _acedirect_ application user on _nodeace_.
* Install and configure NGINX v1.12.2 on _nodeace_. See the _README.md_ file in the _nginx_ repo. Use the _nginx.conf_ file as a template. You may omit this step if you use the automated installation script to install ACE Direct.
* Install and configure Redis v3.2.10 on _nodeace_. You may omit this step if you use the automated installation script to install ACE Direct.
* Update the _/etc/hosts_ file on _nodeace_. You may omit this step if you use the automated installation script to install ACE Direct. Here is a sample template:

  ```
  127.0.0.1 nodeace.domain.com localhost localhost.localdomain localhost4 localhost4.localdomain4
  ::1 localhost localhost.localdomain localhost6 localhost6.localdomain6
  <openamace private IP> openamace openamace.domain.com
  ```

* Create website certificates. We need the registered, active domain name first. You should consult your IT department to generate these certificates. Or, just create self-signed certs for testing. Here is how to configure the certs:
  * The certificates should be *wildcard* certs to have flexibility with domain names.
  * The certificates must have `644` permissions.
  * Name the certificates _cert.pem_ and _key.pem_.
  * Place certificates in this folder on _nodeace_: /home/acedirect/.ssl
  * If the cert is new, you may have to execute _restorecon_ on it, for example: `restorecon -R -v cert.pem`
  * To create self-signed certs (for testing only): ```openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out cert.pem```
  * Remember the location of the .ssl folder, as you will point to this during the configuration process.
* Create the _/home/acedirect/scripts_ folder and install the _itrslookup.sh_ script. You may omit this step if you use the automated installation script to install ACE Direct. For a manual installation copy this script from the _asterisk_ repo:
  ```
  cd /home/acedirect/scripts
  git archive --remote=<Git top-level URL>/asterisk.git HEAD:scripts itrslookup.sh | tar -x
  chmod 755 itrslookup.sh
  ```
* Install and configure OpenAM on _openamace_.
  * See the README.md file in the _iam_ repo for detailed installation and configuration instructions.
  * Install Java v1.8.
* Install and configure MySQL v5.7 (or later) on a server accessible by _nodeace_ and _sipace_.
  * Use the _acedirectdefault.sql_ script in the _dat_ repo to create the default database schema, along with initial data.
  * See the _README.md_ file in the _dat_ repo for detailed instructions.
  * You may optionally install MySQL as an AWS RDS service.
  * Create an ACE Direct database user; note the database user, password, and hostname for the configuration process.
* Install and configure the stun server on _stunace_.
* Install and configure Asterisk v15.1.2 on _sipace_. See the README.md file in the _asterisk_ repo for detailed instructions.
* Install and configure MongoDB v3.2.18 on _nodeace_. You may omit this step if you use the automated installation script to install ACE Direct.
* Install Node.js, git, wget, bower, pm2, and apidoc. The automated installation script will install these tools for you.
* Install and configure Node.js application servers on _nodeace_. The instructions below are for a _Quick_ installation. You may choose different options when installing ACE Direct:
  * You will need Internet access for this step, since the Node servers require external libraries.
  * Go to the home folder: ```cd /home/acedirect```
  * Clone the _autoinstall_ repo.
  * Copy the _autoinstall/installer.py_ script to the /home/acedirect folder.
  * Run the installer by specifying the repo URL and branch, for example: ```python ./installer.py -s <Git top-level URL> -b <Git branch or tag>```
  * Note that the installer allows you to automatically install some of the third party software listed above.
  * When prompted, select Option 1 for a quick installation.
  * When prompted, specify the path to the configuration template: ```/home/acedirect/dat/config.json_TEMPLATE```
  * When prompted, enter appropriate values for each configuration parameter. See the _parameter_desc.json_ file in the _dat_ repo for descriptions of each parameter.
  * When prompted, decide if you want the configuration file to be encoded. If so, set the _cleartext_ parameter to "". Otherwise, set the _cleartext_ parameter to a non-empty string (e.g., "not encoded").
  * Verify that the Node.js servers are running _nodeace_: ```pm2 status```
* Install and connect the BusyLight to the agent computer.
  * Connect the USB BusyLight to the agent computer.
  * Download and the _lightserver.jar_ Java program from the _busylightapi_ repo. Run it on the Agent's client computer.

# ACE Direct Reboot Checklist

  * Set SE Linux variable: ```sudo setsebool -P httpd_can_network_connect 1```
  * Start NGINX: ```sudo service nginx start```
  * Start Redis: ```sudo service redis start```
  * Start MongoDB: ```sudo service mongod start```
  * Restart Node: ```pm2 restart all```

# ACE Direct Troubleshooting Checklist

  * Run ```pm2 status``` and see if the servers are restarting. There should be no automatic restarts.
  * Set the common:debug_level parameter in ~/dat/config.json to *ALL* to receive all messages in the log files.
  * Check the *logs* folder in each application folder for errors.
  * Verify that OpenAM, Redis, MongoDB, NGINX, and MySQL are running.
  * Does the BusyLight device respond? Try the self-test mode on the lightserver.jar UI.
  * Verify that the /etc/hosts file is configured correctly.
  * Verify that the /etc/nginx/nginx.conf file is configured correctly.
  * Verify that ~/dat/config.json is configured correctly.
