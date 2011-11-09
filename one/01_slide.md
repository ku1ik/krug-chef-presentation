!SLIDE center

# **C H E F**

or how to make computers do the work for us

<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>

Marcin Kulik, _Lunar Logic Polska_

KRUG 2011/11/08

!SLIDE

Everyday we're dealing with mechanical, repetitive tasks... we can automate.

!SLIDE subsection

# What is Chef?

!SLIDE

# Automation tool
# written in **Ruby**

!SLIDE

# DSL

!SLIDE

# Created by Opscode

!SLIDE smbullets incremental small

* "Chef is an open source systems integration framework built to bring the
benefits of configuration management to your entire infrastructure."

* "You _write source code_ to describe how you want each part of your
infrastructure to be built, then _apply_ those descriptions to your _servers_."

* "The result is a fully automated infrastructure: when a new server comes on
line, the only thing you have to do is tell Chef what role it should play in
your architecture."

!SLIDE subsection

# Why do you need it?

!SLIDE

## Economics **+** Efficiency **+** Scalability

!SLIDE subsection

# Terms

!SLIDE

# Node

remote server, local machine...

!SLIDE

# Role

web server, database server, **ruby** _dev workstation_...

!SLIDE

# Cookbook

mysql, ssh-access, _dotfiles_...

!SLIDE

# Recipe

install mysql-server, create database, add user...

!SLIDE

# Resource

file, dir, user, package, service, gem, virtual host...

!SLIDE

# Run list

list of recipes to run in order

!SLIDE

    @@@ javascript
    {
      "run_list": [
        "recipe[mysql]",
        "recipe[git]",
        "recipe[ruby19]"
      ]
    }

!SLIDE smaller

# Cookbook structure

<pre>
  |-- config
  |   |-- node.json
  |   `-- solo.rb
  |-- <em>cookbooks</em>
  |   |-- <em>book1</em>
  |   |   |-- attributes
  |   |   |-- files
  |   |   |-- metadata.rb
  |   |   |-- <em>recipes</em>
  |   |   |   |-- <em>default.rb</em>
  |   |   |   `-- source.rb
  |   |   `-- templates
  |   |-- book2
  |   |   |-- attributes
  |   |   |   `-- default.rb
  |   |   |-- files
  |   |   |-- recipes
  |   |   |   `-- default.rb
  |   |   `-- templates
  |   |       `-- default
  |   |           `-- authorized_keys.erb
  |   |-- book3
  |   |   |-- attributes
  |   |   |-- files
  |   |   |   `-- default
  |   |   |       `-- secret-key
  |   |   |-- recipes
  |   |   |   `-- default.rb
  |   |   `-- templates
  |-- config
  |   |-- node.json
  |   `-- solo.rb
  |-- cookbooks
  |   |-- book1
  |   |   |-- attributes
  |   |   |-- files
  |   |   |-- metadata.rb
  |   |   |-- recipes
  |   |   |   |-- default.rb
  |   |   |   `-- libs.rb
  |   |   `-- templates
  |   |-- book2
  |   |   |-- attributes
  |   |   |   `-- default.rb
  |   |   |-- files
  |   |   |-- recipes
  |   |   |   `-- default.rb
  |   |   `-- templates
  |   |       `-- default
  |   |           `-- authorized_keys.erb
  |   |-- book3
  |   |   |-- attributes
  |   |   |-- files
  |   |   |   `-- default
  |   |   |       `-- secret-key
  |   |   |-- recipes
  |   |   |   `-- default.rb
  |   |   `-- templates
</pre>

!SLIDE subsection

# Installation

!SLIDE

    $ gem install chef

!SLIDE subsection

# Modes of operation

!SLIDE

# Cookbooks stored
# in central repository

(free cookbooks hosting by Opscode: https://manage.opscode.com/)

!SLIDE

    $ sudo chef-client

!SLIDE

# Cookbooks stored
# on the node

!SLIDE small

    $ sudo chef-solo -c /path/to/cfg.rb
                     -j /path/to/node-data.json

!SLIDE subsection

# Use cases

!SLIDE

# Configure new machine
# (in the cloud with Knife)

Amazon EC2, Engine Yard, Linode, BrightBox...

!SLIDE

# Manage config of existing
# company servers

Client demo apps (directory, vhost, god config), developers' ssh keys...

!SLIDE

# Bootstrap _workstation_!

rvm + **ruby** 1.9, git, mysql, vim/emacs...

!SLIDE

Enough with theory!

!SLIDE subsection

# Lunar Station

https://github.com/LunarLogicPolska/lunar-station

!SLIDE

Lunar Station is a set of _Chef cookbooks_ and a _bash script_ (???) for
bootstrapping developers machines at Lunar Logic Polska.

!SLIDE

# You need **ruby** to run Chef

!SLIDE

# (We assume) you use RVM

No need for system **ruby** for ruby devs nowadays

!SLIDE code

# bootstrap.sh

!SLIDE bullets incremental

* detects platform (Ubuntu, Fedora, OSX)
* installs compilers and other RVM dependencies
* installs RVM & **ruby** 1.9 & chef gem
* downloads latest Lunar Station cookbooks
* runs chef-solo

!SLIDE commandline incremental

    $ curl -skL http://bit.ly/lunar-station | bash
    Initializing Lunar Workstation...
    >> Fedora Linux detected.
    >> Checking for RVM...
    >> Fetching latest version of Lunar Station cookbooks...
    >> Starting chef-solo run...
    [Mon, 07 Nov 2011 22:19:54 +0100] INFO: *** Chef 0.10.4 ***
    [Mon, 07 Nov 2011 22:19:54 +0100] INFO: Setting the run_list to 
    ...

!SLIDE

# Nodes

!SLIDE small

    @@@ javascript
    # linux-rubydev.json

    {
      "run_list": [ "role[rubydev]" ]
    }

!SLIDE small

    @@@ javascript
    # osx-rubydev.json

    {
      "run_list": [ "role[osx]", "role[rubydev]" ]
    }

!SLIDE

# Roles

!SLIDE small

    @@@ ruby
    # base.rb

    run_list 'recipe[repos]',   'recipe[curl]',
             'recipe[wget]',    'recipe[git]',
             'recipe[libxml2]', 'recipe[ack]',
             'recipe[vim]',     'recipe[ctags]',
             'recipe[skype]',   'recipe[firefox]' ,
             'recipe[google-chrome]'

!SLIDE

    @@@ ruby
    # rubydev.rb

    run_list 'role[base]', 'recipe[mysql]'

!SLIDE

    @@@ ruby
    # osx.rb

    run_list "recipe[homebrew]"

!SLIDE

# Cookbooks

!SLIDE

# _repos_ cookbook

!SLIDE smaller

    @@@ ruby
    # cookbooks/repos/recipes/default.rb

    case node[:platform]
    when 'fedora'
      path = "/tmp/rpmfusion-free-release-stable.noarch.rpm"

      bash "download rpmfusion free package" do
        code "wget http://download1.rpmfusion.org/.../" +
          "rpmfusion-free-release-stable.noarch.rpm -O #{path}"

        not_if { File.exist?(path) }
      end

      package "rpmfusion-free-release-stable" do
        source path
        options "--nogpgcheck"
      end

    when 'ubuntu'
      ...
    end

!SLIDE smaller

    @@@ ruby
    # cookbooks/repos/recipes/default.rb

    case node[:platform]
    when 'fedora'
      ...

    when 'ubuntu'
      bash "enable multiverse repo" do
        code "head -n 1 /etc/apt/sources.list | " +
          "sed 's/main universe/multiverse/' " +
          ">> /etc/apt/sources.list"

        not_if "egrep '^deb.+multiverse' /etc/apt/sources.list"
      end
    end

!SLIDE

# _vim_ cookbook

!SLIDE smaller

    @@@ ruby
    # cookbooks/vim/recipes/default.rb

    case node[:platform]
    when "ubuntu"
      package "vim"
      package "vim-gnome"

    when "fedora"
      package "vim-enhanced"
      package "vim-X11"

    when 'mac_os_x'
      package "macvim"
    end

!SLIDE

# _skype_ cookbook

!SLIDE smaller

    @@@ ruby
    # cookbooks/skype/recipes/default.rb

    case node[:platform]
    when 'ubuntu'
      include_recipe 'init::ubuntu' # for partner repo

      package 'skype'

    when 'mac_os_x'
      dmg_package "Skype" do
        source "http://www.skype.com/go/getskype-macosx.dmg"
        action :install
      end

    when 'fedora'
      ...
    end

!SLIDE subsection

# Lunar Kitchen

!SLIDE

Source of LLP servers configuration data and a set of Chef cookbooks

!SLIDE

# chef-solo invoked on remote machines

# no chef server

!SLIDE

Each server we configure has its corresponding node configuration file in
nodes/ directory of **kitchen** project that specifies *run_list* and few other
settings

!SLIDE smaller

    @@@ javascript
    # nodes/deneb.json

    {
      "run_list": [ "recipe[ssh_access]" ],

      "ssh_access": [ "marcin.kulik", "anna.lesniak", ...],

      "opened_ports": {
        "tcp": [80, 443, 22, 8080],
        "udp": []
      },
      ...

!SLIDE

# How do we run chef-solo on remote machine?

!SLIDE

# Capistrano!

!SLIDE

    # See the list of configured servers:

    $ cap -T


    # Make the changes happen on the server:

    $ cap configure:deneb

!SLIDE

# How does Capfile look like?

!SLIDE smaller

    @@@ ruby
    set :user, 'chef'

    NODE_LIST = Dir["nodes/*.json"].map do |nodefile|
      File.basename(nodefile, '.json')
    end

    NODE_LIST.each do |node|
      role node.to_sym, node
    end

    NODE_CONFIG = <<-EOS
      file_cache_path '/tmp/chef-solo'
      cookbook_path '/tmp/chef-solo/cookbooks'
      role_path '/tmp/chef-solo/roles'
    EOS
    ...

!SLIDE smaller

    @@@ ruby
    ...
    namespace :configure do
      NODE_LIST.each do |node|
        desc "Configure #{node}"
        task node.to_sym, :roles => node.to_sym do
          run "if [ ! -e /tmp/chef-solo ]; then mkdir /tmp/chef-solo; fi"
          upload("cookbooks", "/tmp/chef-solo/", :via => :scp, :recursive => true)
          upload("roles", "/tmp/chef-solo/", :via => :scp, :recursive => true)
          upload("nodes/#{node}.json", "/tmp/chef-solo/node.json", :via => :scp)
          put(NODE_CONFIG, "/tmp/chef-solo/solo.rb")
          run "rvmsudo chef-solo " +
                       "-c /tmp/chef-solo/solo.rb " +
                       "-j /tmp/chef-solo/node.json"
        end
      end
    end

!SLIDE

# SSH access

!SLIDE small

    ├── Capfile
    ├── config
    ├── cookbooks
    ├── nodes
    ├── README.md
    ├── roles
    └── ssh_keys
        ├── anna.lesniak
        ├── artur.bilski
        ├── ...
        └── marcin.kulik

!SLIDE smaller

    @@@ ruby
    # cookbooks/access/recipes/default.rb

    username = 'dev'

    ssh_keys = node[:ssh_access].map do |f|
      File.read("/tmp/chef-solo/ssh_keys/#{f}")
    end

    template "/home/#{username}/.ssh/authorized_keys" do
      source "authorized_keys.erb"
      owner username
      group 'users'
      mode "0600"
      variables :ssh_keys => ssh_keys
    end

!SLIDE small

    # cookbooks/access/templates/authorized_keys.erb


    # Generated by Chef, do not edit!

    <%= @ssh_keys.join("\n") %>

!SLIDE subsection

# Tips

!SLIDE

# Learn step by step

EC2 + Chef + Knife + Opscode... = Fuuuuuuuuuuuuuuuuuuuuu

!SLIDE

# Start with _chef-solo_

!SLIDE

# Run on local machine

Easy to troubleshoot problems

!SLIDE

# Use Vagrant

http://vagrantup.com/

<br>

Great for testing cookbooks - doesn't pollute your system

!SLIDE subsection

# Q?

!SLIDE

# Thanks!

marcin.kulik@llp.pl _|_ @sickill _|_ https://github.com/sickill
