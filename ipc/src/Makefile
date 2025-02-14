#
#  Copyright (C) 2010 Texas Instruments Incorporated - http://www.ti.com/
# 
#  This application is free software; you can redistribute it and/or
#  modify it under the terms of the GNU Lesser General Public License
#  version 2.1, as published by the Free Software Foundation.
# 
#  This library is distributed in the hope that it will be useful,
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
#  Makefile for ipc library

-include $(PCD_ROOT)/.config

CC := $(patsubst "%",%,$(CONFIG_PCD_CROSS_COMPILER_PREFIX))gcc
AR := $(patsubst "%",%,$(CONFIG_PCD_CROSS_COMPILER_PREFIX))ar
CFLAGS += -MMD -Wall -fPIC -g
LDFLAGS += -shared

# includes
CFLAGS += -I$(PCD_ROOT)/ipc/include -I$(PCD_ROOT)/pcd/include

obj-y := $(patsubst %.c,%.o,$(shell ls *.c 2> /dev/null))
ifdef CONFIG_PCD_USE_STATIC_LIBS
TARGET = libipc.a
else
TARGET = libipc.so
endif

all: $(TARGET) install

$(TARGET): $(obj-y)
	@echo "  LINK	 	$@"
ifdef CONFIG_PCD_USE_STATIC_LIBS
	$(AR) rcs $@ $(obj-y)
else
	$(CC) $(CFLAGS) $(obj-y) -o $@ -Wl,-soname,$@ $(LDFLAGS) -Wl,-Map,$(@:.so=.map)
endif
	install -m644 -p $(PCD_ROOT)/ipc/include/*.h $(PCD_ROOT)/include

install: $(TARGET) install_internal
ifneq ($(CONFIG_PCD_INSTALL_DIR_PREFIX),"")
	mkdir -p $(CONFIG_PCD_INSTALL_DIR_PREFIX)/lib
	install -m644 $(TARGET) $(CONFIG_PCD_INSTALL_DIR_PREFIX)/lib
endif

install_internal: $(TARGET)
	mkdir -p $(PCD_BIN)/target/lib
	install -m644 $(TARGET) $(PCD_BIN)/target/lib

clean:
	rm -f $(TARGET) $(TARGET:.so=.map) $(obj-y) $(obj-y:.o=.d)
	rm -f $(PCD_BIN)/target/lib/$(TARGET)
ifneq ($(CONFIG_PCD_INSTALL_DIR_PREFIX),"")
	rm -f $(CONFIG_PCD_INSTALL_DIR_PREFIX)/lib/$(TARGET)
endif

%.o: %.c
	@echo "  CC [C] 	$@"
	$(CC) $(CFLAGS) -c $(CURDIR)/$< -o $@

-include $(obj-y:.o=.d)

