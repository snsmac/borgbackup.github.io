Borg 1.2
========

`« back to all releases <.>`_

:Links: `Documentation <https://borgbackup.readthedocs.io/en/1.2-maint/>`_ · `Installation <https://borgbackup.readthedocs.io/en/1.2-maint/installation.html>`_ · `Downloads <https://github.com/borgbackup/borg/releases/latest>`_
:Date: 2022-02-22

This marks the first stable release in the Borg 1.2 series. Many improvements and new features
were incorporated into Borg 1.2 since the 1.1 release of Borg in early 2017. Even more changes
were made "under the hood": cleanups and refactorings.

Many people contributed code to this release — see the AUTHORS file in the git repo and the
git history for details.

Special thanks also go to everyone and every organization donating funds to support development
and maintenance!

Since Borg 1.2 is now stable, it will primarily receive fixes and minor additions,
but not potentially problematic code changes. Principal development continues in master branch.

Changelog summary
-----------------

This is only a summary of the changes between 1.1 and 1.2.
Check the `full changelog <https://borgbackup.readthedocs.io/en/1.2-maint/changes.html>`_
to see all changes.

Major new features in the 1.2 release series are:

- create: internally use file descriptors rather than file names (as much as
  possible) to avoid race conditions on active file systems
- create: externalize file discovery via --paths-from-stdin and --paths-from-command
- create: --content-from-command: create archive using stdout of given command
- create --compression: new 'obfuscate' pseudo compressor obfuscates compressed
  chunk size in repository
- create --chunker-params: new, very fast 'fixed' block size chunker
- create: improved sparse file support
- compact: separate "borg compact" needs to be used to free repository space.
- repository: better speed and less stuff moving around
- check --max-duration: incremental, time-limited repo check (crc32 check only)
- mount: support new/maintained pyfuse3 as an alternative to old llfuse lib
- import-tar: added the complement of export-tar
- minimal native Windows support, see windows readme (WIP)

Other changes:

- create/recreate: showing the current file name **before** starting to back it
  up makes life easier, especially for very big files which take a while...
- create: first ctrl-c (SIGINT) triggers checkpoint and aborts
- create --remote-buffer: use an upload buffer for remote repos
- prune: show which rule was applied to keep archive (kind of self-explaining)
- check: much faster when recovering data from corrupted segment files
- new BORG_WORKAROUNDS mechanism
- works with latest Python, msgpack, PyInstaller, etc.
- major setup code refactoring (esp. library handling), needs pypi "pkgconfig"
- other major internal refactorings / cleanups
- internal AEAD-style crypto api
- internal msgpack-wrapper to avoid current and future compatibility issues
- improved C code portability / basic MSC compatibility
- improved documentation (a lot of this was also backported to 1.1.x though)

Compatibility notes for upgrading from Borg 1.1 to Borg 1.2:

- **No explicit "borg upgrade" is required.**
- dropped support / testing for older Pythons, minimum requirement is 3.8.
  In case your OS does not provide Python >= 3.8, consider using our binary,
  which does not need an external Python interpreter. Or continue using
  borg 1.1.x, which is still supported.
- freeing repository space only happens when "borg compact" is invoked.
- mount: the default for --numeric-ids is False now (same as borg extract)
- borg create --noatime is deprecated. Not storing atime is the default behaviour
  now (use --atime if you want to store the atime).
- list: corrected mix-up of "isomtime" and "mtime" formats.
  Previously, "isomtime" was the default but produced a verbose human format,
  while "mtime" produced a ISO-8601-like format.
  The behaviours have been swapped (so "mtime" is human, "isomtime" is ISO-like),
  and the default is now "mtime".
  "isomtime" is now a real ISO-8601 format ("T" between date and time, not a space).
- create/recreate --list: file status for all files used to get announced *AFTER*
  the file (with borg < 1.2). Now, file status is announced *BEFORE* the file
  contents are processed. If the file status changes later (e.g. due to an error
  or a content change), the updated/final file status will be printed again.
- removed deprecated-since-long stuff (deprecated since):

  - command "borg change-passphrase" (2017-02), use "borg key ..."
  - option "--keep-tag-files" (2017-01), use "--keep-exclude-tags"
  - option "--list-format" (2017-10), use "--format"
  - option "--ignore-inode" (2017-09), use "--files-cache" w/o "inode"
  - option "--no-files-cache" (2017-09), use "--files-cache=disabled"
- removed BORG_HOSTNAME_IS_UNIQUE env var.
  to use borg you must implement one of these 2 scenarios:

  - 1) the combination of FQDN and result of uuid.getnode() must be unique
       and stable (this should be the case for almost everybody, except when
       having duplicate FQDN *and* MAC address or all-zero MAC address)
  - 2) if you are aware that 1) is not the case for you, you must set
       BORG_HOST_ID env var to something unique.
- exit with 128 + signal number, #5161.
  if you have scripts expecting rc == 2 for a signal exit, you need to update
  them to check for >= 128.
