---
layout: post
title: "Starting to Work with Databases on Linux"
date: 2016-05-14 08:00:17 -0700
comments: true
author: "Kenneth Berland <ken@grnds.com>"
categories: mysql, postgres, SQL
---
## Starting to Work with Database on Linux
It irks and surprises me when I see engineers invoke `mysql` or `postgres` on their development box (i.e., localhost) in some funky way.  These include: `mysql -uroot` and `psql -Uroot`.  You should be able to work locally without pretending to be someone else.  It's just weird.

So, do yourself a favor and start right with the correct set of permissions.

### MySQL
After you `sudo apt-get install -y mysql-server-5.6`, and install without a password issue a:
`mysql -uroot -e"grant ALL on *.* to $USER" mysql`

### PostgreSQL
After you ` sudo apt-get install -y postgresql`, issue a:
`sudo su -c "psql -c\"create user $USER with SUPERUSER\"" postgres`
