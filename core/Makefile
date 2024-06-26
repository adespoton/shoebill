
CC = clang
CFLAGS = -O3 -ggdb -flto -Wno-deprecated-declarations
# CFLAGS = -O0 -ggdb -Wno-deprecated-declarations


DEPS = mc68851.h shoebill.h Makefile macro.pl
NEED_DECODER = cpu dis
NEED_PREPROCESSING = adb mc68851 mem via floppy core_api fpu
NEED_NOTHING = VMNet atrap_tab coff exception macii_symbols redblack scsi video filesystem alloc_pool toby_frame_buffer sound ethernet SoftFloat/softfloat

# Object files that can be compiled directly from the source
OBJ_NEED_NOTHING = $(patsubst %,$(TEMP)/%.o,$(NEED_NOTHING))

# Object files than need preprocessing with macro.pl
OBJ_NEED_PREPROCESSING = $(patsubst %,$(TEMP)/%.o,$(NEED_PREPROCESSING))

# Object files that depend on the instruction decoder
OBJ_NEED_DECODER = $(patsubst %,$(TEMP)/%.o,$(NEED_DECODER))

# Files that NEED_DECODER also NEED_PREPROCESSING
POST_PREPROCESSING = $(patsubst %,$(TEMP)/%.post.c,$(NEED_PREPROCESSING)) $(patsubst %,$(TEMP)/%.post.c,$(NEED_DECODER))



# All the object files compiled for x86_64
OBJ_x86_64 = $(OBJ_NEED_NOTHING) $(OBJ_NEED_PREPROCESSING) $(OBJ_NEED_DECODER)

# The object files compiled for arm64 (the same as x86_64 files, but with .arm64 appended)
OBJ_arm64 = $(patsubst %,%.arm64,$(OBJ_x86_64))


MACRO = perl macro.pl

TEMP = ../intermediates


all: $(TEMP)/libshoebill_core.a 

$(TEMP)/libshoebill_core.a: $(TEMP) $(DEPS) $(OBJ_x86_64)
	libtool -static -v -o $(TEMP)/libshoebill_core.a.x86_64 $(OBJ_x86_64)
	libtool -static -v -o $(TEMP)/libshoebill_core.a.arm64 $(OBJ_arm64)
	lipo -create -output $(TEMP)/libshoebill_core.a $(TEMP)/libshoebill_core.a.x86_64 $(TEMP)/libshoebill_core.a.arm64


# Split object files into arm64/x86_64 versions, since it seems that libtool is unable to
# link a static universal library for -O4 object files.
# x86_64 object files have the form "intermediates/<file_name>.o
# arm64 object files have the form "intermediates/<file_name>.o.arm64

# Build object files
$(OBJ_NEED_NOTHING): $(TEMP)/%.o: %.c $(DEPS)
	$(CC) -c -arch x86_64 $(CFLAGS) $< -o $@
	$(CC) -c -arch arm64 $(CFLAGS) $< -o $@.arm64

$(OBJ_NEED_PREPROCESSING): $(TEMP)/%.o: $(TEMP)/%.post.c $(DEPS)
	$(CC) -c -arch x86_64 $(CFLAGS) $< -o $@
	$(CC) -c -arch arm64 $(CFLAGS) $< -o $@.arm64

$(OBJ_NEED_DECODER): $(TEMP)/%.o: $(TEMP)/%.post.c $(DEPS) $(TEMP)/dis_decoder_guts.c $(TEMP)/inst_decoder_guts.c
	$(CC) -c -arch x86_64 $(CFLAGS) $< -o $@
	$(CC) -c -arch arm64 $(CFLAGS) $< -o $@.arm64

# Preprocess C files 
$(POST_PREPROCESSING): $(TEMP)/%.post.c: %.c $(DEPS)
	$(MACRO) $< $@

# Generate instruction decoders
$(TEMP)/inst_decoder_guts.c: $(TEMP)/decoder_gen $(DEPS)
	$(TEMP)/decoder_gen inst $(TEMP)/
$(TEMP)/dis_decoder_guts.c: $(TEMP)/decoder_gen $(DEPS)
	$(TEMP)/decoder_gen dis $(TEMP)/

# Compile the decoder generator
$(TEMP)/decoder_gen: decoder_gen.c $(DEPS)
	$(CC) decoder_gen.c -o $(TEMP)/decoder_gen


$(TEMP):
	mkdir -p $(TEMP)
	mkdir -p $(TEMP)/SoftFloat

clean:
	rm -rf $(TEMP)

