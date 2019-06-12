#
#  Copyright (C) 2010 Texas Instruments Incorporated - http://www.ti.com/
# 
#  This application is free software; you can redistribute it and/or
#  modify it under the terms of the GNU Lesser General Public License
#  version 2.1, as published by the Free Software Foundation.
# 
#  This application is distributed in the hope that it will be useful,
#  but WITHOUT ANY WARRANTY; without even the implied warranty of
#  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
#  Lesser General Public License for more details.
# 
#  You should have received a copy of the GNU Lesser General Public
#  License along with this library; if not, write to the Free Software
#  Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301  USA
#
#  Copyright (C) 2010 PCD Project - http://www.rt-embedded.com/pcd
# 
#  Makefile for pcdparser application

-include $(PCD_ROOT)/.config

CONFIG_PCD_INSTALL_DIR_HOST ?= ""

HOST_CC = gcc
HOST_CFLAGS += -MMD -O2 -Wall -DPCD_HOST_BUILD -DWITHOUT_PCD_AUTOCONF
HOST_LDFLAGS =

# includes
HOST_CFLAGS += -I$(PCD_ROOT)/pcd/src/parser/include -I$(PCD_ROOT)/pcd/include

obj-y := $(patsubst %.c,%.o,$(shell ls *.c 2> /dev/null)) parser.o
TARGET = pcdparser

all: $(TARGET)

$(TARGET): $(obj-y)
	@echo "  HOST LINK	$@"
	$(HOST_CC) $(HOST_CFLAGS) $(obj-y) -o $@ $(HOST_LDFLAGS) -Wl,-Map,$@.map

install: $(TARGET)
	mkdir -p $(PCD_BIN)/host
	install -m755 $(TARGET) $(PCD_BIN)/host
ifneq ($(CONFIG_PCD_INSTALL_DIR_HOST),"")
	mkdir -p $(CONFIG_PCD_INSTALL_DIR_HOST)
	install -m755 $(TARGET) $(CONFIG_PCD_INSTALL_DIR_HOST)
endif	

clean:
	rm -f $(TARGET) $(obj-y) $(obj-y:.o=.d) $(TARGET).map
	rm -f $(PCD_ROOT)/bin/host/$(TARGET)
ifneq ($(CONFIG_PCD_INSTALL_DIR_HOST),"")
	rm -f $(CONFIG_PCD_INSTALL_DIR_HOST)/$(TARGET)
endif

%.o: %.c
	@echo "  HOST_CC [C]	$@"
	$(HOST_CC) $(HOST_CFLAGS) -c $(CURDIR)/$< -o $@

parser.o: ../../parser.c
	@echo "  HOST_CC [C]	$@"
	$(HOST_CC) $(HOST_CFLAGS) -c $(CURDIR)/$< -o $@

-include $(obj-y:.o=.d)

