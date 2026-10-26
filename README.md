# gaster

checkm8 exploit tool (fork of 0x7ff/gaster): puts a device in pwned DFU
mode and can decrypt files with the device's GID0 AES key. The target
device must already be in DFU mode (Apple VID 0x5AC, DFU PID 0x1227);
the CPID and its offsets are detected automatically from the DFU serial
string.

## Build

```sh
make macos    # macOS host (IOKit + CoreFoundation, via xcrun)
make libusb   # Linux host (needs libusb-1.0 + OpenSSL)
make ios      # iOS host (armv7 + arm64)
make payload  # regenerate the prebuilt payload_*.bin files (needs binutils gobjcopy)
make clean    # remove the binary, generated payload headers, and the ios headers dir
```

The `payload_*.bin` files are prebuilt and committed, so a normal build
does not need `make payload`.

`ent.plist` holds entitlements for the iOS binary
(`platform-application`, `com.apple.private.security.no-container`);
it is used when signing the iOS build, not by the Makefile.

## Usage

```sh
./gaster reset                    # reset DFU state
./gaster pwn                      # put the device in pwned DFU mode
./gaster decrypt src dst          # decrypt a file using the GID0 AES key
./gaster decrypt_kbag <96-hex-kbag>  # decrypt a KBAG using the GID0 AES key
```

Environment variables (milliseconds; defaults shown by the tool on start):

```sh
USB_TIMEOUT=5 USB_ABORT_TIMEOUT_MIN=0 ./gaster pwn
```

## Notes

- If `make ios` fails on a current Xcode, check the `-arch` and
  `-mios-version-min` flags in the Makefile first: the ios target still
  lists armv7 and iOS 9.0, which newer toolchains may no longer accept.
  The same applies to `-mmacosx-version-min=10.9` in the macos target.
- The exploit stages (RESET, SETUP, SPRAY, PATCH) run automatically
  once a DFU device is found; if a stage fails, the tool resets the USB
  handle and retries from RESET.
