# Scripts used in testing MOM6 commits against the various regression repositories

- These scripts are not for general consumption - they are mostly written in gmake so good luck figuring out how they work! :)
  But seriously they are written for a particular sequence of tests assuming a specific setup and so they really will not be much use for general application.

## Build executable

From top-level of MOM6-examples

```bash
make -f tools/MRS/Makefile.build repro_gnu -s -j
make -f tools/MRS/Makefile.build repro_intel -s -j
make -f tools/MRS/Makefile.build repro_pgi -s -j
```

## Run model

```bash
make -f tools/MRS/Makefile.run gnu_all MEMORY=dynamic_symmetric -s -j
make -f tools/MRS/Makefile.run intel_all MEMORY=dynamic_symmetric -s -j
make -f tools/MRS/Makefile.run pgi_all MEMORY=dynamic_symmetric -s -j
```
should yield a clean MOM6-examples (uses the correct layouts).

Test the non-symmetric executables
```bash
make -f tools/MRS/Makefile.run gnu_all MEMORY=dynamic_nonsymmetric -s -j
make -f tools/MRS/Makefile.run intel_all MEMORY=dynamic_nonsymmetric -s -j
make -f tools/MRS/Makefile.run pgi_all MEMORY=dynamic_nonsymmetric -s -j
```
will produce difference MOM_parameter_doc.layout files in MOM6-examples but with the right answers.

To use static executables:
```bash
make -f tools/MRS/Makefile.run gnu_static_ocean_only MEMORY=static -s -j
make -f tools/MRS/Makefile.run intel_static_ocean_only MEMORY=static -s -j
make -f tools/MRS/Makefile.run pgi_static_ocean_only MEMORY=static -s -j
```
Test with alternative PE counts
```bash
make -f tools/MRS/Makefile.run gnu_all -s -j LAYOUT=alt
make -f tools/MRS/Makefile.run intel_all -s -j LAYOUT=alt
make -f tools/MRS/Makefile.run pgi_all -s -j LAYOUT=alt
```

## Copy results to regressions/
```bash
make -f tools/MRS/Makefile.sync -s -k
```
will sync the newly generated ocean/seaice.stats files and report their status.

```bash
make -f tools/MRS/Makefile.sync -s -k gnu
```
will sync only the gnu stats files.


## Test restarts

```bash
make -f tools/MRS/Makefile.restart gnu_ocean_only -s -j RESTART_STAGE=02
make -f tools/MRS/Makefile.restart gnu_ocean_only -s -j RESTART_STAGE=01
make -f tools/MRS/Makefile.restart gnu_ocean_only -s -j RESTART_STAGE=12
make -f tools/MRS/Makefile.restart gnu_ice_ocean_SIS2 -s -j RESTART_STAGE=02
make -f tools/MRS/Makefile.restart gnu_ice_ocean_SIS2 -s -j RESTART_STAGE=01
make -f tools/MRS/Makefile.restart gnu_ice_ocean_SIS2 -s -j RESTART_STAGE=12
make -f tools/MRS/Makefile.restart restart_gnu_ocean_only -s -j
make -f tools/MRS/Makefile.restart restart_gnu_ice_ocean_SIS2 -s -j
make -f tools/MRS/Makefile.restart gnu_ocean_only gnu_ice_ocean_SIS2 -s -j RESTART_STAGE=02
make -f tools/MRS/Makefile.restart gnu_ocean_only gnu_ice_ocean_SIS2 -s -j RESTART_STAGE=01
make -f tools/MRS/Makefile.restart gnu_ocean_only gnu_ice_ocean_SIS2 -s -j RESTART_STAGE=12
make -f tools/MRS/Makefile.restart restart_gnu_ocean_only restart_gnu_ice_ocean_SIS2 -s -j
```
Last commands alone is sufficient but seems more susceptible to lustre file systems flakiness.

## Build coverage report

```bash
make -f MRS/Makefile.coverage gnu_ocean_only -s MEMORY=dynamic_symmetric
make -f MRS/Makefile.coverage gnu_ice_ocean_SIS2 -s
make -f MRS/Makefile.coverage coverage
```

```bash
make -f tools/MRS/Makefile.run gnu_all MEMORY=dynamic_symmetric -s -j && make -f tools/MRS/Makefile.run intel_all -s -j && make -f tools/MRS/Makefile.run all MEMORY=dynamic_symmetric -s -j
```
