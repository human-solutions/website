+++
title="x-path"
description="An if-it-compiles-it-works solution for safe Rust paths with abs/rel and file/dir variants for API safety and cross platform support."
date = 2018-09-29T11:36:33+08:00
draft = false
[taxonomies]
tags=["test", "original_post"]
[extra]
toc=true
+++

## WIP: X-Path

X-Path is a if-it-compiles-it-works solution for safe Rust paths with abs/rel and file/dir
variants for API safety and cross platform support

# Use cases

## Config files

The paths below are valid on any platform. They will be cleaned
and have environment variables resolved at load.

```toml
dir1 = "~/mydir/${SOME_ENV}/../"
dir2 = "c:\\anotherdir\\%ANOTHER_ENV%"
```

## Clear expectations

Use one of the below to communicate what your function or API expects.

|     | Any       | Dir          | File          |
| --- | --------- | ------------ | ------------- |
| Any | [AnyPath] | [DirPath]    | [FilePath]    |
| Rel | [RelPath] | [RelDirPath] | [RelFilePath] |
| Abs | [AbsPath] | [AbsDirPath] | [AbsFilePath] |

```rust
fn mirror(file: RelFilePath, from: AbsDirPath, to: AbsDirPath) {}
```

## Testable

```rust
#[test]
fn test() -> anyhow::Result<()> {
    // imagine that the path string is read from a conf.toml file:
    let dir = AbsDirPath::new(r"~/dir1//..\dir2");

    // when using the alternative debug specifier, if the path starts
    // with current working directory or user home, then they are replaced
    // with '.' or '~' respectively. The path separator used is always '/'.
    assert_eq!(format!("{:#?}", dir), "AbsDirPath(~/dir2)");

    // standard debug output uses the internal representation of the path
    // which uses the full path with platform specific path separators.
    // linux:
    assert_eq!(format!("{:?}", dir), "AbsDirPath(/home/me/code/dir2)");
    // windows:
    assert_eq!(format!("{:?}", dir), r"AbsDirPath(c:\Users\me\code\dir2)");
}
```

## Cross-platform

Both Windows-style and Unix-style paths can be used on all platforms. They
are all resolved and converted into a unified format that is comparable.

The typical file system restrictions are enforced when read.
On Windows, the NTFS, VFAT and exFAT restrictions are applied which are
much more stringent than the Unix ones. Enable the feature `strict` if
you want the same restrictions applied when running on Unix.

## Convenient

Access the paths as `&str`, all paths implement:

- [Display](std::fmt::Display) for easy display.
- `AsRef<Path>` for interoperability with all the [std::fs] operations.
- Iterate through all the path segments as `&str`ings with `path.segments()`.
- Many convenient functions: see the doc for each path type.
