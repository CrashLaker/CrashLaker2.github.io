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

Problem: User's report about NAMD lacking tcl + python.  
Solution: Install NAMD with tcl + python

`spack find -lvd namd`  
```
==> 2 installed packages.
-- linux-centos7-x86_64 / gcc@7.3.0 -----------------------------
7c5bwih    namd@2.12 fftw=3 interface=none +mpi
gvgjuab        ^charm@6.7.1 backend=mpi ~papi patches=92d507d8b72ac01f114f7772510049cc7656f93daeadef46e1f2b00ad8533d2b,e636c3f88a6944c87d910d4c8fdf25146b60f57e09228497a6cc01c7dbae9569 +shared+smp~tcp
hdjeffn            ^openmpi@3.0.1~cuda fabrics= ~java~memchecker~pmi schedulers= ~sqlite3~thread_multiple~ucx+vt
7xxgxbg                ^hwloc@1.11.9~cairo~cuda+libxml2+pci+shared
f5k35g4                    ^libpciaccess@0.13.5
etonamb                    ^libxml2@2.9.4~python
f6q324m                        ^xz@5.2.3
zknfwan                        ^zlib@1.2.11+optimize+pic+shared
52qome7                    ^numactl@2.0.11
wubl2of        ^fftw@3.3.7+double+float~fma+long_double+mpi~openmp~pfft_patches~quad simd=avx,avx2,sse2
```

```
spack install --use-cache namd@2.12 %gcc@7.3.0 fftw=3 interface=python +mpi ^openmpi@3.0.0 %gcc@7.3.0~cuda fabrics=pmi,verbs ~java schedulers=slurm ~sqlite3~thread_multiple~ucx+vt ^hwloc@1.11.9~cairo~cuda+libxml2+pci+shared arch=linux-centos7-x86_64
```  

`spack module refresh --delete-tree -m tcl -y namd`  
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
