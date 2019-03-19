Ansible Role: apache2
=========

This rule aims to configure Apache 2.4+ on the supported Ubuntu LTSes (however not ESM).

When creating this role I used [geerlingguy.apache](https://github.com/geerlingguy/ansible-role-apache) and [pulse-mind.ansible-role-apache-vhosts](https://github.com/pulse-mind/ansible-role-apache-vhosts) for inspiration, but none of them went all the way where I wanted to end up.

Role Variables
--------------

Available variables are listed below, along with default values (see `defaults/main.yml`):

Which modules to enable or disable. See the `mods-available` directory inside the apache configuration directory (`/etc/apache2/mods-available`) for all the available modules.
We use the `apache2_module` for making these changes. Do NOT specify MPM modules in this section, they require special handling.

    apache2_mods_enabled: []
    apache2_mods_disabled: []

Add a set of properties per virtualhost, including servername (required), documentroot (required), allow_override (optional: defaults to the value of apache2_allow_override), options (optional: defaults to the value of apache2_options), serveradmin (optional), serveralias (optional: this has to be a list, even with a single value) and extra_parameters (optional: this is a list, with one line per entry. you can add whatever additional configuration lines you'd like in here), separate_logs (optional: true if you want for example example.com-{access,error}.log, false for {access,error}.log)

There is also a special value, template, which default to apache2_vhost_template: 'vhost.conf.j2'. Using this option you are free to specify whatever variables you support using your custom template.

    apache2_vhost_sites:
      example.com:
        servername: www.example.com
        serveralias:
          - example.com
        serveradmin: "admin@example.com"
        documentroot: "/var/www/html"
        allow_override: "None"
        options: "-Indexes -FollowSymlinks"
        extra_parameters:
          - RewriteCond %{HTTP_HOST} !^www\. [NC]
          - RewriteRule ^(.*)$ http://www.%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
        template: "vhost.conf.j2"
    apache2_vhost_remove_sites:
      - example.net

On Debian/Ubuntu, a default virtualhost that basically say the server works is included in the packaging of Apache.
Set this to `true` to disable that default (it will still be available in /etc/apache2/sites-available).

    apache2_vhost_disable_default: false

Set apache state when configuration changes are made.
Recommended values: `restarted` or `reloaded`

    apache2_restart_state: 'restarted'

The variables here are used to set `/etc/apache2/conf-available/security.conf`. Please refer to the file to see the options.

    apache2_server_tokens: 'OS'
    apache2_server_signature: 'On'
    apache2_server_trace_enabled: 'Off'

We can set MPM type we want to use, and change the default variables taken from a basic Apache installation on Ubuntu 16.04.3 LTS.

    apache2_mpm: 'event'
    apache2_mpm_startservers: 2
    apache2_mpm_minspareservers: 5
    apache2_mpm_maxspareservers: 10
    apache2_mpm_minsparethreads: 25
    apache2_mpm_maxsparethreads: 75
    apache2_mpm_threadlimit: 64
    apache2_mpm_threadsperchild: 25
    apache2_mpm_serverlimit: 512
    apache2_mpm_maxrequestworkers: 150
    apache2_mpm_maxconnectionsperchild: 0

In `vars/main.yml` we can find a list of which MPMs are included in the default Ubuntu packaging. We use this list to determine which ones to disable based on which one the administrator have selected.

    apache2_mpms:
      - event
      - prefork
      - worker

Example Playbook
----------------

    - hosts: webservers
      vars:
        apache2_mods_enabled:
          - rewrite
        apache2_vhost_sites:
          example.com:
            servername: www.example.com
            serveralias: example.com
            extra_parameters: |
              RewriteCond %{HTTP_HOST} !^www\. [NC]
              RewriteRule ^(.*)$ http://www.%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
          custom.example.com:
            servername: custom.example.com
            template: custom_template.conf.j2
            customvar: "This only works in the custom_template.conf.j2"
        apache2_vhost_remove_sites:
          - begone.example.com
        apache2_vhost_disable_default: true
      roles:
        - vcc-caeit.apache2

License
-------

GPLv2

Author Information
------------------

This role was created in 2018 by Nafallo Bjälevik, whilst doing consultancy work for [Volvo Cars Corporation](http://www.volvocars.com/).
