---
title: My personal Linux commands cheatsheet
date: 2019-01-18T13:30:48+01:00
draft: false
tags:
  - Archlinux
  - Linux
  - Bash
categories:
  - Linux
description: >-
  This is a collection of Linux commands that I use regularly. I present some tools, together with commands I use. I would be happy if you could use some of these!
---

This is a collection of Linux commands that I use regularly. I present some tools, together with commands I use. I would be happy if you could use some of these!

## Reflector

Reflector is a script that allows you to download the latest mirror list from the MirrorStatus page, filter it by the latest mirrors, and sort it by speed.

Sort the 50 most recent HTTP mirror servers  by download rate and save the new mirror list with running progress messages:

```
reflector --verbose --latest 50 --sort rate --protocol https --save mirrorlist
```

**Info:** This article is updated periodically and will probably never be complete. :-)
