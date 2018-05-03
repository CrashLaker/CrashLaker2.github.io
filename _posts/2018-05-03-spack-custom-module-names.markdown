---
layout: post
title: "Spack custom module names"
categories: [spack, modules]
comments: true
date: "2018-05-03 20:12:20 -0300"
---

Compiling on SPACK with custom dependencies  

Spack [bug](http://spack.readthedocs.io/en/latest/known_issues.html#default-variants-are-not-taken-into-account-during-concretization) reported  
---
`spack install --use-cache namd@2.12 %gcc@7.3.0 fftw=3 interface=python +mpi ^openmpi@3.0.0 %gcc@7.3.0~cuda fabrics=pmi,verbs ~java schedulers=slurm ~sqlite3~thread_multiple~ucx+vt ^hwloc@1.11.9~cairo~cuda+libxml2+pci+shared arch=linux-centos7-x86_64`  

`spack module refresh --delete-tree -m tcl -y`  
```
==> Error: Name clashes detected in module files:

file: /root/spack/share/spack/modules/linux-centos7-x86_64/namd/2.12-gcc-7.3.0
spec: namd@2.12%gcc@7.3.0 fftw=3 interface=none +mpi arch=linux-centos7-x86_64 
spec: namd@2.12%gcc@7.3.0 fftw=3 interface=python +mpi arch=linux-centos7-x86_64 `
```

`cat /root/.spack/modules.yaml`  
```
modules:
  tcl:
    hash_length: 0
    naming_scheme: '${PACKAGE}/${VERSION}-${COMPILERNAME}-${COMPILERVER}'
    all:
        suffixes:
            ^cuda: 'cuda'
    namd:
        suffixes:
            'interface=python': 'tcl'
    gromacs:
      suffixes:
        '~cuda~double+epyc~mpi~plumed+shared': epyc
```

`spack module refresh --delete-tree -m tcl -y`  
`module av`  
```
namd/2.12-gcc-7.3.0-tcl
namd/2.12-gcc-7.3.0
```
