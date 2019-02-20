---
title: Sqlite
date: 2016-07-28 19:11:33
tags: [db, sqlite]
categories: sqlite
---

# Sqlite3 在Android shell中操作
adb shell
cd /data/data/com.android.providers.securityguard/databases
sqlite3 securityguard.db
.table
select * from SystemTaskPerm;



INSERT INTO SystemTaskPerm (id, 'packagename', 'cmd', 'args', 'reserved', auth) VALUES (4, 'com.a.b', 'ip' , '', '', 1);

