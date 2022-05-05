---
title: Tricks for finding directory space usage.
date: 2016-07-14 14:21:36
tags:
author:      "Benjamin Rizkowsky"
---


# Tricks for finding directory space usage.

``` bash
find . -iname "*regex" -printf "%s\n" | awk '{f+=$1}END{print f/(1024 *1024 * 1024),"GB"}'
```

``` bash
find . -iname "*.regex" -printf "%s\n" | awk '{f+=$1}END{print f}'
```

another way.

``` bash
find -name \*.regex  -print0 | du -ch --files0-from=- |tail -1
```

``` bash
du -ch -b --max-depth=1 |sort -n|awk '{ print $1/(1024 *1024 * 1024),"gb", $2 }'
```

Heres almost the same command excluding anything requiring special notation close to zero.

``` bash
du -ch -b --max-depth=1 |sort -n|awk '{ print $1/(1024 *1024 * 1024),"gb", $2 }' |grep -v e-0
```

another way.

``` bash
du -h | perl -e 'sub h{%h=(K=>10,M=>20,G=>30);($n,$u)=shift=~/([0-9.]+)(\D)/;return $n*2**$h{$u}}print sort{h($b)<=>h($a)}<>;'
```

another way.

``` bash
du -ch --max-depth=1 --time | perl -e 'sub h{%h=(K=>10,M=>20,G=>30);($n,$u)=shift=~/([0-9.]+)(\D)/;return $n*2**$h{$u}}print sort{h($b)<=>h($a)}<>;'
```

