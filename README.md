# Introduction

This project will set you up with a developer virtual machine using Vagrant with provisioning for the DotCom site.

# Prerequisites

To run this project you will need to have Vagrant and Virtualbox running on your local machine.

https://www.vagrantup.com/downloads.html
https://www.virtualbox.org/

It is always advisable to run the latest versions of both.


# Quick Start
 
- Create a new directory for the vagrant box:
  ```
  mkdir -p ~/eurostar/vagrant && cd ~/eurostar/vagrant
  ```

  
- Acquire a copy of the project: via git clone

  You can acquire the vagrant project direct via git (including submodules) recursively:

  ```shell
  git clone \
    --branch dev \
        git@github.com:EurostarDigital/eurostar_vagrant.git \
        --recursive
  ```

- (optional) create / modify your local config file

  You can run at any time the `configure` script to create you a new `local.config.yaml` file or to help you edit your settings:

  ```bash
    ./configure
  ```

  If you have an existing `local.config.yaml` it will UPDATE not overwrite your local config.


- Start the vagrant up process (to lauch the vm): 

  ```bash
  vagrant up --provider=virtualbox
  ```

  Once this has completed (booting, downloading and provisoning the site) you should have a working vagrant box.

- You can customise the branch or tag to deploy via the `site_args:` and `branch_tag:` options in your `local.config.yaml` file. 


# Configuring the virtual environment


## Puppet Node classes, environments, providers and other options using environment variables

This Vagrant project utilises our Puppet scripts allowing us to create a large variety of nodes matching our production environment wherever possible. Nodes can be differentiated by:-


-  __node class__ for example `mysql` or `memcache`
-  __environment__ for example `integration`, `test` or `EWT`(UAT equivalent)
-  __provider__ for example `virtualbox`, `rackspace` cloud or AWS.
- plus various other vm customisation options



## Config via environment variables

We select these options by using environment variables which we set via terminal before running any vagrant commands, for example:

```
export VAGRANT_PUPPET_NODE_CLASS=evomysql
export VAGRANT_PUPPET_NODE_ENVIRONMENT=ewt
export VAGRANT_PUPPET_PROVIDER=virtualbox
```

The above example would create a MySql node suitable for dotcom (evo) configured correctly for the ewt(UAT equivalent) environment in Virtualbox.

When using the virtualbox provider you can also supply the following environment options:

- `VAGRANT_CPUS=<number_cpu_cores>` how many virtual CPU cores to apply to the virtual box vm (defaults to *1*)
- `VAGRANT_MEMORY=<memory_size_mb>` how much RAM in mb to apply to the virtual box (defaults to *2048*)
- `VAGRANT_IOAPIC=on|off` enable or disable the IOAPIC (defaults to *on*) - when using centos+virtualbox will improve performance when _on_

Complete Example (8 cpu cores + 8GB ram):

```
export VAGRANT_CPUS=8 VAGRANT_MEMORY=8192 VAGRANT_PROVIDER=virtualbox
vagrant destroy -f ; vagrant up
```


## ... via local config file

For a more persistent config, and as an alternative to environment variables, you can supply most config values to vagrant via setting the appropriate values inside a __local yaml configuration file__ called `local.config.yaml` which can be found in the same folder as your `Vagrantfile`, Here is an example config file plus brief overview of some common config values:

```yaml
box_name_suffix: vm1
branch_tag: dev
cpus: 4
memory: 4096
provider: virtualbox
puppet_environment: developer
puppet_eurostar_install: jenkins
puppet_node_class: evodevboxvirtualbox
s3_access_key: AKIAIRDONOTUSETHISKEYZQ
s3_secret_key: PpbYStkDONOTUSETHISKETYbKJADGe
site_args: --skip-vpn 
vagrant_git_branch: dev
vagrant_git_url: git@github.com:EurostarDigital/eurostar_vagrant.git
```

Where:
- `box_name_suffix` sets the suffix of the VM to allow multiple instances to
    run at once.
- `branch_tag` to device what git branch / tag / commit to deploy.
- `cpus` to define how many cores the VM should use.
- `memory` to define how much RAM to allocate to the VM.
- `provider` sets what provider to make vagrant use
- `puppet_environment`
- `puppet_eurostar_install`
- `puppet_node_class` defines 
- `s3_access_key`, `s3_secret_key` set to you own S3 bucket keys
- [`site_args`](#SITE_ARGS) defines a number of options to be passed to the provisioning script (see [`site_args`](#SITE_ARGS))
- `vagrant_git_branch`, `vagrant_git_url` define which vagrant config to clone



# SITE_ARGS

You can specify a number of options to pass to the shell provisioning script that effect what gets run (or not run), find below a quick summary of these options:

Option | Description 
-------|------------
`--skip-ntp`   | disable syncing  guest clock with the host
`--skip-vpn`   | disable use of VPN for connectivity into test environment
`--skip-db`    | disable MySql DB download and import 
`--force-db`   | force download and import of database ignoring lock files
`--skip-s3`    | disable copy and deploy from S3 bucket 
`--skip-drush` | disable drush command execution

# re-provisioning VM to pick up changes made to config or environment variables

Once you have run `vagrant up` and you have a running VM, you can make changes to your `local.config.yaml` file (for example to set some `site_args` or to define a  `branch_tag`) and have them applied to the up-and-running-vm by running the following command:

```
vagrant provision --provison-with=shell
```

This will skip the puppet run and save you some time. You might also consider using some of the `--skip-blar` site_args too - for example:

```
site_args: --skip-db --skip-s3
```

Which would inhibit the dropping and re-importing of the database and also skip the re-deployment of the code from the artefacts archive stored in the S3 bucket - leaving just the drush commands to be executed.



## Vagrant Plugins

We now make use of the following plugins:
- `vagrant-cachier`: prevent duplicate downloads and speed up package installs
- `vagrant-vbguest`: ensure guest additions are up to date / installed
- `vagrant-hostmanager`: manage hostname's and hosts file entries
(all of which will be auto installed upon the first execution run)


    
## AWS s3 bucket access

You must provide a AWS 'secret_key' and 'access_key' to facilitate acquisition of the packaged Drupal artefacts. These AWS credentials should be provided as the following environment variables:

```
export S3_SECRET_KEY=<aws_s3_secret_ket> S3_ACCESS_KEY=<aws_s3_access_key>
```

Both of the above variables can also be defined via the `local.config.yaml` file using the keys `s3_secret_key` and `s3_access_key` options.



## Acquiring a specific branch or tag of the Drupal site

You can make the provisioner fetch a specific git branch or tag to deploy by setting the `BRANCH_TAG` environment variable, for
example:

```
export BRANCH_TAG=dev
```

Or

```
export BRANCH_TAG=MYTAG-123
```

If unset, will default to using *dev* branch.

This can also be set via the `branch_tag:` line in the `local.config.yaml` file.



## Handling Failure

In the highly unlikely event the post-puppet provisioner shell script should fail or you are forced to land in water, you can supply various arguments to the provisioning shell script via the use of the `SITE_ARGS` environment variable.

This allows you to, for example skip previously completed steps such as the database import - saving much time.


For example to re-run the shell provisioning, whilst skipping the database import step you could:

```
SITE_ARGS=--skip-db vagrant provision
```


## Node Classes

Technically you have the option of creating any node that has a .yaml file within puppet/hieradata however the profiles that are of most interest are the ones prefixed with EVO or ECM as these are complete specifications for a node e.g evomysql.yaml contains the config to build a full mysql node including sudo and other config whereas mysql.yaml contains only raw mysql config.  

Node classes are set using the environment variable `VAGRANT_PUPPET_NODE_CLASS`

The nodes that are most interest to us in dotcom are:-

- evodevboxvirtualbox
- evodevboxrackspace
- evodevboxaws
- evomysql
- evowebserver
- evonfs
- evomemcached

we also have the POCs:-
- evodevboxnginx
- evodevboxrackspace55 (dotcom site on PHP 5.5)

The nodes that are of interest to the mobile project are:-
- ecmdevbox
- ecmwebdev
- ecm_preprod_a
- ecm_preprod_b
- ecm_perf_a
- ecm_perf_b
- ecmeif
- ecmweb
- ecmzeus
- ecm_stg
- ecm_prod_a
- ecm_prod_b

## Environments

For dotcom there are environments in hiera for each environment we use from developer through to production. The changes in each environment can be seen by viewing the appropriate file in puppet/hiera/{environment}. These changes may relate to endpoints for APIs, test or live values etc.
The available environments in dotcom are:-
- developer
- integration
- test
- ewt
- e2e
- staging
- dr
- prod

`VAGRANT_PUPPET_ENVIRONMENT`controls which environment you want to use. For example


```export VAGRANT_PUPPET_ENVIRONMENT=ewt``` 


Environments in Puppet have inheritance, so the main config is held in common 
then this is either extended or overridden in the appropriate environment. For more info please view either the main Puppet project

https://github.com/EurostarDigital/eurostar_puppet

or


https://eurostarwiki.atlassian.net/wiki/display/infra/Configuration+Management

## Providers

We currently cater for the following providers through Vagrant:-
- virtualbox
- rackspace
- aws

Providers are selected by the environment variable `VAGRANT_PUPPET_PROVIDER`.

### Virtualbox

Virtualbox is the default provider so we don't need to do anything other than selecting the appropriate node class for example evodevboxvirtualbox

### Rackspace

Rackspace Cloud integration requires a Vagrant plugin

`vagrant plugin install vagrant-rackspace --plugin-version 0.1.5`

You will need to select the appropriate provider

`export VAGRANT_PUPPET_PROVIDER=rackspace`

Then vagrant up with the command:-

`vagrant up --provider=rackspace`


### aws

AWS requires a vagrant plugin

`vagrant plugin install vagrant-aws`

You will need to select the appropriate provider

`export VAGRANT_PUPPET_PROVIDER=aws`

Then vagrant up with the command:-

`vagrant up --provider=aws`

More information can be found on using aws with Vagrant at:-

https://eurostarwiki.atlassian.net/wiki/pages/viewpage.action?pageId=40863704


## Other Options
You can install the eurostar.com site or mobile site by using the environment variable

`VAGRANT_PUPPET_EUROSTAR_INSTALL`

The available options are:-
- no (don't install site)
- jenkins (install an up to date version of the site and grab a fresh copy of the db from Jenkins)
- local (build a version of the site based on the copies of code and db existing on your machine)
- ecmjenkins (build a version of the ecm site)

`VAGRANT_MEMORY` controls how much memory is assigned to the virtualbox vm.   default is 2GB




# mount guest path (via sshfs)

To assist mounting a path in the guest onto tyour host, use the script `mount-guest-sshfs` which takes care of permissions and ssh-foo:

## usage

```
cd [PATH_TO_VAGRANT_FOLDER]
./mount-guest-sshfs
```


## config entries
- `guest_mount_path` path on guest to sshfs mount
- `host_mount_path` path on host to mount from guest

### example config entries for mount-gurest-sshfs

To get the path `/var/www` on the guest to be mounted into the host path `$PWD/my-guest-www` you could use the following config file snippet:

```
guest_mount_path: /var/www
host_mount_path: ./my-guest-www
```




 # Dotcom quick start
If you want to create a dotcom developer machine in Virtualbox simply ensure you have the prerequisites, clone the repo as per the instructions above then

`vagrant up`

This will give you a box with the following options

```
vagrant_puppet_provider = ENV['VAGRANT_PUPPET_PROVIDER'] || 'virtualbox'
vagrant_puppet_node_class = ENV['VAGRANT_PUPPET_NODE_CLASS'] || 'evodevboxvirtualbox'
vagrant_puppet_eurostar_install = ENV['VAGRANT_PUPPET_EUROSTAR_INSTALL'] || 'jenkins'
vagrant_memory = ENV['VAGRANT_MEMORY'] || '2048'
vagrant_puppet_environment = ENV['VAGRANT_PUPPET_ENVIRONMENT'] || 'developer'
```

You should be able to visit 10.0.0.42 to see your local site

You can ssh into your machine using

`vagrant ssh`

For more details on using vagrant please visit

http://www.vagrantup.com
 
# ECM quick start
To create a ecm (mobile) site on Virtualbox use the following instructions:-
```
export VAGRANT_PUPPET_NODE_CLASS=ecmdevbox
export VAGRANT_PUPPET_EUROSTAR_INSTALL=ecm_jenkins
vagrant up
```

# Dev workflow
Since the split into two repos all of the vagrant related resources are in this repo and the puppet specific resources are added as a submodule.

When you clone recursively this will populate all of the necessary resources to your local repo, however the workflow is slightly different.

Any changes made to the vagrant related resources will be committed and pushed as normal. (Don't try and stage and commit anything that is in the 'puppet' directory from within the vagrant repo).

To commit and changes made to the puppet repo and ensure you push to your own fork:

```
cd puppet
git remote rm origin 
git remote add origin git@github.com:path-to-my-fork.git
git checkout master
```

Changes can be staged and commited to your fork of the puppet repo as normal now


Just a note about the synced folder, you will need to add read permissions to external users on your local machine I.E. Mac. If you don't then vagrant won't be able to access the folder.

# Environment Variables accepted by this vagrant

Here is a list of all the environment variables that are currently accepted or made use of my the vagrant provisioner:

- `SITE_ARGS` allows you to supply additional arguments to the post-puppet-provisioner.
   example:  `SITE_ARGS=--skip-db`
- `BRANCH_TAG` define which version of the site code to deploy.
- `VM_HEADLESS` set vm to run non headless with a gui (boolean 1/0) ie set `VM_HEADLESS=0 vagrant reload`


# @todo
- AMQ on aws boxes
- aws requires restart for selinux to work
 sudo permissions for eurostar user
=======
- sudo permissions for eurostar user


# Gotchas

### Handling LF and CRLF Line Endings
People using different operating systems may result in problems with line endings. Unix,Linux, and OS X use LF and Windows uses CRLF to denote the end of a line. 

To have Git solve such problems automatically, you need to set the `core.autocrlf` attribute from your .gitconfig to `true` on Windows and to `input` on Linux and OS X.

More info: https://help.github.com/articles/dealing-with-line-endings/#platform-all
