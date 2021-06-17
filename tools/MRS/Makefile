# Convenience make file with targets for complex tasks
# - Some targets update the working directory so be careful
# - These targets contain the commands that were in the .gitlab-ci.yml for various repositories. Putting them here allows us to develop and debug them at the command line. 

# Path to this and other make files
MPTH ?= $(dir $(firstword $(MAKEFILE_LIST)))
# Path to configuration working directory (MOM6-examples)
CONFIGS ?= .

all:

# MOM6-examples gitlab pipeline (to be run from top of MOM6-examples)
# - Assumes a recursive clone of MOM6-examples is complete
# - Steps implemented in target:
#   - clone source code components for coupled models
#   - build executables (only dynamic symmetric)
#   - create manifest
#   - run standard tests
#   - check status of doc files
m6e-pipeline:
	test -d src/LM3 || make -f tools/MRS/Makefile.clone clone_gfdl -s
	time make -f tools/MRS/Makefile.build repro_gnu -s -j
	bash tools/MRS//generate_manifest.sh . tools/MRS/excluded-expts.txt > manifest.mk
	(echo '#!/bin/tcsh';echo 'make -f tools/MRS/Makefile.run gnu_all MEMORY=dynamic_symmetric -s -j') > job.sh
	sbatch --clusters=c3,c4 --nodes=25 --time=0:08:00 --account=gfdl_o --qos=debug --job-name=mom6_examples_tests --output=log.$(CI_PIPELINE_ID) --wait job.sh
	git status
	git diff --exit-code

# Gaea-stats-MOM6-examples gitlab pipelines (to be run from above MOM6-examples)
# Note: multiple targets for each job.
# These commands are run before each job stage
# - recursively clone submodules
# - clone other components
# - link to data sets
stats-pipeline-pre-script:
	echo Cache directory env var = $(CACHE_DIR)
	cd MOM6-examples && make -f tools/MRS/Makefile.clone clone_gfdl -s
	cd MOM6-examples && make -f tools/MRS/Makefile.clone .datasets -s
# Build gnu repro executables
# - compile dynamic executables
# - compile static executables
# - cache executables in a tarfile
stats-pipeline-build-repro-gnu: $(CACHE_DIR)/build-gnu-repro-$(CI_PIPELINE_ID).tgz
$(CACHE_DIR)/build-gnu-repro-$(CI_PIPELINE_ID).tgz:
	cd MOM6-examples && time make -f tools/MRS/Makefile.build repro_gnu -s -j
	cd MOM6-examples && time make -f tools/MRS/Makefile.build static_gnu -s -j
	time tar zvcf $@ `find MOM6-examples/build/gnu/repro -name MOM6`
stats-pipeline-build-repro-intel: $(CACHE_DIR)/build-intel-repro-$(CI_PIPELINE_ID).tgz
$(CACHE_DIR)/build-intel-repro-$(CI_PIPELINE_ID).tgz:
	cd MOM6-examples && time make -f tools/MRS/Makefile.build repro_intel -s -j
	time tar zvcf $@ `find MOM6-examples/build/intel/repro -name MOM6`
stats-pipeline-build-repro-pgi: $(CACHE_DIR)/build-pgi-repro-$(CI_PIPELINE_ID).tgz
$(CACHE_DIR)/build-pgi-repro-$(CI_PIPELINE_ID).tgz:
	cd MOM6-examples && time make -f tools/MRS/Makefile.build repro_pgi -s -j
	time tar zvcf $@ `find MOM6-examples/build/pgi/repro -name MOM6`
stats-pipeline-build-debug-gnu: $(CACHE_DIR)/build-gnu-debug-$(CI_PIPELINE_ID).tgz
$(CACHE_DIR)/build-gnu-debug-$(CI_PIPELINE_ID).tgz:
	cd MOM6-examples && time make -f tools/MRS/Makefile.build debug_gnu -s -j
	time tar zvcf $@ `find MOM6-examples/build/gnu/debug -name MOM6`
# Run model
# - Fetch executables from cache
# - Create the manifest
# - Use Makefile.test to create tar files of the ocean.stats "in place" within MOM6-examples
# - Check the last was written
# - Cache all the tar files
stats-pipeline-run: $(CACHE_DIR)/results-$(CI_PIPELINE_ID).tgz
$(CACHE_DIR)/results-$(CI_PIPELINE_ID).tgz: $(CACHE_DIR)/build-gnu-repro-$(CI_PIPELINE_ID).tgz $(CACHE_DIR)/build-intel-repro-$(CI_PIPELINE_ID).tgz $(CACHE_DIR)/build-pgi-repro-$(CI_PIPELINE_ID).tgz $(CACHE_DIR)/build-gnu-debug-$(CI_PIPELINE_ID).tgz
	time tar zxf $(CACHE_DIR)/build-gnu-repro-$(CI_PIPELINE_ID).tgz
	time tar zxf $(CACHE_DIR)/build-intel-repro-$(CI_PIPELINE_ID).tgz
	time tar zxf $(CACHE_DIR)/build-pgi-repro-$(CI_PIPELINE_ID).tgz
	time tar zxf $(CACHE_DIR)/build-gnu-debug-$(CI_PIPELINE_ID).tgz
	cd MOM6-examples && bash tools/MRS//generate_manifest.sh . tools/MRS/excluded-expts.txt > manifest.mk
	(echo '#!/bin/tcsh';echo 'cd MOM6-examples && make -f tools/MRS/Makefile.tests all') > job.sh
	sbatch --clusters=c3,c4 --nodes=30 --time=0:42:00 --account=gfdl_o --qos=debug --job-name=mom6_full_regression --output=log.$(CI_PIPELINE_ID) --wait job.sh
	cat log.$(CI_PIPELINE_ID)
	test -f MOM6-examples/results.ignore/restart_stats_gnu.tar.gz
	cd MOM6-examples/results.ignore && time tar zvcf $(CACHE_DIR)/results-$(CI_PIPELINE_ID).tgz *.tar.gz
# Tests
# - unpack cached ocean.stats files
stats-pipeline-test-gnu-nonsym: | MOM6-examples/results.ignore
	cd MOM6-examples/results.ignore && time tar vzxf $(CACHE_DIR)/results-$(CI_PIPELINE_ID).tgz
	cd MOM6-examples && make -f tools/MRS/Makefile.tests gnu_non_symmetric
stats-pipeline-test-intel-nonsym: | MOM6-examples/results.ignore
	cd MOM6-examples/results.ignore && time tar vzxf $(CACHE_DIR)/results-$(CI_PIPELINE_ID).tgz
	cd MOM6-examples && make -f tools/MRS/Makefile.tests intel_non_symmetric
stats-pipeline-test-pgi-nonsym: | MOM6-examples/results.ignore
	cd MOM6-examples/results.ignore && time tar vzxf $(CACHE_DIR)/results-$(CI_PIPELINE_ID).tgz
	cd MOM6-examples && make -f tools/MRS/Makefile.tests pgi_non_symmetric
stats-pipeline-test-gnu-sym: | MOM6-examples/results.ignore
	cd MOM6-examples/results.ignore && time tar vzxf $(CACHE_DIR)/results-$(CI_PIPELINE_ID).tgz
	cd MOM6-examples && make -f tools/MRS/Makefile.tests gnu_symmetric
stats-pipeline-test-intel-sym: | MOM6-examples/results.ignore
	cd MOM6-examples/results.ignore && time tar vzxf $(CACHE_DIR)/results-$(CI_PIPELINE_ID).tgz
	cd MOM6-examples && make -f tools/MRS/Makefile.tests intel_symmetric
stats-pipeline-test-pgi-sym: | MOM6-examples/results.ignore
	cd MOM6-examples/results.ignore && time tar vzxf $(CACHE_DIR)/results-$(CI_PIPELINE_ID).tgz
	cd MOM6-examples && make -f tools/MRS/Makefile.tests pgi_symmetric
stats-pipeline-test-gnu-layout: | MOM6-examples/results.ignore
	cd MOM6-examples/results.ignore && time tar vzxf $(CACHE_DIR)/results-$(CI_PIPELINE_ID).tgz
	cd MOM6-examples && make -f tools/MRS/Makefile.tests gnu_layout
stats-pipeline-test-intel-layout: | MOM6-examples/results.ignore
	cd MOM6-examples/results.ignore && time tar vzxf $(CACHE_DIR)/results-$(CI_PIPELINE_ID).tgz
	cd MOM6-examples && make -f tools/MRS/Makefile.tests intel_layout
stats-pipeline-test-pgi-layout: | MOM6-examples/results.ignore
	cd MOM6-examples/results.ignore && time tar vzxf $(CACHE_DIR)/results-$(CI_PIPELINE_ID).tgz
	cd MOM6-examples && make -f tools/MRS/Makefile.tests pgi_layout
stats-pipeline-test-gnu-static: | MOM6-examples/results.ignore
	cd MOM6-examples/results.ignore && time tar vzxf $(CACHE_DIR)/results-$(CI_PIPELINE_ID).tgz
	cd MOM6-examples && make -f tools/MRS/Makefile.tests gnu_static
stats-pipeline-test-gnu-restart: | MOM6-examples/results.ignore
	cd MOM6-examples/results.ignore && time tar vzxf $(CACHE_DIR)/results-$(CI_PIPELINE_ID).tgz
	cd MOM6-examples && make -f tools/MRS/Makefile.tests gnu_check_restarts
MOM6-examples/results.ignore:
	mkdir -p $@


# `make build` to build all executables
build: repro_gnu repro_intel repro_pgi
repro_gnu repro_intel repro_pgi debug_gnu debug_intel debug_pgi:
	$(MAKE) -f $(MPTH)/Makefile.build $@

# Manifest of experiments to run. This target scans the layout log files for PE sets, so assumes all are present and correct (clean checkout).
MANIFEST ?= $(MPTH)manifest.mk
manifest: $(MANIFEST)
$(MANIFEST): $(MPTH)excluded-expts.txt
	bash $(MPTH)/generate_manifest.sh $(CONFIGS) $< > $@

# `make run_sym`
# `make run_nonsym`
MEMORY ?= dynamic_symmetric
run_sym: MEMORY=dynamic_symmetric
run_nonsym: MEMORY=dynamic_nonsymmetric
run run_sym run_nonsym: gnu_all intel_all pgi_all
gnu_all intel_all pgi_all:
	$(MAKE) -f $(MPTH)/Makefile.run MEMORY=$(MEMORY) $@
