Pakrat
-------

A stateless python library and CLI tool to sync and version YUM repositories
from multiple sources

What does it do?
----------------

* You invoke pakrat and pass in a few YUM repos (see below for how to do
  this)
* Pakrat fetches all packages found in the repositories you specified and saves
  them into a shared `packages` directory.
* Pakrat uses the `createrepo` library to compile metadata about the packages
  that were downloaded and saves it into a versioned directory (see explanation
  below)
* You expose the top-level directory with any HTTP server, and you now have
  access to versioned YUM repositories!

Features
-------

* Data deduplication by using a common packages directory. Each run, the only
  new data created is the repodata (usually a few Mb)
* Threaded downloading and repo creation
* CLI provides easy interface for use by CRON or similar scheduler
* Supports multiple baseurls
* Supports mirrorlists

How to use it
-------------

### Specify some *.repo file paths to load repositories from:

```python
pakrat.sync('/root/mirrors', repofiles=['/root/yumrepos/CentOS-Base.repo'])
```

### Load from a repos.d directory

You can pass in multiple directories to load:

```python
from pakrat import sync
sync('/root/mirrors', repodirs=['/root/yumrepos'])
```

### Inline repositories using `repo()`

```python
from pakrat import repo, sync
sync('/root/mirrors', repos=[
    repo('base', baseurls=['http://mirror.centos.org/centos/6/os/x86_64']),
    repo('updates', baseurls=['http://mirror.centos.org/centos/6/updates/x86_64']),
    repo('epel', baseurls=['http://dl.fedoraproject.org/pub/epel/6/x86_64'])
])
```

### Mix and Match

Keep in mind that you can mix all 3 of the above input types. You can have
repository directories, files, and in-line definitions all working together
additively.

```python
from pakrat import repo, sync
inline_repos = [
    repo('epel', baseurls=['http://dl.fedoraproject.org/pub/epel/6/x86_64'])
]
repo_dirs = [ '/etc/yum.repos.d' ]
repo_files = [ '/root/my-yum-repo.conf' ]
sync('/root/mirrors', repos=inline_repos, repodirs=repo_dirs, repofiles=repo_files)
```

CLI interface
-------------

```
Usage: pakrat [options]

Options:
  --version             show program's version number and exit
  -h, --help            show this help message and exit
  --dest=DEST           Root destination for all YUM repositories
  -d REPODIR, --repodir=REPODIR
                        A "repos.d" directory of YUM configurations.
                        (repeatable)
  -f REPOFILE, --repofile=REPOFILE
                        A YUM configuration file. (repeatable)
  -r REPOVERSION, --repoversion=REPOVERSION
                        The version of the repository to create. By default,
                        this will be the current date in format: YYYY-MM-DD
```

What's missing
--------------

* Better logging (currently console-only)
* Thread cleanup on exit, signal trapping
