apache2
=======

This rule aims to configure Apache 2.4+ on the supported Ubuntu.

Role Variables
--------------

Available variables are listed below, along with default values (see `defaults/main.yml`):

Which modules to enable or disable, determined by `state` (defaults to 'present'). See the `mods-available` directory inside the apache configuration directory (`/etc/apache2/mods-available`) for all the available modules.
We use the `community.general.apache2_module` for making these changes. The MPM modules does not normally need to be specified here, as they are added or removed automatically depending on the `apache2_mpm` variable.

    apache2_modules:
      - name: alias
      - name: cgid
        state: absent
      - name: rewrite
        state: present

Add a set of properties per virtualhost, including `servername` (required), `documentroot` (required), `allow_override` (optional: defaults to the value of `apache2_allow_override`), `options` (optional: defaults to the value of `apache2_options`), `serveradmin` (optional), `serveralias` (optional: this has to be a list, even with a single value) and `extra_parameters` (optional: this is a list, with one line per entry. you can add whatever additional configuration lines you'd like in here), `separate_logs` (optional: true if you want for example example.com-{access,error}.log, false for {access,error}.log)

For SSL you can define `ssl_key` (required), `ssl_cert` (required) and `ssl_chain` (optional). You will need to ensure the files are populated before configuring them.

There is also a special value, template, which default to `apache2_vhost_template`: 'vhost.conf.j2'. Using this option you are free to specify whatever variables you support using your custom template.

    apache2_enabled_sites:
      example.com:
        servername: www.example.com
        serveralias:
          - example.com
        serveradmin: 'admin@example.com'
        documentroot: /var/www/html
        allow_override: None
        options: -Indexes -FollowSymlinks
        extra_parameters:
          - RewriteCond %{HTTP_HOST} !^www\. [NC]
          - RewriteRule ^(.*)$ http://www.%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
        template: vhost.conf.j2
    apache2_disabled_sites:
      example.net:
        servername: www.example.net
        documentroot: /var/www/example.net

On Debian/Ubuntu, a default virtualhost that basically say the server works is included in the packaging of Apache.
You will need to handle these yourself, possibly using the above variables.

Set apache state when configuration changes are made.
Recommended values: `restarted` or `reloaded`

    apache2_restart_state: restarted

The variables here are used to set `/etc/apache2/conf-available/security.conf`. `apache2_server_filesystem` basically sets a DirectoryMatch for `/` that denies access to all files not specifically configured later. Notice by default there are no restrictions because it will break the configuration of some web application Debian packages.

There is also a special variable, `apache2_server_security` where you can specify configuration to be added to the bottom of the file.

For the rest of the variables, please refer to the file to see the available options.

    apache2_server_filesystem: true
    apache2_server_tokens: 'OS'
    apache2_server_signature: 'On'
    apache2_server_trace_enabled: 'Off'
    apache2_server_security: |-
      <DirectoryMatch "\/.git">
         AllowOverride None
         Require all denied
      </DirectoryMatch>

We can set MPM type we want to use, and change the default variables taken from a basic Apache installation on Ubuntu 20.04.3 LTS.

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
      - name: mpm_event
      - name: mpm_prefork
      - name: mpm_worker

Example Playbook
----------------

    - hosts: webservers
      vars:
        apache2_modules:
          - rewrite
        apache2_enabled_sites:
          example.com:
            servername: www.example.com
            serveralias:
              - example.com
            documentroot: /var/www/example.com
            extra_parameters: |
              RewriteCond %{HTTP_HOST} !^www\. [NC]
              RewriteRule ^(.*)$ http://www.%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
          custom.example.com:
            servername: custom.example.com
            documentroot: /var/www/custom.example.com
            template: custom_template.conf.j2
            customvar: "This only works in the custom_template.conf.j2"
        apache2_vhost_disabled_sites:
          example.net:
            servername: www.example.net
            serveralias:
              - example.net
            documentroot: /var/www/example.net
      roles:
        - vcc_caeit.apache2

License
-------

GPLv2

Author Information
------------------

This role was created in 2018 by Nafallo Bjälevik, whilst doing consultancy work for [Volvo Cars](http://www.volvocars.com/).
