# git-dat

Allow git to track large/binary files without storing their contents in git.

## Overview

Tracks a (git ignored) ``data`` directory in your existing project. When a data file is "git dat pushed", its sha1 is stored in a simple json file (git-dat.json) managed by git, and the file contents are copied to a local storage folder (outside your git repository). The local storage folder may safely be shared (e.g. via Syncthing or Bittorrent Sync) with other users of your git repository. The version of a data file people "git dat pull" depends on git-dat.json.

NOTE: this is just a proof of concept for hashing out ideas!


## Usage

All within an existing git repository...

### Initialize 

```
$ git dat init
```

This will ask you:

* what transport mechanism (currently must be ``local``)
* where on disk to store the hashed data (can be anywhere you want)
* (if no git-data.json already exists) where in the repository you want the data directory

Note that if you are using a repository that already contains a git-dat.json, you will be prompted to create the data directory (e.g. ``Could not load config file git-dat.json: Invalid data path (does 'data2' exist?)``) before you can run ``git dat init``.

### See the status of data files

```
$ git dat status
```

This will tell you about data files listed in git-dat.json that you do not already have,
and about data files you have locally that differ from (or do not exist in) git-dat.json.

```
$ git dat status
# git-dat is tracking 'my/data'
#  ! my/data/652.xlsx
#  ! my/data/556.xlsx
#
# --------------------
#
# * = local file differs from git-json.dat
#   ('push' to update git-dat.json, or 'pull --force [filename]' to overwrite)
# ! = local file missing
#   (use 'pull [filename]')
# ? = file not tracked in git-dat.json
#   (use 'push filename' to start tracking)
```

To learn about data files which are different from any particular version of git-dat.json, use ``--against treeish``. For instance, if ``git dat status`` shows you ``*`` for some files, but you don't know if that's
because you have done a ``git pull`` (causing git-dat.json to be updated with external changes) or because you have made local modifications to those files, you can pass ``--against HEAD~1`` to see which files differ from git-dat.json at HEAD~1. Any files that do not differ from e.g. HEAD~1
(or any other committed version you choose to check) can safely be deleted.

### Pull data files

```
$ git dat pull
```

This will get any data files listed in git-dat.json that you do not already
have. Depending on the amount of data to copy, this may take a few moments (at
least the time it takes to copy the files into your repository).

If you already have a file at the same path as an entry in git-dat.json, your file
will not be overwritten.

To keep your life simple, *before* you get an updated git-dat.json (e.g. via a ``git pull``), you should first
``git dat push`` (see below) any of your modified data files.


### Push data files

```
$ git dat push
```

This will push all data files already tracked in git-dat.json. After pushing the files, you
should commit and push git-dat.json to share the changes with others.

To push an untracked file, specify its name e.g. ``git dat push data/testfile``. Or e.g. to push
all csv files in a particular directory, ``git dat push data/path/*.csv`` (note: no recursive processing
available yet).

### Stop tracking a data file

```
$ git dat rm filename
```

This removes the file from git-dat.json, but does not delete your copy of the file. (XXX rename command)

## Future work

Make it easier for the user to replace files for which the only change is from an external modification (i.e. from an external update to git-dat.json).

## Background

XXX Something about the problems of storing binary files in git (repository size, slow, etc).

### Alternatives to git-dat?

I'd be happy to hear about things I've got wrong in the following...

* Why not GitHub LFS (https://git-lfs.github.com/), git-media (https://github.com/alebedev/git-media), or git-bigfile (https://github.com/beenje/git-bigfile)? Firstly, the smudge/clean filters must operate frequently, which is bad news if you have many data files. Secondly, after a ``git media sync``, something like ``git update-index --assume-unchanged -- (media files)`` is run (e.g. https://github.com/alebedev/git-media/blob/master/lib/git-media/sync.rb#L52), which I think makes it easy to mistakenly delete your data (e.g. via git stash). A further problem I have with the smudge/clean filters is that they don't seem to operate consistently across different versions of git, although I did not analyze this problem carefully and presumably it will gradually disappear if it hasn't already (but at least with git 1.8.4 on windows, the filters did not run for all necessary operations). (XXX to consider: using git should allow you to take advantage of git's speed, e.g. in very quickly determining which files in a huge directory have changed. XXX to consider: wouldn't ``git update-index`` without the ``--assume-unchanged`` be a better choice for git-media to use?)

* Why not git-fit (https://github.com/dailymuse/git-fit)? git-dat is based on git-fit. git-fit avoids smudge/clean filters, stores metadata in a simple json file, and is written in a language that can provide fast static compiled binaries for various platforms. However, git-fit does not work on Windows (XXX check again) and does not support local transport. Local transport is very fast, and is also useful in conjunction with e.g. peer-to-peer file synchronization (e.g. Syncthing or bittorrent sync). git-dat copies files (i.e. files named with the hash of their contents) to a local directory (rather than s3), and that directory can then be synchronized automatically by a file synchronization tool. Only requiring a local copy (rather than e.g. upload to s3) means working with many/large data files is very fast. Furthermore, the synchronization is very fast for machines on the same network, and does not rely on an external service like s3.
