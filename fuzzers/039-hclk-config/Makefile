N ?= 50

include ../fuzzer.mk

database: build/segbits_hclk_ioi3.db

build/segbits_hclk_ioi3.rdb: $(SPECIMENS_OK)
	${XRAY_SEGMATCH} -c 5 -o build/segbits_hclk_ioi3.rdb \
		$(addsuffix /segdata_hclk_ioi3.txt,$(SPECIMENS))

build/segbits_hclk_ioi3.db: build/segbits_hclk_ioi3.rdb
	${XRAY_DBFIXUP} --db-root build --zero-db bits.dbf \
		--seg-fn-in build/segbits_hclk_ioi3.rdb \
		--seg-fn-out build/segbits_hclk_ioi3.db
	${XRAY_MASKMERGE} build/mask_hclk_ioi3.db \
		$(addsuffix /segdata_hclk_ioi3.txt,$(SPECIMENS))

pushdb: database
	${XRAY_MERGEDB} hclk_ioi3 build/segbits_hclk_ioi3.db
	${XRAY_MERGEDB} mask_hclk_ioi3 build/mask_hclk_ioi3.db

.PHONY: database pushdb
