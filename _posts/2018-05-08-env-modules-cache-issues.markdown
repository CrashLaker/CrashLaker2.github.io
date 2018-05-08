---
layout: post
title: "Environment modules cache issues"
categories: [modules]
comments: true
date: "2018-05-08 18:30:20 -0300"
---

Having issues updating your module files and nothing seems to alter in `module av`?  

It's your cache directory!  

So you have 3 alternatives:
1. Erase your cache directory
    `rm -rf ~/.lmod.d/.cache`
2. `module --ignore_cache av`
3. `export LMOD_IGNORE_CACHE=1`


voilà ;)  
