---
title: BASH tricks
date: 2016-08-02 14:11:47
tags:
---

# Just a bunch of bash tricks i’ve picked up.

Save a backup copy of a file with a datestamp.

``` bash
cp script.sh script.sh.$(date +%Y%m%d%H%M)
```

Generate a random string maybe for api key or password.

``` bash
cat /dev/urandom | env LC_CTYPE=C tr -dc 'a-zA-Z0-9' | fold -w 32 | head -n 1
```

Find empty directories.

``` bash
find . -mindepth 1 -maxdepth 1 -type d -empty
```

List processes being used by remote terminals and then show the files that may be opened by them. Good for tracking down what another ssh user might be doing .

``` bash
for i in $(ps aux |grep pts |awk '{print $2}');do lsof -p $i;done
```

grep out IP addresses from a log file.

``` bash
grep -E -o '(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)' access.log
```

Fping to show unused IP’s on a subnet.

``` bash
fping -r1 -g 192.168.1.0/24 2> /dev/null | grep unreachable | cut -f1 -d' '
```

arping a /24 subnet to find alive hosts.

``` bash
for i in `seq 1 254` ; do arping -c 1 192.168.1.$i | grep reply ; done
```

