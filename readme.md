# polytaxis 0.0.0

## What is polytaxis?

polytaxis is a specification of a universal (non-filetype-specific) tag file header.

## Why is polytaxis?

- Many filetypes do not support metadata, and adding metadata to new filetypes requires extra developer effort and planning. A unified metadata specification should ease new development and increase the availability of file metadata.

- Disparate metadata implementations decrease compatibility and increase maintenance effort for every piece of software that interacts with metadata.

- Individual metadata formats tend to be domain specific. This impedes developing general metadata-based file management and organization solutions.

  As a concrete example, there are many software packages that will catalogue audio files, and there are many software packages that will catalogue image files, but few if not none that catalogue both.

- Individual metadata formats are arbitrarily limited: allowing only certain tags, not supporting multiple values per tag, etc.

- Individual metadata formats are overengineered: XMP stores structured, typed metadata. This makes interpreting and categorizing the data difficult without knowlege of the usage of the file.

## A brief description

polytaxis supports key/value tags as well as valueless-tags. The polytaxis header is always utf-8 encoded, and textfiles with a polytaxis header must also be utf-8 encoded.

The header has two formats - sized and unsized. The sized header begins with an explicitly specified length and can have extra allocated space so that tags can be modified without shifting the core file data. The unsized header has no unprintable characters and ends with a delimiter. The unsized header is intended to be used with purely textual data and is not recommended to be used with large files.

Polytaxis is designed to be mostly non-binary to ease implementation and debugging, and aid data recovery if something goes wrong.

## Technical specification and examples

In the following description, `\n` indicates the new-line character (`0x0A`). `\0` indicates a null character (`0x00`).

The polytaxis header is always the first data in a tagged file. The header starts with

```
polytaxis00
```

(the 'magic') which specifies that the header is present and the version of the header.

### Sized header

For sized headers, the magic is suffixed with a space and followed by a 10-digit decimal integer length, `\n`, the encoded tags, and a `\0` if the amount written to this point is less than the indicated length.

As an example:
```
polytaxis00 0000000512
author=Whole Wheat Pasta
date=1988-13-32
critical
meeting-2014
format=FormA
\0...
DOCX HEADER
```
The `...` after the first `\0` indicate the remaining unused header bytes, and are ignored.

### Unsized header

For unsized headers, the magic is suffixed with `u` then followed by `\n`, the encoded tags, and a marker to indicate the end of the tags.

The marker is always `<<<<\n`.

This looks something like this:

```
polytaxis00u
TAGS
<<<<
FILE DATA
```

With tags, the header may look like:

```
polytaxis00u
author=Rendaw
date=1988-13-32
critical
diary
format=FormA
<<<<
This is my diary.

Today I wrote another standard specification. Nobody likes my standards but I don't care. I will show them. I will keep on writing specifications until I die.
```

### Tag encoding

Tags are always in the format `part\n` or `part=part\n`, where a `part` is utf-8 text with the special sequences `=`, `\n`, and `<<<<\n` escaped by `\`.

## Libraries and software

- [polytaxis-python](https://github.com/Rendaw/polytaxis-python)

  This is a library with common methods for interacting with the polytaxis header and files containing the header.

  It also contains the utility `ptmod` for modifying the polytaxis header on files.

- [ptunwrap](https://github.com/Rendaw/ptunwrap)

  `ptunwrap` provides a compatibility layer for software that doesn't support polytaxis headers. `ptunwrapd` exposes a virtual filesystem that hides the polytaxis header on tagged files. `ptunwrap` translates real filenames into paths in the virtual filesystem.

- [ptutils](https://github.com/Rendaw/ptutils)

  `ptutils` provides the following utilities: 

  `ptmonitor` - Monitors directories and creates a database of files and metadata.

  `ptq` - Queries the `ptmonitor` database.

  `ptimport` - Translates tags from filetype-specific metadata fields to polytaxis headers.

---
&copy; Rendaw, Zarbosoft 2015

