tmpfs-sync: Keep write operations on disks minimal
==================================================

Introduction
------------

The motivation of *tmpfs-sync* is to reduce the number of write
operations to your hard disk to a minimum in order to increase the
potential lifetime of SSD drives. 

*tmpfs-sync* operates on a per-directory basis. Before a directory is
accessed (e.g. during boot), the following actions are performed:

1. A copy is created on a `tmpfs`-mount (e.g. `/dev/shm`).
2. The original directory is renamed.
3. A link to the copy is put in place of the original directory.

When the directory is not accessed anymore (e.g. at shutdown), the
content of the `tmpfs`-copy is moved back to the original directory by:

1. Copying the content of the `tmpfs`-copy to the renamed original
   directory (sync).
2. Deleting the link to the `tmpfs`-copy.
3. Renaming the on-disk directory to its original name.
4. Deteting the `tmpfs`-copy.

To keep the risk of data-loss due to OS-crashes minimal the *sync* step
above can (should?) be performed regularly.

Requirements
------------

Zsh_, lsof_, rsync_

At the moment, *tmpfs-sync* is implemented as a  Zsh_ script which calls
lsof_ to check if releasing is safe and rsync_ to synchronize.

For integration with systemd_, the script is
accompanied by a number of `.service` and `.timer` files for `systemd`.

Usage
-----

Command line interface::

    usage: tmpfs-sync COMMAND DIR
                                                             
    possible COMMANDs:
      init        Initialize tmpfs copy
      sync        Synchronize tmpfs content to on-disk copy
      release     Release directory sync (does not sync!)
      sync-release   Synchronize and release directory sync
      status      Print status of DIR
      recover     Fix DIR after a crash

Other options are controlled via environment variables:

TMPFSDIR
    The directory where the `tmpfs` copies are created. Defaults to
    `/dev/shm/tmpfs-sync.d/`.
TMPFSSYNC_LOGLEVEL
    The verbosity of the script (1-5). Lower means more verbose. Defaults to
    3.

The `systemd` service defined in `tmpfs-sync.service` requires a config
file at `/etc/tmpfs-sync.conf` which must contain one directory per line
(no comments or such allowed). A `systemd` timer to regularly (every 2
hours) synchronize is defined in `tmpfs-sync.sync.timer` and
`tmpfs-sync.sync.service`.

.. _Zsh: http://www.zsh.org
.. _lsof: https://people.freebsd.org/~abe/
.. _rsync: http://rsync.samba.org/
.. _systemd: http://www.freedesktop.org/wiki/Software/systemd/
