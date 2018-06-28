---
#layout: single
comments: true
title: Build a Docker Stack with Ansible Container
---

A great way to test a prototype, app or configuration is by using it in a container. If you decide it does not work for whatever reason then you can simply delete the container without risk of altering your work environment. You may however want to take the next step and implement different environments (production and development for example). With this comes the supporting requirements such as a database server or a load balancer. 

This is where configuration management comes in, you define the recipe to get your container from a clean image to its final usable state. This comes with some additional benefits too such as:
  - Consistency across environments - If it runs on one copy of the container then it will run on all the others
  - Predictable and Repeatable Deploys - By enforcing the specifically what is installed and how it is installed you remove unintended and unexpected issues
  - Easy and fast to deploy once configured - Creating the same container again is a matter of running a single command in most cases
 
In this post we will set up Ansible, Ansible-Container and Docker locally to create a container containing Apache, PHP and sync a directory containing a HTML/PHP app of your choosing.

# What do these software packages do?

Here is an explanation of the different software packages with a reason as to why they are used:
- **Ansible** (https://www.ansible.com) - An orchestration, provisioning and configuration management tool. It allows configuration of systems against playbooks written in YAML syntax that can scale from the simple to large and convuluted systems with relative ease.
- **Ansible-Container** (https://www.ansible.com/integrations/containers/ansible-container) - A useful addon to Ansible that allows direct orchestration of containers such as Docker and Openshift (Ansible alone uses SSH to orchestrate which is not advised for Docker). It also has the capability to deal with large scale container orchestration by building container stacks.
- **Docker** (https://www.docker.com/what-docker) - A container platform that allows the compartmentalisation of an application or environment. Unlike traditional Virtual Machines it is relatively lightweight while providing the many benefits that come with running a Virtual machine.

# Software Versions

There are many tutorials across the web on how to build containers and many of them are very useful. I found that most of them did not state what versions were used which can make them hard to follow. This area is moving very fast and new versions are not only released on a fast basis, the core functionality changes very rapidly and what was able to be run in a previous version has to be changed or modified.

So here are the versions that are used for this tutorial:

  - Ansible: 2.5.4
  - Docker version 17.05.0-ce, build 89658be
  - Python 2.7.9
  - Pip: 10.0.1
  - pipenv 2018.05.18
   
  
 And inside the pip virtual environment (pipenv)
  - Ansible Container: 0.9.2
  - Pip: 9.0.3
  
The bare metal Operating system this is run on is Ubuntu 14.04.5 LTS 

# Installing the environment

First thing to do is ensure Python, pip and pipenv are installed.

{% highlight console %}
sudo apt-get update
sudo apt-get install python2.7 python-pip
sudo pip install pipenv
{% endhighlight %}

To get the latest version of Ansible we need to add the PPA (Personal Package Archive) repository which contains Ansible. Then we can install the latest version of Ansible and Ansible-Container.

{% highlight console %}
sudo apt-get install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible
{% endhighlight %}

Now you need to create a directory to run your virtual environment in with all the configuration files required (I created a directory inside my home directory for this example). Finally create the virtual environment with pipenv:

{% highlight console %}
mkdir ~/apache-php-container
cd ~/apache-php-container
pipenv install
{% endhighlight %}

# Ansible Container Environment Setup

Now we need to get into the virtual environment, run the following:
{% highlight console %}
pipenv shell
{% endhighlight %}

At the time of writing this there was a bug that prevented ansible-container from installing with the latest version of pip (10.0.1 being the latest). We needed to revert to an older version (9.0.3 was the last) before installing. Additionally the latest docker version causes some incompatibility (version 3.3.0). We install docker version 2.7.0 which is known to work.
{% highlight console %}
pip install --force-reinstall pip==9.0.3
pip install docker==2.7.0
pip install ansible-container[docker]
{% endhighlight %}

Then initialise the ansible-container project.
{% highlight console %}
ansible-container init
{% endhighlight %}

The following files are created inside your project directory:

{% highlight console %}
ansible.cfg
ansible-requirements.txt
container.yml
meta.yml
requirements.yml
{% endhighlight %}

Visit the Ansible Container getting started guide for details about each of the files just created. (https://docs.ansible.com/ansible-container/getting_started.html#dipping-a-toe-in-starting-from-scratch) 


# Ansible Container Configuration

We want to tell Ansible how to configure the container so lets make the role directories for this.

{% highlight console %}
mkdir -p roles/webserver/tasks roles/webserver/files html
{% endhighlight %}

The main definitions in Ansible container are held in the **./container.yml** file in the root directory of your project. For this example it will contain the below.

{% highlight yml %}
version: "2"
settings:
  conductor:
    base: "debian:jessie"
  project_name: "apache-php"

services:
  webserver:
    from: "debian:jessie"
    roles:
      - role: "webserver"
    ports:
      - "80:80"
    volumes:
      - ${PWD}/html:/var/www/html:rw,delegated
    command: ["/usr/sbin/apachectl", "-D", "FOREGROUND"]
{% endhighlight %}

What this does is create a conductor container with Debian Jessie, after which a *webserver* service is created (also with Debian Jessie). The *webserver* role contains instructions on installing Apache2 and PHP on the container. Port 85 is mapped externally and the webserver directory is mapped to the /html directory in the project directory.

It is advised to use the same flavour of linux for the conductor and the services so that the conductor can maintain similar if not the same dependencies when provisioning.

Next we open **./roles/webserver/tasks/main.yml** and insert the following.

{% highlight yml %}
---
- name: Update apt cache
  apt:
    update_cache: yes
    cache_valid_time: 600

- name: Install apt requirements
  apt:
    name: "{{ item }}"
    state: present
  with_items:
    - "locales"
    - "apache2"
    - "php5"
    - "php5-cli"
    - "php5-common"
    - "php5-gd"
    - "php5-json"
    - "php5-ldap"
    - "php5-mysql"
    - "php5-pgsql"
    - "libapache2-mod-php5"

- name: Delete index.html from default directory
  file:
    path: /var/www/html/index.html
    state: absent

- name: Copy apache2 configuration
  copy:
    src: apache_config
    dest: /etc/apache2/sites-available/000-default.conf
{% endhighlight %}

This will install the webserver in addition to removing the example *index.html* file from the fresh install and copy over a virtual host configuration.

Now lets go to **./roles/webserver/files/apache_config** and create a virtual host configuration for use in our container.

{% highlight xml %}
<VirtualHost *:80>
    # The ServerName directive sets the request scheme, hostname and port that
    # the server uses to identify itself. This is used when creating
    # redirection URLs. In the context of virtual hosts, the ServerName
    # specifies what hostname must appear in the request's Host: header to
    # match this virtual host. For the default virtual host (this file) this
    # value is not decisive as it is used as a last resort host regardless.
    # However, you must set it for any further virtual host explicitly.
    #ServerName www.example.com

    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html

    <Directory /var/www/html>
        AllowOverride All
        Require all granted
    </Directory>

    # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
    # error, crit, alert, emerg.
    # It is also possible to configure the loglevel for particular
    # modules, e.g.
    #LogLevel info ssl:warn

    ErrorLog /dev/stdout
    CustomLog /dev/stdout combined

    # For most configuration files from conf-available/, which are
    # enabled or disabled at a global level, it is possible to
    # include a line for only one particular virtual host. For example the
    # following line enables the CGI configuration for this host only
    # after it has been globally disabled with "a2disconf".
    #Include conf-available/serve-cgi-bin.conf
</VirtualHost>
{% endhighlight %}

Finally to check if our container is up and working lets create a test file.

Go to **./html/index.php** and insert the following:

{% highlight php %}
<?php
phpinfo();
?>
{% endhighlight %}

# Building and Running the Project

Now we are ready to build the project. Ensure you are in the root directory of your project and inside your virtual environment (if used) and type the following:

{% highlight console %}
ansible-container build
{% endhighlight %}

This will fully provision your project into docker containers but not run them. To do that run the following:

{% highlight console %}
ansible-container run
{% endhighlight %}

Open up a browser and go to your machines IP. You should now see the result of phpinfo() on screen :)

# Stopping the Project and Cleaning Up

Once you are finished you will want to stop the stack. You can do this by running the following command:

{% highlight console %}
ansible-container stop
{% endhighlight %}

If you want to get rid of all generated Docker images and containers then run the following:

{% highlight console %}
ansible destroy
{% endhighlight %}

# Thats it!

This project is up on Github and Ansible Galaxy. Feel free to use this example and change it to your needs.

To pull down this example from Ansible Galaxy type in the following:

{% highlight console %}
ansible-container init adamstraube.Ansible-Container-Webserver-Demo
{% endhighlight %}
Then go through the instructions above from **Building and Running the Project**
