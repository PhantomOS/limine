# Limine configuration file

## Location of the config file

Limine scans for a config file on *the boot drive*. Every partition on the boot drive
is scanned sequentially (first partition first, last partition last) for the presence
of either a `/limine.cfg`, `/boot/limine.cfg`, or a `/EFI/BOOT/limine.cfg` file, in that order.

If no config file is found in the aforementioned locations, Limine looks for the file on the fw_cfg
interface called `opt/org.limine-bootloader.config`. If that is not present and the kernel is found at
`opt/org.limine-bootloader.kernel`, Limine enters the so-called "simple mode", where the kernel is loaded from
`opt/org.limine-bootloader.kernel`, and, (if present), the background is loaded from
`opt/org.limine-bootloader.background`.

Once the file is located, Limine will use it as its config file. Other possible
candidates in subsequent partitions or directories are ignored.

It is thus imperative that the intended config file is placed in a location that will
not be shadowed by another potentially candidate config file.

## Structure of the config file

The Limine configuration file is comprised of *assignments* and *entries*.
Comments begin in '#'.

### Entries and sub-entries

*Entries* describe boot *entries* which the user can select in the *boot menu*.

An *entry* is simply a line starting with `:` followed by a newline-terminated
string.
Any *locally assignable* key that comes after it, and before another *entry*, or
the end of the file, will be tied to the *entry*.

An *entry* can be a directory, meaning it can hold sub-entries. In order for an
entry to become a directory, it needs to have a sub-entry following right after it.

A *sub-entry* is an entry with a number of `:` greater than 1 prepended to it.
Each `:` represents 1 level deeper down the tree hierarchy of directories and
entries.

Directories can be expanded (meaning they will not show up as collapsed in the
menu) by default if a `+` is put between the `:`s and the beginning of the entry's name.

### Assignments

*Assignments* are simple `KEY=VALUE` style assignments.
`VALUE` can have spaces and `=` symbols, without requiring quotations. New lines
are delimiters.

Some *assignments* are part of an entry (*local*), some other assignments are *global*.
*Global assignments* can appear anywhere in the file and are not part of an entry,
although usually one would put them at the beginning of the config.
Some *local assignments* are shared between entries using any *protocol*, while other
*local assignments* are specific to a given *protocol*.

Some keys take *URIs* as values; these are described in the next section.

*Globally assignable* keys are:
* `TIMEOUT` - Specifies the timeout in seconds before the first *entry* is automatically booted. If set to `no`, disable automatic boot. If set to `0`, boots default entry instantly (see `DEFAULT_ENTRY` key).
* `QUIET` - If set to `yes`, enable quiet mode, where all screen output except panics and important warnings is suppressed. If `TIMEOUT` is not 0, the `TIMEOUT` still occurs, and pressing any key during the timeout will reveal the menu and disable quiet mode.
* `SERIAL` - If set to `yes`, enable serial I/O for the bootloader.
* `DEFAULT_ENTRY` - 1-based entry index of the entry which will be automatically selected at startup. If unspecified, it is `1`.
* `GRAPHICS` - If set to `no`, force CGA text mode for the boot menu, else use a video mode. Ignored with Limine UEFI.
* `VERBOSE` - If set to `yes`, print additional information during boot. Defaults to not verbose.
* `RANDOMISE_MEMORY` - If set to `yes`, randomise the contents of RAM at bootup in order to find bugs related to non zeroed memory or for security reasons. This option will slow down boot time significantly. For the BIOS port of Limine, this will only randomise memory below 4GiB.
* `RANDOMIZE_MEMORY` - Alias of `RANDOMISE_MEMORY`.

Limine interface control options.

* `INTERFACE_RESOLUTION` - Specify screen resolution to be used by the Limine interface (menu, editor, console...) in the form `<width>x<height>`. This will *only* affect the Limine interface, not any booted OS. If not specified, Limine will pick a resolution automatically. If the resolution is not available, Limine will pick another one automatically. Ignored if using text mode.
* `INTERFACE_BRANDING` - A string that will be displayed on top of the Limine interface.
* `INTERFACE_BRANDING_COLOUR` - A value between 0 and 7 specifying the colour of the branding string. Default is cyan (6).
* `INTERFACE_BRANDING_COLOR` - Alias of `INTERFACE_BRANDING_COLOUR`.

Limine graphical terminal control options. They are ignored if using text mode.

* `TERM_FONT` - URI path to a font file to be used instead of the default one for the menu and terminal. The font file must be a code page 437 character set comprised of 256 consecutive glyph bitmaps. Each glyph's bitmap must be expressed left to right (1 byte per row), and top to bottom (16 bytes per whole glyph by default; see `TERM_FONT_SIZE`). See e.g. the [VGA text mode font collection](https://github.com/viler-int10h/vga-text-mode-fonts) for fonts.
* `TERM_FONT_SIZE` - The size of the font in dots, which must correspond to the font file or the display will be garbled. Note that glyphs are always one byte wide, and columns over 8 are empty. Many fonts may be used in both 8- and 9-dot wide variants. Defaults to `8x16`. Ignored if `TERM_FONT` not set or if the font fails to load.
* `TERM_FONT_SCALE` - Scaling for the font in the x and y directions. `2x2` would display the font in double size, which is useful on high-DPI displays at native resolution. `2x1` only makes the font twice as wide, similar to the VGA 40 column mode. `4x2` might be good for a narrow font on a high resolution display. Values over 8 are disallowed. Default is no scaling, i.e. `1x1`.
* `TERM_FONT_SPACING` - Horizontal spacing, in pixels, between glyphs on screen. It is equivalent to setting a font width of `<specified width>+<this value>`, except this value is preserved even in case font loading fails, and it also applies to the built-in Limine font. Defaults to 1. 0 is allowed.
* `TERM_PALETTE` - Specifies the colour palette used by the terminal (RRGGBB). It is a `;` separated array of 8 colours: black, red, green, brown, blue, magenta, cyan, and gray. Ignored if not using a graphical terminal.
* `TERM_PALETTE_BRIGHT` - Specifies the bright colour palette used by the terminal (RRGGBB). It is a `;` separated array of 8 bright colours: dark gray, bright red, bright green, yellow, bright blue, bright magenta, bright cyan, and white. Ignored if not using a graphical terminal.
* `TERM_BACKGROUND` - Terminal text background colour (TTRRGGBB). TT stands for transparency.
* `TERM_FOREGROUND` - Terminal text foreground colour (RRGGBB).
* `TERM_MARGIN` - Set the amount of margin around the terminal.
* `TERM_MARGIN_GRADIENT` - Set the thickness in pixel for the gradient around the terminal.
* `TERM_WALLPAPER` - URI where to find the .BMP file to use as wallpaper.
* `TERM_WALLPAPER_STYLE` - The style which will be used to display the wallpaper image: `tiled`, `centered`, or `stretched`. Default is `stretched`.
* `TERM_BACKDROP` - When the background style is `centered`, this specifies the colour of the backdrop for parts of the screen not covered by the background image, in RRGGBB format.

Editor control options.

* `EDITOR_ENABLED` - If set to `no`, the editor will not be accessible. Defaults to `yes`.
* `EDITOR_HIGHLIGHTING` - If set to `no`, syntax highlighting in the editor will be disabled. Defaults to `yes`.
* `EDITOR_VALIDATION` - If set to `no`, the editor will not alert you about invalid keys / syntax errors. Defaults to `yes`.

*Locally assignable (non protocol specific)* keys are:
* `COMMENT` - An optional comment string that will be displayed by the bootloader on the menu when an entry is selected.
* `PROTOCOL` - The boot protocol that will be used to boot the kernel. Valid protocols are: `linux`, `limine`, `stivale`, `stivale2`, `chainload`, `multiboot` or `multiboot1` and `multiboot2`. If the protocol is omitted, Limine will try to autodetect it, following this list of protocols, in this order: `stivale2 -> stivale1 -> multiboot2 -> multiboot1 -> limine -> linux -> failure`.
* `CMDLINE` - The command line string to be passed to the kernel. Can be omitted.
* `KERNEL_CMDLINE` - Alias of `CMDLINE`.

*Locally assignable (protocol specific)* keys are:
* Linux protocol:
  * `KERNEL_PATH` - The URI path of the kernel.
  * `MODULE_PATH` - The URI path to a module (such as initramfs).

  Note that one can define this last variable multiple times to specify multiple
  modules.
  * `RESOLUTION` - The resolution to be used. This setting takes the form of `<width>x<height>x<bpp>`. If the resolution is not available, Limine will pick another one automatically. Omitting `<bpp>` will default to 32.
  * `TEXTMODE` - If set to `yes`, prefer text mode. (BIOS only)
* Limine protocol:
  * `KERNEL_PATH` - The URI path of the kernel.
  * `MODULE_PATH` - The URI path to a module.
  * `MODULE_CMDLINE` - A command line to be passed to a module.

  **Note:** One can define these 2 last variable multiple times to specify multiple
  modules.
  The entries will be matched in order. E.g.: The 1st module path entry will be matched
  to the 1st module string entry that appear, and so on.

  * `RESOLUTION` - The resolution to be used. This setting takes the form of `<width>x<height>x<bpp>`. If the resolution is not available, Limine will pick another one automatically. Omitting `<bpp>` will default to 32.
  * `KASLR` - For relocatable kernels, if set to `no`, disable kernel address space layout randomisation. KASLR is enabled by default.
* Chainload protocol on BIOS:
  * `DRIVE` - The 1-based BIOS drive to chainload, if omitted, assume boot drive.
  * `PARTITION` - The 1-based BIOS partition to chainload, if omitted, chainload drive (MBR).
* Chainload protocol on UEFI:
  * `IMAGE_PATH` - URI of the EFI application to chainload.
  * `RESOLUTION` - The resolution to be used. This setting takes the form of `<width>x<height>x<bpp>`. If the resolution is not available, Limine will pick another one automatically. Omitting `<bpp>` will default to 32.
* stivale and stivale2 protocols:
  * `KERNEL_PATH` - The URI path of the kernel.
  * `MODULE_PATH` - The URI path to a module.
  * `MODULE_STRING` - A string to be passed to a module.

  **Note:** One can define these 2 last variable multiple times to specify multiple
  modules.
  The entries will be matched in order. E.g.: The 1st module path entry will be matched
  to the 1st module string entry that appear, and so on.

  **Note**: If `MODULE_STRING` is not specified for an entry, the `MODULE_STRING` will default to the `MODULE_PATH`.

  * `RESOLUTION` - The resolution to be used should the kernel request a graphical framebuffer. This setting takes the form of `<width>x<height>x<bpp>` and *overrides* any resolution requested by the kernel, or automatic resolution requests. If the resolution is not available, Limine will pick another one automatically. Omitting `<bpp>` will default to 32.
  * `KASLR` - For relocatable kernels, if set to `no`, disable kernel address space layout randomisation. KASLR is enabled by default.
  * `TEXTMODE` - If set to `yes`, prefer text mode if the kernel has no video mode requirements. (Only for stivale2)
* Chainload protocol on BIOS:
  * `DRIVE` - The 1-based BIOS drive to chainload, if omitted, assume boot drive.
  * `PARTITION` - The 1-based BIOS partition to chainload, if omitted, chainload drive (MBR).
* Chainload protocol on UEFI:
  * `IMAGE_PATH` - URI of the EFI application to chainload.
  * `RESOLUTION` - The resolution to be used. This setting takes the form of `<width>x<height>x<bpp>`. If the resolution is not available, Limine will pick another one automatically. Omitting `<bpp>` will default to 32.
* multiboot1 and multiboot2 protocols:
  * `KERNEL_PATH` - The URI path of the kernel.
  * `MODULE_PATH` - The URI path to a module.
  * `MODULE_STRING` - A string to be passed to a module.

  Note that one can define these 2 last variable multiple times to specify multiple
  modules.
  The entries will be matched in order. E.g.: the 1st module path entry will be matched
  to the 1st module string entry that appear, and so on.
  * `RESOLUTION` - The resolution to be used should the kernel request a graphical framebuffer. This setting takes the form of `<width>x<height>x<bpp>` and *overrides* any resolution requested by the kernel. If the resolution is not available, Limine will pick another one automatically. Omitting `<bpp>` will default to 32.

## URIs

A URI is a path that Limine uses to locate resources in the whole system. It is
comprised of a *resource*, a *root*, and a *path*. It takes the form of:
```
resource://root/path
```

The format for `root` changes depending on the resource used.

A resource can be one of the following:
* `boot` - If booted off PXE this is an alias of `tftp`. Else the `root` is the 1-based decimal value representing the partition on the boot drive (values of 5+ for MBR logical partitions). If omitted, the partition containing the configuration file on the boot drive is used. For example: `boot://2/...` will use partition 2 of the boot drive and `boot:///...` will use the partition containing the config file on the boot drive.
* `hdd` - Hard disk drives. The `root` takes the form of `drive:partition`; for example: `hdd://3:1/...` would use hard drive 3, partition 1. Partitions and drives are both 1-based (partition values of 5+ for MBR logical partitions). Omitting the partition is possible; for example: `hdd://2:/...`. Omitting the partition will access the entire volume instead of a specific partition (useful for unpartitioned media).
* `odd` - Optical disk drives (CDs/DVDs/...). The `root` takes the form of `drive:partition`; for example: `odd://3:1/...` would use optical drive 3, partition 1. Partitions and drives are both 1-based (partition values of 5+ for MBR logical partitions). Omitting the partition is possible; for example: `odd://2:/...`. Omitting the partition will access the entire volume instead of a specific partition (useful for unpartitioned media, which is often the case for optical media).
* `guid` - The `root` takes the form of a GUID/UUID, such as `guid://736b5698-5ae1-4dff-be2c-ef8f44a61c52/...`. The GUID is that of either a filesystem, when available, or a GPT partition GUID, when using GPT, in a unified namespace.
* `uuid` - Alias of `guid`.
* `tftp` - The `root` is the IP address of the tftp server to load the file from. If the root is left empty (`tftp:///...`) the file will be loaded from the server Limine booted from. This resource is only available when booting off PXE.
* `fwcfg` - The `root` must be empty. The `path` refers to a fw_cfg filename to be loaded. The canonical place to put fw_cfg files is in the `opt/fqdn.<name>` namespace.

A URI can optionally be prefixed by a `$` character to indicate that the file
pointed to be the URI is a gzip-compressed payload to be uncompressed on the
fly. E.g.: `$boot:///somemodule.gz`.

## Macros

Macros are strings that can be arbitrarily assigned to represent other strings. For example:
```
${MY_MACRO}=Some text
```

Now, whenever `${MY_MACRO}` is used in the config file (except for an assignment as above), it will
be replaced by the text `Some text`. For example:
```
CMDLINE=something before ${MY_MACRO} something after
```

Macros must always be placed inside `${...}` where `...` is the arbitrary macro name.
