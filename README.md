# Scoop Bucket

[![Excavator](https://github.com/0b1000/bucket/actions/workflows/excavator.yml/badge.svg)](https://github.com/0b1000/bucket/actions/workflows/excavator.yml)

Personal use bucket for [Scoop](https://scoop.sh), the Windows command-line installer.

## Usage

Just add this bucket to your scoop.

```powershell
# Add bucket
scoop bucket add https://github.com/0b1000/bucket 0b1000
# Install package in this bucket
scoop install 0b1000/<package>
```

## Tips

If you encounter a **hash verification** error, it may be because the provider modified the release file without changing the version number. Please **skip verification** to install or update it and wait for the next time CI to be automatically updated.

```powershell
# For install package
scoop install <package> -s
# For update package
scoop update <package> -s
```
