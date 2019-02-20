---
title: Linux cmd
date: 2016-00-00 00:00:00
tags: [cmd, find]
categories: Linux
---

# find 搜索指定目录以外的文件
find .  -path <./proc> -prune -o -name <pattern>  -print | grep -nr <key-words in file>


...