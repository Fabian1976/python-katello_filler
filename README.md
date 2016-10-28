# python-katello_filler

#STILL WORKING ON DOCUMENTATION. REPO WILL BE ADDED SOON

##Configuration
The script needs a configuration file with this content:
```
[common]
katello-fqdn=katello.example.com
katello-admin_user=admin
katello-admin_password=password
organizations_location=/vagrant/organizations
organizations=Example
```

The items are pretty self explanitory. The most important one is the organizations_location. This points to a place on your harddrive where the script can find the Organizations listed in the config item 'organizations'.
The organization is then built up with a specific directory structure. An example structure is in this repo, but it should be like this:
Asume: organizations_location=\katello_organizations
Folder structure (f = file, d = directory):
f operating-systems.conf
d Example
   |
 f activation-keys.conf  
 d gpgkeys
 d products
 d Location1
    |
  f domains.conf
  f environments.conf
  f hostgroups.conf
  f installation-media.conf
  f partition-tables.conf
  f proxies.conf
  f subnets.conf
  f templates.conf
  d partition_tables
  d templates
     |
   d provision
   d snippet

##The conf files and folders
###operating-systems.conf
This file is placed at the top of the directory structure because it is not part of an organization or location. It exists in every organization and location.
This does not mean that it is processed as the first item! This is not possible because it is dependent on installation-media and a Kickstart file (which can also be custom)
An example:
```
[CentOS 7]
name=CentOS
major=7
os-family=Redhat
architectures=x86_64
installation-media=CentOS 7 Installer
partition-tables=Kickstart default
```

###activation-keys.conf
In this file we put the activation keys which we wish to use and which product (subscriptions) are accessable through this key.
```
[CentOS 7]
lifecycle-environment=Library
subscriptions=CentOS7, EPEL_custom
```
The subscriptions mentioned in this file must be present in the products folder of an organization. The products can also contain seperate RPM's files instead of a external repo.
EPEL_custom will be used as a custom repo with only includes some packages from the EPEL repo. We only use a couple of packages and don't want any other packages from EPEL because of possible dependency conflicts.

###gpgkeys folder
In this folder we just place the gpg keys which we want to import in Katello.
So if we download this file in this folder:
`$ wget http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-7`
This file will now be imported in Katello under the organization of which the folder gpgkeys is placed under.

###products folder
In this folder we can place seperate config files for each product we want to add to this organization. For example, if we wish to add the CentOS7 repo, just place a file called CentOS7.conf in this folder with the appropiate info, and it will be added to this organization.
`CentOS7.conf`
```
[base x86-64]
gpg-key=RPM-GPG-KEY-CentOS-7
content-type=yum
url=http://ftp.nluug.nl/ftp/pub/os/Linux/distr/CentOS/7/os/x86_64/
publish-via-http=true
async=true

[updates x86-64]
gpg-key=RPM-GPG-KEY-CentOS-7
content-type=yum
url=http://ftp.nluug.nl/ftp/pub/os/Linux/distr/CentOS/7/updates/x86_64/
publish-via-http=true
async=true
```
Now the product CentOS7 will be created and 2 repositories will be added named "base x86-64" and "updates x86-64".

So all .conf files in this directory will be added as a product. The filename (minus the extension offcourse) will be the product name.

In the product folder, we can also say which RPM's we want to upload to a repository (custom RPM's or just a selection of EPEL for example)
`EPEL_custom.conf`
```
[el7 x86-64]
gpg-key=RPM-GPG-KEY-EPEL-7
content-type=yum
#url=https://dl.fedoraproject.org/pub/epel/7/x86_64/
#files_location=/custom_rpms/epel7/
publish-via-http=false
async=true
```
If you want to add custom RPM's to a repository, just remove the url section of the conf file. You can then add a files_location item to the conf file. All the files in this location, will be added to the repository.
The files_location item is not mandatory. If you also ommit the files_location, the script will asume that there is a folder called `EPEL_custom` in the products folder and add all the files in that folder to the repository.
The item `async` tells the script to add a task to synchronize the repository and continue with adding the rest. If it is set to false, the script will wait until the synchronization is completed (which could take some time), so it is advised to set this to true.

###domains.conf
