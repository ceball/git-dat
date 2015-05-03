# git-dat

Allow git to track large/binary files without storing their contents in git.

## Usage

All within an existing git repository...

### Initialize 

```
$ git dat init
```

This will ask you:

* what transport mechanism (currently must be ``local``)
* where on disk to store the hashed data (can be anywhere you want)
* where in the repository you want the data directory (if one has not already been set)

### See the status of data files

```
$ git dat status
```

This will tell you about data files listed in git-dat.json that you do not already have,
and about data files you have locally that differ from (or do not exist in) git-dat.json.


### Pull data files

```
$ git dat pull
```

This will get any data files listed in git-dat.json that you do not already
have. 

If you already have a file at the same path as an entry in git-dat.json, your file
will not be overwritten. You can force overwriting of your own files by passing
``--force``. 

XXX Before you get an updated git-dat.json (e.g. via a ``git pull``), you should first
``git dat push`` any of your modified data files (see below).

XXX overwrite your own files (e.g. via ``git dat pull --force``


### Push data files

XXX

## Why?

XXX Something about the problems of storing binary files in git (repository size, slow, etc).

### Why git-dat rather than alternatives?

I'd be happy to hear about things I've got wrong in the following...

* Why not GitHub LFS (https://git-lfs.github.com/), git-media (https://github.com/alebedev/git-media), or git-bigfile (https://github.com/beenje/git-bigfile)? Firstly, the smudge/clean filters must operate frequently, which is bad news if you have many data files. Secondly, after a ``git media sync``, something like ``git update-index --assume-unchanged -- (media files)`` is run (e.g. https://github.com/alebedev/git-media/blob/master/lib/git-media/sync.rb#L52), which I think makes it easy to mistakenly delete your data (e.g. via git stash). A further problem I have with the smudge/clean filters is that they don't seem to operate consistently across different versions of git, although I did not analyze this problem carefully and presumably it will gradually disappear if it hasn't already (but at least with git 1.8.4 on windows, the filters did not run for all necessary operations).

* Why not git-fit: like git-dat, this avoids smudge/clean filters, stores metadata in a simple json file, and is written in a language that can provide a fast static compiled binary for various platforms. However, does not work on Windows, and does not work with local transport. Local transport seems useful in conjunction with e.g. peer-to-peer file synchronization (e.g. Syncthing or bittorrent sync). git-dat copies files (i.e. files named with the hash of their contents) to a local directory (rather than s3), and that directory can then be synchronized automatically by a file synchronization tool. Only requiring a local copy (rather than e.g. upload to s3) means working with many/large data files is very fast. Furthermore, the synchronization is very fast for machines on the same network, and does not rely on an external service like s3.
