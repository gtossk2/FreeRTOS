PROJECT = led_blink

EXECUTABLE = $(PROJECT).elf
BIN_IMAGE = $(PROJECT).bin
HEX_IMAGE = $(PROJECT).hex

# Toolchain configurations
CROSS_COMPILE ?= arm-none-eabi-
CC = $(CROSS_COMPILE)gcc
LD = $(CROSS_COMPILE)ld
OBJCOPY = $(CROSS_COMPILE)objcopy
OBJDUMP = $(CROSS_COMPILE)objdump
SIZE = $(CROSS_COMPILE)size

# Cortex-M4 implements the ARMv7E-M architecture
CPU = cortex-m4
CFLAGS = -mcpu=$(CPU) -march=armv7e-m -mtune=cortex-m4
CFLAGS += -mlittle-endian -mthumb
CFLAGS += -mfpu=fpv4-sp-d16 -mfloat-abi=softfp -O0

define get_library_path
    $(shell dirname $(shell $(CC) $(CFLAGS) -print-file-name=$(1)))
endef
LDFLAGS += -L $(call get_library_path,libc.a)
LDFLAGS += -L $(call get_library_path,libgcc.a)

# Basic configurations
CFLAGS += -g -std=c99
CFLAGS += -Wall

# Optimizations
CFLAGS += -g -std=c99 -O3 -ffast-math
CFLAGS += -ffunction-sections -fdata-sections
CFLAGS += -Wl,--gc-sections
CFLAGS += -fno-common
CFLAGS += --param max-inline-insns-single=1000

# specify STM32F429
CFLAGS += -DSTM32F429xx

# to run from FLASH
CFLAGS += -DVECT_TAB_FLASH
LDFLAGS += -T $(PWD)/STM32F429ZITx_FLASH.ld

# STARTUP FILE
OBJS += $(PWD)/Libraries/CMSIS/Device/ST/STM32F4xx/Source/Templates/gcc/startup_stm32f429xx.o

#My restart
OBJS += \
      $(PWD)/main.o \
      $(PWD)/system_stm32f4xx.o \

OBJS += \
      $(PWD)/../../Source/croutine.o \
      $(PWD)/../../Source/event_groups.o \
      $(PWD)/../../Source/list.o \
      $(PWD)/../../Source/queue.o \
      $(PWD)/../../Source/tasks.o \
      $(PWD)/../../Source/timers.o \
      $(PWD)/../../Source/portable/GCC/ARM_CM4F/port.o \
      $(PWD)/../../Source/portable/MemMang/heap_1.o \

OBJS += \
    $(PWD)/Libraries/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal.o \
    $(PWD)/Libraries/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_cortex.o \
    $(PWD)/Libraries/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_dma.o \
    $(PWD)/Libraries/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_flash.o \
    $(PWD)/Libraries/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_flash_ex.o \
    $(PWD)/Libraries/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_gpio.o \
    $(PWD)/Libraries/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_i2c.o \
    $(PWD)/Libraries/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_i2c_ex.o \
    $(PWD)/Libraries/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_pwr.o \
    $(PWD)/Libraries/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_pwr_ex.o \
    $(PWD)/Libraries/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_rcc.o \
    $(PWD)/Libraries/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_rcc_ex.o \
    $(PWD)/Libraries/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_spi.o \
    $(PWD)/Libraries/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_tim.o \
    $(PWD)/Libraries/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_tim_ex.o \

OBJS += \
    $(PWD)/Libraries/BSP/STM32F429I-Discovery/stm32f429i_discovery.o \

CFLAGS += -I $(PWD) \
	-I $(PWD)/Libraries/CMSIS/Device/ST/STM32F4xx/Include \
	-I $(PWD)/Libraries/CMSIS/Include \
	-I $(PWD)/Libraries/STM32F4xx_HAL_Driver/Inc \
	-I $(PWD)/Libraries/BSP/STM32F429I-Discovery \
	-I $(PWD)/../../Source/include \
	-I $(PWD)/../../Source/portable/GCC/ARM_CM4F \
	-I $(PWD)/../Common/include

all: $(BIN_IMAGE)

$(BIN_IMAGE): $(EXECUTABLE)
	$(OBJCOPY) -O binary $^ $@
	$(OBJCOPY) -O ihex $^ $(HEX_IMAGE)
	$(OBJDUMP) -h -S -D $(EXECUTABLE) > $(PROJECT).lst
	$(SIZE) $(EXECUTABLE)
	
$(EXECUTABLE): $(OBJS)
	$(LD) -o $@ $(OBJS) \
		--start-group $(LIBS) --end-group \
		$(LDFLAGS)

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

%.o: %.s
	$(CC) $(CFLAGS) -c $< -o $@

flash:
	st-flash write $(BIN_IMAGE) 0x8000000

openocd_flash:
	openocd \
	-f board/stm32f429discovery.cfg \
	-c "init" \
	-c "reset init" \
	-c "flash probe 0" \
	-c "flash info 0" \
	-c "flash write_image erase $(BIN_IMAGE)  0x08000000" \
	-c "reset run" -c shutdown

.PHONY: clean
clean:
	rm -rf $(EXECUTABLE)
	rm -rf $(BIN_IMAGE)
	rm -rf $(HEX_IMAGE)
	rm -f $(OBJS)
	rm -f $(PROJECT).lst
