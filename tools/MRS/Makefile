# Convenience Makefile with targets for complex tasks
# - Run from top of MOM6-examples

# Path to this and other makefiles
MPTH ?= $(dir $(firstword $(MAKEFILE_LIST)))

all:

# Instructions used in gitlab pipeline
m6e-pipeline:
# Recursively clone submodules
	git submodule sync --recursive && git submodule update --init --recursive
# Clone other components
	test -d src/LM3 || make -f tools/MRS/Makefile.clone clone_gfdl -s
# Build executables
	time make -f tools/MRS/Makefile.build build_gnu -s -j
# Create manifest
	time make -f tools/MRS/Makefile manifest
# Run standard tests
	(echo '#!/bin/tcsh';echo 'make -f tools/MRS/Makefile.run gnu_all MEMORY=dynamic_symmetric -s -j') > job.sh
	sbatch --clusters=c3,c4 --nodes=25 --time=0:08:00 --account=gfdl_o --qos=debug --job-name=mom6_examples_tests --output=log.$$CI_PIPELINE_ID --wait job.sh
# Check status of doc files
	git status
	git diff --exit-code


# `make build` to build all executables
build: repro_gnu repro_intel repro_pgi
repro_gnu repro_intel repro_pgi debug_gnu debug_intel debug_pgi:
	$(MAKE) -f $(MPTH)/Makefile.build $@

# Manifest of experiments to run. This target scans the layout log files for PE sets, so assumes all are present and correct (clean checkout).
MANIFEST ?= $(MPTH)manifest.mk
manifest: $(MANIFEST)
$(MANIFEST): $(MPTH)excluded-expts.txt 
	bash $(MPTH)generate_manifest.sh $< > $@

# `make run_sym`
# `make run_nonsym`
MEMORY ?= dynamic_symmetric
run_sym: MEMORY=dynamic_symmetric
run_nonsym: MEMORY=dynamic_nonsymmetric
run run_sym run_nonsym: gnu_all intel_all pgi_all
gnu_all intel_all pgi_all:
	$(MAKE) -f $(MPTH)/Makefile.run MEMORY=$(MEMORY) $@
