OpenERP-inator
==============

*Dominate the entire ERP tri-state area by easilly installing and creating an army of OpenERP server instances in your server. The self-destruct button is intended only for test databases.*

OpenERP-inator is an utility script to manage a server with multiple OpenERP server instances and variants.
The motivation behind `oetor` was to make it easy to branch, develop and test OpenERP branches, and avoid the chaos that quickly piles up when working on multiple projects.


Installation
------------

To install in an Ubuntu system:

```bash
wget https://raw.github.com/dreispt/master/oetor  # download oetor script
bash oetor install                                # run install command
rm oetor                                          # cleanup
```


Quickstart full installation
---------------------------

The `quickstart` command provides a one line full installation of the latest stable version of OpenERP:

    /opt/openerp/oetor quickstart  # OpenERP dependencies and server isntallation 
    /opt/openerp/v7/main/start     # Start the 'v7' server instance

All needed dependencies are installed, including the PostgreSQL server.
The installed instance, `v7`, is built using the latest nightly build. 


Usage
-----

A step-by-step guide on OpenERP-inator's main features.
These steps can be followed either you used the `quickstart` or not.


### Preparation
 
For convenience, let's position in the home directory and confirm that all system dependecies are installed:

    cd /opt/openerp           # position at oetor home
    ./oetor get-dependencies  # install missing system dependencies


### Create server instances

Create two server instances, one using nightly build and running on port 8070,
and another using Launchpad sources and listening on 8071:

    ./oetor get nightly-7.0                # download latest 7.0 nighlty build
    ./oetor create prod7 nightly-7.0 8070  # create prod7 instance on port 8070
    prod7/main/start &                     # start prod7 instance in the background
    
    ./oetor get openerp-7.0                # download Launchpad 7.0 source code
    ./oetor create dev7 openerp-7.0 8071   # create dev7 instance on port 8071
    dev7/main/start &                      # start dev7 instance in the background

    ps aux | grep openerp-server           # list running instances and ports


### Chek versions and update sources

    ./oetor version ./src/nightly-7.0      # check shared nightly version
    ./oetor version ./prod7/main/server    # check instance source code version
    ./oetor get nightly-7.0 --update       # update nighlty build source code

    ./oetor version ./src/openerp-7.0      # check shared Launchpad source version
    ./oetor get openerp-7.0 --update       # update Launchpad source code


### Work on a project and test branches

Create an instance for the Department Management project, including it's modules in the addons path:

    ### Create an instance to work on the department-mgmt project ###
    ./oetor create deptm7 nightly-7.0      # create "dptm7" server instance
    bzr branch lp:department-mgmt/7.0 deptm7/main/deptm   # add the project's source
    ./deptm7/main/start                    # start instance (<ctrl+c> to stop it).
                                           # project is automatically added to the addons path
    
    ### Create a branch to work on Feature X ###
    cp dptm7/main dptm7/featX              # create a copy from the 'main' branch 
                                           # ...and you could work on it
    deptm7/featX/start -i crm_department --test-enable --stop-after-init  # test one module
    deptm7/featX/start -I dptm7 --test-enable --stop-after-init           # test all modules
    
    ### Remove an obsolete instance branch ###
    rm ./deptm7/featX -R && dropdb deptm7-featX


What's in the box?
------------------

Here is how source code directories and server instances are organized:

                                   ###$ ./install.sh
        /opt/openerp               # HOME directory
          |- oetor                 # oetor script
          |- /env                  # generic virtualenv (optional)
          |- /src                  # shared SOURCE REPOSITORY
          |    |- /nightly-7.0       # a SOURCE DIR, from nightly builds
          |    |    |- /server
          |    |                     ###$ `oetor setup sources 7.0`
          |    |- /sources-7.0       # a source dir (Launchpad checkout)
          |    |    |- /server       #    (/repos contains addons and web)
          |    |    |- /addons
          |    |    |- /web
          |    |                     ###$ `oetor setup sources trunk`
          |    |- /sources-trunk     # ...another version source dir
          |    |    |- ...
          |    ...                   # ...add other shared sources as needed
          |
          |                          ###$ ./oetor create server1 sources-7.0 
          |- /server1                # an OpenERP server instance
          |    |- openerp-server.conf 
          |    |- /main
          |    |    |- start         # script to start this server
          |    |    |- /env          # python virtualenv used (symlinked, optional)
          |    |    |- /server       # server sources: symlinked to dir in /opt/openerp/src
          |    |    | 
          |    |    |- /addons       # ... as many module directories as needed
          |    |    |- /web
          |    |    |- /projx
          |    |    ... 
          |    |
          |    |- /branchz           # instance source code (version z)
          |    |    |- start         
          |    |    |- /env          
          |    |    |- /server       
          |    |    | 
          |    |    |- /addons 
          |    |    |- /web
          |    |    |- /projx-z       # trying projx version z
          |    |    ... 
          |    |
          |    ...                 # ...add as many branches as needed
          |
          ...                      # ...create as many instances as you need
                                   # for help run:  `oetor create --help`


Development guidelines
----------------------

Main features intended:

 - [X] Easy full installation command, to get you an up an running instance in a blink.
 - [X] Download sources from Launchpad or nightly builds.
 - [X] Verify Source code version/revision numbers and retrieve updates.
 - [X] Create new server instances.
 - [X] Run tests for a server instance or modified version of it.
 - [X] Add a modules directory to an instance by simply copying it into a directory.
 - [X]  List running instances with info on the xmlrpc ports used.

Directivesi for design and code:

* Usable: the UI should be simple and intuitive.
* Simple: commands should wrap annoying tasks, and no more than that.
* Safe: repeating or misusing commands must de safe - no data is detroyed.
* Readable: reding the script code should allow to quickly understand what a command will do.

