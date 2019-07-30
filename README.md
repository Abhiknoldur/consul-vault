Hashicorp vault & consul cluster
================================

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Installs & confgures Hashicorp's Consul & Vault + HAProxy to run as an HA cluster

### Symbolic representation overview of deployment
![Alt text](/images/Vault-Consul-Cluster.jpg "Overview of deployment")

### Consul UI Dashboard
[![Alt text](/images/Consul-Vault-UI.jpg "Consul UI Dashboard")](https://www.consul.io/docs/index.html)

Although the vault installation creates OpenSSL TLS private key, CSR & resultant certificate, the URI modules in these roles currently use "validate_certs: no". It is up to you to complete the TLS configuration.

> __WARNING__ - When vault is initialized, the master key shards & root token are stored in the ansible user's HOME dir on the Ansible control machine. This is __NOT__ good practice, but was used to get things running. I am considering various future options that won't break the non-interactive execution of the playbooks, such as ansible vault'ing the file with a pre-defined ansible vault password file. But this is really no more secure than the current setup. Hashicorp vault has the ability to encrypt the master key shards using [PGP, GPG, and Keybase](<https://www.vaultproject.io/docs/concepts/pgp-gpg-keybase.html>). This is the ideal solution but might prove too difficult to implement while maintaining non-interactive playbook execution.

Heavily based on the documentation supplied by HashiCorp at <https://www.vaultproject.io/guides/operations/vault-ha-consul.html>

Currently tested on these Operating Systems
* Oracle Linux/RHEL/CentOS 7 (Note: Enables EPEL repo using Jeff Geerling's [EPEL role](<https://galaxy.ansible.com/geerlingguy/repo-epel/>))
* Debian/Stretch64

Terraform currently tested on these cloud providers
* DigitalOcean

Requirements
------------

* Hashicorp Vagrant
* Hashicorp Terraform (*installed from ansible playbooks*)
* Ansible 2.5 or higher

Dependencies
------------

* Requires elevated root privileges
* Vagrant - Copy Ansible control machine user's public SSH key (usually called id_rsa.pub) into the vagrant working directory
* Terraform (DigitalOcean) - Export DO_API_TOKEN & DO_SSH_KEYS environment variables (see below for helpful tips and env source file)

*DigitalOcean Environment Variables*

First you will need to create a DigitalOcean "Personal Access Token" by following these [instructions](https://www.digitalocean.com/community/tutorials/how-to-use-the-digitalocean-api-v2#HowToGenerateaPersonalAccessToken)

Then you will need the fingerprint of your private SSH key you will use to connect to the droplets. This can be done in a couple of ways:
1. Login in to digitalocean website, navigate to account, security and copy the appropriate key's fingerprint
2. Generate the fingerprint by running the following command against the local private key

```
ssh-keygen -E md5 -lf ~/.ssh/id_rsa.pub | awk '{print substr($2,5)}'
```

Now create a helpful environment source file

`vi ~/do_rc.sh`

```
export DO_API_TOKEN="53fb6cddc2db7b1d293c30b1c8fe37da1908f7a8bb82ec14d9ca4eacca6c3fa5"
export DO_SSH_KEYS="3a:92:29:fc:a0:8d:77:d7:d3:01:80:c7:cc:fd:9c:71"
```

Source this env file before running the ansible playbooks

`source ~/do_rc.sh`


Getting the code
----------------

`git clone https://github.com/AdamGoldsmith/consul-vault.git --recurse-submodules`

Running the deployment
----------------------

*Vagrant (Currently not working under terraform branch)*

```
cd vagrant
vagrant up
```

On the Ansible Control Machine  

`ansible-playbook playbooks/site.yml -i inv.d/vagrant --skip-tags 'terraform'`

*Terraform*

On the Ansible Control Machine  

__To deploy__

`./deploy.sh`

or

`ansible-playbook playbooks/site.yml -i inv.d/do --tags 'epel,install,init,unseal,configure,approle,sshkeysign'`

__To remove__

`./deploy.sh -t remove`

or

`ansible-playbook playbooks/site.yml --tags 'remove'`

[![asciicast](https://asciinema.org/a/dhB4noDm88CdISBGAUGonetAv.png)](https://asciinema.org/a/dhB4noDm88CdISBGAUGonetAv?autoplay=1&size=small&cols=140&rows=40)

Known Issues
------------

If you get the message "You need to have PyOpenSSL>=0.15 to generate CSRs", then it is most likely an issue with the OpenSSL package that python has imported. When pyOpenSSL is installed/upgraded via the PIP Ansible module in this playbok, it will install the python package under /usr/lib/pythonx.x/site-packages, however it is possible that another OpenSSL python package could be installed under /usr/lib64/pythonx.x/site-packages that is being loaded in preference to the higher-level package.  
In order to prevent this happening, temporarily move the directory "/usr/lib64/pythonx.x/site-packages/OpenSSL" out of the way while running this playbook.  

When testing terraform with DigitalOcean cloud provider, I would sometimes see a limited number of instances being created simultaneously. For example, I was testing with creating 7 instances (3 x consul, 2 x vault, 2 x haproxy), and often only 6 instances would get created. I do not know if this was a terraform or DigitalOcean bug/limitation. I originally thought it was happening after upgrading from terraform 0.11.14 to 0.12.x (hashicorp's latest major version at time of writing), but I have seen this issue with 0.11.14 too. If this happens, just re-run the ansible playbook and it will add the missing instance(s) to the environment.

License
-------

MIT License

Author Information
------------------

Adam Goldsmith

