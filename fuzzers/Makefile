SELF_DIR := $(dir $(lastword $(MAKEFILE_LIST)))

# Developer options for quicker runs
# Run algorithm, but only a small number of iterations
QUICK=N
# Skip metadata steps like tile and part generation
# Ex: FUZZONLY=N: 30 min, FUZZONLY=Y: 6 min
FUZZONLY=N
# 074 fuzzer isn't changed often and is slow
# Allow building without it
BITONLY=N

all:
clean: clean_fuzzers clean_piplists clean_logs clean_locks

clean_locks:
	rm -rf /tmp/segbits_*.db.lock

clean_piplists:
	rm -rf $(XRAY_FUZZERS_DIR)/piplist/build

define fuzzer

# $(1) - Fuzzer name/directory
# $(2) - Space seperated list of dependencies for the fuzzer.

# Make the all target depend on the run.ok file for the fuzzer.
all: $(1)/run.ok

# Make the clean target run `make clean` in the fuzzer's directory.
clean_fuzzers::
	$$(MAKE) -C $(1) clean

clean_logs::
	rm -rf $(1)/logs

# Describe how to create the fuzzer's run.ok file.
# This command must start with a + to tell make to pass the jobserver
# parameters downwards.
ifeq ($(VERBOSE),Y)

# When verbose we just call make directory
$(1)/run.ok: $(addsuffix /run.ok,$(2))
	$$(MAKE) -C $(1) run

else

# When not verbose, we use the run_fuzzer wrapper which will save the results
# to log files.
$(1)/run.ok: $(addsuffix /run.ok,$(2))
	+@$(SELF_DIR)/run_fuzzer.py $(1)

endif

endef

$(eval $(call fuzzer,000-init-db,))

ifneq ($(FUZZONLY),Y)
$(eval $(call fuzzer,001-part-yaml,000-init-db))
$(eval $(call fuzzer,005-tilegrid,001-part-yaml))
else
all:: 005-tilegrid/run.ok
	touch 005-tilegrid/run.ok
endif

ifneq ($(XRAY_DATABASE),kintex7)
$(eval $(call fuzzer,007-timing,005-tilegrid))
endif
$(eval $(call fuzzer,010-clb-lutinit,005-tilegrid))
$(eval $(call fuzzer,011-clb-ffconfig,005-tilegrid))
$(eval $(call fuzzer,012-clb-n5ffmux,005-tilegrid))
$(eval $(call fuzzer,013-clb-ncy0,005-tilegrid))
$(eval $(call fuzzer,014-clb-ffsrcemux,005-tilegrid))
$(eval $(call fuzzer,015-clb-nffmux,005-tilegrid))
$(eval $(call fuzzer,016-clb-noutmux,005-tilegrid))
$(eval $(call fuzzer,017-clb-precyinit,005-tilegrid))
$(eval $(call fuzzer,018-clb-ram,005-tilegrid))
$(eval $(call fuzzer,019-clb-ndi1mux,005-tilegrid))
$(eval $(call fuzzer,025-bram-config,005-tilegrid))
$(eval $(call fuzzer,026-bram-data,005-tilegrid))
$(eval $(call fuzzer,027-bram36-config,005-tilegrid))
$(eval $(call fuzzer,028-fifo-config,005-tilegrid))
$(eval $(call fuzzer,029-bram-fifo-config,005-tilegrid))
$(eval $(call fuzzer,030-iob,005-tilegrid))
$(eval $(call fuzzer,032-cmt-pll,005-tilegrid))
$(eval $(call fuzzer,034-cmt-pll-pips,005-tilegrid 071-ppips))
$(eval $(call fuzzer,035-iob-ilogic,005-tilegrid))
$(eval $(call fuzzer,035a-iob-idelay,005-tilegrid))
$(eval $(call fuzzer,036-iob-ologic,005-tilegrid))
$(eval $(call fuzzer,037-iob-pips,005-tilegrid))
$(eval $(call fuzzer,038-cfg,005-tilegrid))
$(eval $(call fuzzer,039-hclk-config,005-tilegrid))
$(eval $(call fuzzer,040-clk-hrow-config,005-tilegrid))
$(eval $(call fuzzer,041-clk-hrow-pips,005-tilegrid))
$(eval $(call fuzzer,042-clk-bufg-config,005-tilegrid))
$(eval $(call fuzzer,043-clk-rebuf-pips,005-tilegrid))
$(eval $(call fuzzer,044-clk-bufg-pips,046-clk-bufg-muxed-pips))
$(eval $(call fuzzer,045-hclk-cmt-pips,005-tilegrid))
$(eval $(call fuzzer,046-clk-bufg-muxed-pips,005-tilegrid))
$(eval $(call fuzzer,047-hclk-ioi-pips,005-tilegrid))
$(eval $(call fuzzer,047a-hclk-idelayctrl-pips,047-hclk-ioi-pips))
$(eval $(call fuzzer,048-int-piplist,005-tilegrid))
$(eval $(call fuzzer,049-int-imux-gfan,048-int-piplist))
$(eval $(call fuzzer,050-pip-seed,048-int-piplist))
$(eval $(call fuzzer,051-pip-imuxlout-bypalts,048-int-piplist))
$(eval $(call fuzzer,052-pip-clkin,048-int-piplist))
$(eval $(call fuzzer,053-pip-ctrlin,048-int-piplist))
$(eval $(call fuzzer,054-pip-fan-alt,048-int-piplist))
$(eval $(call fuzzer,055-pip-gnd,048-int-piplist))
$(eval $(call fuzzer,056-pip-rem,049-int-imux-gfan 050-pip-seed 051-pip-imuxlout-bypalts 052-pip-clkin 053-pip-ctrlin 054-pip-fan-alt 055-pip-gnd 059-pip-byp-bounce))
$(eval $(call fuzzer,057-pip-bi,056-pip-rem))
ifneq ($(QUICK),Y)
$(eval $(call fuzzer,058-pip-hclk,005-tilegrid))
$(eval $(call fuzzer,059-pip-byp-bounce,048-int-piplist))
$(eval $(call fuzzer,060-bram-cascades,005-tilegrid))
$(eval $(call fuzzer,071-ppips,057-pip-bi 058-pip-hclk 060-bram-cascades))
ifneq ($(BITONLY),Y)
$(eval $(call fuzzer,072-ordered_wires,))
$(eval $(call fuzzer,073-get_counts,))
$(eval $(call fuzzer,074-dump_all,005-tilegrid 072-ordered_wires))
$(eval $(call fuzzer,075-pins,))
endif
endif
$(eval $(call fuzzer,100-dsp-mskpat,005-tilegrid))

quick:
	$(MAKE) QUICK=Y

# Part only targets
# -----------------

$(addprefix ${XRAY_FAMILY_DIR}/${XRAY_PART}/part.,yaml json):
	$(MAKE) -C 001-part-yaml run

${XRAY_FAMILY_DIR}/${XRAY_PART}/package_pins.csv:
	$(MAKE) -C 075-pins run

${XRAY_FAMILY_DIR}/${XRAY_PART}/tilegrid.json:
	$(MAKE) -C 005-tilegrid run

${XRAY_FAMILY_DIR}/${XRAY_PART}/tileconn.json:
	$(MAKE) -C 072-ordered_wires run
	$(MAKE) -C 073-get_counts run
	$(MAKE) -C 074-dump_all run

part_only_yaml: $(addprefix ${XRAY_FAMILY_DIR}/${XRAY_PART}/part.,yaml json)
part_only_tilegrid: ${XRAY_FAMILY_DIR}/${XRAY_PART}/tilegrid.json
part_only_tileconn: ${XRAY_FAMILY_DIR}/${XRAY_PART}/tileconn.json
part_only_pins: ${XRAY_FAMILY_DIR}/${XRAY_PART}/package_pins.csv

# part_only runs the fuzzers required to support additional parts
part_only:
	# Create PART-specific directory
	mkdir -p ${XRAY_FAMILY_DIR}/${XRAY_PART}
	# Generate YAML
	+$(MAKE) part_only_yaml
	# Generate Tile grid
	+$(MAKE) part_only_tilegrid
	# Generate Tile connections
	+$(MAKE) part_only_tileconn
	# Generate package pins
	+$(MAKE) part_only_pins

# roi_only runs the fuzzers required for supporting additional parts when building
# a roi harness.
roi_only:
	# Create PART-specific directory
	mkdir -p ${XRAY_FAMILY_DIR}/${XRAY_PART}
	# Generate YAML
	+$(MAKE) part_only_yaml
	# Generate package pins
	+$(MAKE) part_only_pins
	# Copy tilegrid and tileconn
	cp ${XRAY_FAMILY_DIR}/${XRAY_EQUIV_PART}/tilegrid.json ${XRAY_FAMILY_DIR}/${XRAY_PART}/tilegrid.json
	cp ${XRAY_FAMILY_DIR}/${XRAY_EQUIV_PART}/tileconn.json ${XRAY_FAMILY_DIR}/${XRAY_PART}/tileconn.json

.PHONY: all clean clean_fuzzers clean_logs quick part_only roi_only
