.. _example-rrdcached-systemd:

=====================================
 Systemd configuration for rrdcached
=====================================

This example sets up a dedicated rrdcached instance for munin. 

You must save it under /etc/systemd/system/rrdcached.service

Once created, you need to enable it with :

ln -s /etc/systemd/system/rrdcached.service /etc/systemd/system/multi-user.target.wants/rrdcached.service

If rrdcached stops, it is restarted. 

A pre-start script ensures we have the needed directories

A post-start script adds permissions for the munin fastcgi process. This assumes that your fastcgi
graph process is running as the user "www-data", and that the file system is mounted with "acl".
Under RHEL7/CentOS/Fedora, you should use "apache" instead of www-data.

Don't forget to adjust the rrdcached_socket to "rrdcached_socket /run/munin/rrdcached.sock" in your /etc/munin/munin.conf

::

    [Unit]
    Description=Munin rrdcached
    
    [Service]
    Restart=always
    User=munin
    PermissionsStartOnly=yes
    ExecStartPre=/usr/bin/install -d -o munin -g munin -m 0755 \
      /var/lib/munin/rrdcached-journal /run/munin
    ExecStart=/usr/bin/rrdcached \
      -g -B -b /var/lib/munin/ \
      -p /run/munin/munin-rrdcached.pid \
      -F -j /var/lib/munin/rrdcached-journal/ \
      -m 0660 -l unix:/run/munin/rrdcached.sock \
      -w 1800 -z 1800 -f 3600
    ExecStartPost=/bin/sleep 1 ; /bin/setfacl -m u:www-data:rw /run/munin/rrdcached.sock
    [Install]
    WantedBy=multi-user.target
