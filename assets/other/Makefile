# To use the right shell
SHELL = /bin/bash

## GCC dir is added to the path
GCCDIR = 
# name to all compilator tools 
CC = $(GCCDIR)arm-none-eabi-gcc
LD = $(GCCDIR)arm-none-eabi-gcc
AR = $(GCCDIR)arm-none-eabi-ar
OBJCOPY = $(GCCDIR)arm-none-eabi-objcopy
OBJDUMP = $(GCCDIR)arm-none-eabi-objdump

# Path to the linker script for the linkage
LINKER = linker/linker_script.ld

# To include with #include "file111.h" instead of #include "subfolder1/file111.h"
INCLUDE_DIR = -Iheaders -Isubfolder1\
			  # Different kind of option to give to gcc, adate to your case
DEBUG_OPTS = -g3 -gdwarf-2 -gstrict-dwarf
OPTS = 
WARN = -Wno-unused-but-set-variable -Wno-unused-variable 
TARGET = -mcpu=cortex-m0plus

# Gcc flags from options and others
CFLAGS = -O0 -ffunction-sections -fdata-sections \
		 -fmessage-length=0 $(TARGET) -mthumb -mfloat-abi=soft \
		 $(DEBUG_OPTS) $(OPTS) $(WARN) $(INCLUDE_DIR)\

# Option to give to linker
LDFLAGS += -T$(LINKER) -mthumb $(TARGET)

All output type file
SREC = main.srec
DUMP = main.dump
ELF = main.elf
BIN = main.bin
HEX = main.hex

# Principal rule for the makefile
ALL = $(SREC) $(DUMP) $(ELF) $(BIN) $(HEX)

# Project structure path to find all .c and .h to generate the .o at the right place
BUILD_DIR = build-obj/
SRC_DIR = sources/
SRC = $(shell find . -name '*.c') 
# Variable contenant le nombre de sources à compiler
nbsources = $(shell find . -name '*.c' | wc -l)
HEADERS = $(shell find . -name '*.h') 
SRCNAME = $(shell find . -name '*.c' -type f -exec basename {} \;)
OBJS = $(SRC:%.c=$(BUILD_DIR)%.o)

# Differents color to use be used like : echo -e ${rouge}Bonjour${fin}
noir='\033[0;30m'
rouge='\033[0;31m'
vert='\033[0;32m'
orange='\033[0;33m'
jaune='\033[1;33m'
bleu='\033[0;34m'
violet='\033[0;35m'
blanc='\033[1;37m'
fin='\033[0m'

# Funny counter to count compiled file
i=0

# To don't have trouble wit dependancies
.PHONY: clean deploy
# -----------------------------------------------------------------------------

# Main rule
all: $(ALL)

flash: all deploy

libnxp.a: $(OBJS)
	@echo -e "${jaune} '|        Creation of the static library     |'  ${fin}"
	@($(AR) -r libnxp.a $(OBJS))
	@echo -e "${jaune} '|                     OK                    |'  ${fin}"

clean:
	@echo -e "${blanc} '|    delete of *.o *.out *.a *.srec *.dump  |'  ${fin}"
	@rm -f $(shell find . -name '*.o')
	@rm -f *.out libnxp.a *.srec *.dump
	@echo -e "${blanc} '|                It is clean ;)             |'  ${fin}"

$(BUILD_DIR)%.o: %.c $(HEADERS)
	@if [ $i -eq 0 ]; then \
			@echo -e "${orange} '|             Compilation des .c to .o     |'  ${fin}"; \
		else\
			@echo -ne  "${orange} |             $(i)      /$(nbsources)            |  ${fin} \\r"; \
		fi
	@$(CC) -o $@ -c $< $(CFLAGS) 
	@$(eval i=$(shell echo -e $$(($(i)+1))))
	
	@if [ $i -eq $(nbsources) ]; then \
		@echo -e  "${orange} '|        $(nbsources)      /$(nbsources)            |' ${fin}"; \
		@echo -e "${orange} '|                     OK                    |'  ${fin}"; \
	fi

$(BUILD_DIR)main.o: $(SRC_DIR)main.c $(HEADERS)
	@echo -e "${rouge} '|       Compilation of main.c to main.o     |'  ${fin}"
	@$(CC) -o $@ -c $< $(CFLAGS)
	@echo -e "${rouge} '|                     OK                    |'  ${fin}"

%.srec: %.out
	@echo -e "${bleu} '|          Generation of main.srec          |'  ${fin}"
	@$(OBJCOPY) -O srec $< $@
	@echo -e "${bleu} '|                     OK                    |'  ${fin}"

%.dump: %.out
	@echo -e "${vert} '|        Generation of asm main.dump        |'  ${fin}"
	@$(OBJDUMP) --disassemble -S $< >$@
	@echo -e "${vert} '|                     OK                    |'  ${fin}"

# Generation of the .out
%.out: $(BUILD_DIR)%.o $(LINKER) libnxp.a
	@echo -e "${violet} '|           Generation of main.out          |'  ${fin}"
	@$(CC) $(CFLAGS) -T $(LINKER) -o $@ $< libnxp.a
	@echo -e "${violet} '|                     OK                    |'  ${fin}"

# Generation of the .elf
$(ELF): $(OBJS)
	@echo -e "${vert} '|           Generation of main.elf          |'  ${fin}"
	@$(LD) $(LDFLAGS) -o $@ $(OBJS)
	@echo -e "${vert} '|                     OK                    |'  ${fin}"

# Generation of the .bin
%.bin: %.elf
	@echo -e "${vert} '|           Generation of main.bin          |'  ${fin}"
	@$(OBJCOPY) -O binary $< $@
	@echo -e "${vert} '|                     OK                    |'  ${fin}"

# Generation of the .hex
%.hex: %.elf
	@echo -e "${vert} '|           Generation of main.hex          |'  ${fin}"
	@$(OBJCOPY) -O ihex $< $@
	@echo -e "${vert} '|                     OK                    |'  ${fin}"


# -----------------------------------------------------------------------------
# This rule will send the .bin into the board
MBED_VOLUME=$(shell df -h 2>/dev/null | fgrep "MBED" | awk '{print $9}')
deploy: $(BIN)
	@echo -e "${vert} '| Flashing the board with your amazing code |'  ${fin}"
	@cp $< MBED_VOLUME
	@sync
	@echo -e "${vert} '|                     OK                    |'  ${fin}"

