# sparsefile
Python package for creating and managing sparse files

This is a pure-Python package that uses ctypes to call OS-native libraries to create sparse files and to remove physical storage assigned to those files. It was inspired by the [fallocate PyPi module](https://github.com/trbs/fallocate), but requires no compilation. I would be willing to extend this to work on OS X but personally lack the means to test the code.

## Usage
`open_sparse(file, mode='r', buffering=- 1, encoding=None, errors=None, newline=None, closefd=True, opener=None)`

This function wraps the [built-in open() function](https://docs.python.org/3/library/functions.html#open). An additional method, `hole()` is attached to the object returned by open. Refer to the Python documentation for open for explanations on the arguments, returned object, and potential exceptions.

As a requirement for use, the opened file must be writable, as verified by the writeable() method of the file object. If not, this function raises a RuntimeError and the file is closed.

`file.hole(start, length)`

This method attempts to create a "hole" in the file by removing the associated storage used by a file starting at byte `start` for a length of `length` bytes. Future reads of that portion of the file will return zeroes for all bytes in the deallocated range. Future writes to that range will reallocate file storage for written data (including all zeroes). 

If successful at deallocating the storage at that range, returns True. If unsuccessful, it throws an `OSError` exception containing the appropriate error code generated by the operating system.

## Implementation Details
### Windows
Windows supports sparse files on certain versions of Windows and on specific filesystems. See [FSCTL_SET_ZERO_DATA IOCTL](https://learn.microsoft.com/en-us/windows/win32/api/winioctl/ni-winioctl-fsctl_set_zero_data) for more details.

`open_sparse()` Notes:

Windows requires a file to be flagged as a sparse file in order to be able to make a file sparse. This means that the file must already exist in order for this function to work. This package attempts to set the file as sparse at the time of opening. If unable, an OSError is thown.

### Linux
Broadly speaking, a file in Linux only needs to be on a supported filesystem in order to become sparse. The [Debian Man Pages site](https://manpages.debian.org/bookworm/manpages-dev/fallocate.2.en.html) has a current list as of this writing.