# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# this converts rotate instructions from "ro[lr] <reg>" -> "ro[lr] <reg>, 1" for yasm compatibility
PERL_FIXUP_ROTATE=perl -i -pe 's/(ro[rl]\s+\w{2,3})$$/\1, 1/'

C2GOASM=c2goasm
CC=clang-19
C_FLAGS=-target x86_64-unknown-none -masm=intel -mno-red-zone -mstackrealign -mllvm -inline-threshold=1000 \
				-fno-asynchronous-unwind-tables -fno-exceptions -fno-rtti -O3 -fno-builtin -ffast-math -fno-jump-tables -I_lib
ASM_FLAGS_AVX2=-mavx2 -mfma
ASM_FLAGS_SSE4=-msse4
ASM_FLAGS_BMI2=-mbmi2
ASM_FLAGS_POPCNT=-mpopcnt

C_FLAGS_NEON=-O3 -fvectorize -mllvm -force-vector-width=16 -fno-asynchronous-unwind-tables -mno-red-zone -mstackrealign -fno-exceptions \
	-fno-rtti -fno-builtin -ffast-math -fno-jump-tables -I_lib

GO_SOURCES  := $(shell find . -path ./_lib -prune -o -name '*.go' -not -name '*_test.go')
ALL_SOURCES := $(shell find . -path ./_lib -prune -o -name '*.go' -name '*.s' -not -name '*_test.go')

.PHONY: assembly

INTEL_SOURCES := \
	bloom_filter_block_avx2_amd64.s bloom_filter_block_sse4_amd64.s 

ARM_SOURCES := \
	bloom_filter_block_neon_arm64.s


assembly: $(INTEL_SOURCES)

_lib/bloom_filter_block_avx2_amd64.s: _lib/bloom_filter_block.c
	$(CC) -S $(C_FLAGS) $(ASM_FLAGS_AVX2) $^ -o $@ ; $(PERL_FIXUP_ROTATE) $@; perl -i -pe 's/mem(cpy|set)/clib·_mem\1(SB)/' $@

_lib/bloom_filter_block_sse4_amd64.s: _lib/bloom_filter_block.c
	$(CC) -S $(C_FLAGS) $(ASM_FLAGS_SSE4) $^ -o $@ ; $(PERL_FIXUP_ROTATE) $@; perl -i -pe 's/mem(cpy|set)/clib·_mem\1(SB)/' $@

# neon not supported by c2goasm, will have to do it manually 
#_lib/bloom_filter_block_neon.s: _lib/bloom_filter_block.c
#	$(CC) -S $(C_FLAGS_NEON) $^ -o $@ ; $(PERL_FIXUP_ROTATE) $@

bloom_filter_block_avx2_amd64.s: _lib/bloom_filter_block_avx2_amd64.s
	$(C2GOASM) -a -f $^ $@

bloom_filter_block_sse4_amd64.s: _lib/bloom_filter_block_sse4_amd64.s
	$(C2GOASM) -a -f $^ $@

clean:
	rm -f $(INTEL_SOURCES)
	rm -f $(addprefix _lib/,$(INTEL_SOURCES))