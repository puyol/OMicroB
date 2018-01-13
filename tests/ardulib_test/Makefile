include ../../etc/Makefile.conf

ATTY     := /dev/tty.usbmodem14331
# CFLAGS   := -mmcu=atmega32u4 -DUSB_VID=0x2341 -DUSB_PID=0x8036 -DF_CPU=16000000L -DARDUINO=105 -ffunction-sections -fdata-sections -Os -w -fno-exceptions -I/usr/share/arduino/hardware/arduino/cores/arduino/avr-libc
CFLAGS   := -mmcu=atmega32u4
LFLAGS   := -Wl,-Os -Wl,--gc-sections
INCLUDES := -I./stdlib/ -I$(SRC)/byterun/

################################################################################

TARGETS := ardulib.byte ardulib.c ardulib.elf ardulib.avr ardulib.hex

ardulib: $(TARGETS)

run: ardulib.elf
	./ardulib.elf

clean:
	@rm -f *.cmi *.cmo *.cmx *.o *~ *.byte $(TARGETS)

.PHONY: ardulib run clean flash

################################################################################

ardulib.byte: avr.ml spi.ml oled.ml ardulib.ml
	CAMLLIB=$(LIB) ocamlc -custom -I $(SRC)/byterun $^ -o $@
	ocamlclean $@ -o $@

ardulib.c: ardulib.byte $(BIN)/bc2c
	$(BIN)/bc2c -local -arch 32 -heap-size 64 -stack-size 64 $< -o $@

ardulib.elf: ardulib.c
	# $(GCC) -D__PC__ -I $(OCAMLWHERE) -I$(SRC)/byterun/ -g -std=c99 -D_POSIX_C_SOURCE=199309L -Wnarrowing -DDEBUG $^ -o $@
	$(GCC) -D__PC__ -I $(OCAMLWHERE) -I$(SRC)/byterun/ -g -std=c99 -D_POSIX_C_SOURCE=199309L -Wnarrowing -DDEBUG $^ -o $@

################################################################################

ardulib.avr: $(wildcard stdlib/*.c) $(wildcard stdlib/*.cpp) $(wildcard stdlib/*/*.cpp) ardulib.c
	$(AVR_GXX) -O2 $(INCLUDES) $(CFLAGS) $(LFLAGS) -Wall  $^  -Wnarrowing -o $@

ardulib.hex: ardulib.avr
	$(AVR_OBJCOPY) -O ihex -R .eeprom $< $@

flash: ardulib.avr
	stty -f $(ATTY) 1200
	sleep 3
	$(AVRDUDE) -c avr109 -P $(ATTY) -p ATmega32U4 -b 57600 -v -D -U flash:w:$<:e

listen:
	stty -f $(ATTY) cs8 9600 ignbrk -brkint -icrnl -imaxbel -opost -onlcr -isig -icanon -iexten -echo -echoe -echok -echoctl -echoke noflsh -ixon -crtscts
	cat $(ATTY)

################################################################################