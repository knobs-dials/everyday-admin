Various shell scripts I've made in the process of having sysadmin as a side job.

Sorted roughly in order of how much I seem to actually use them.




file-open-permissions
===

Helper that does chmod and chown, that you can use recursively around directories.  For example,

```
   file-open-permissions --user uname --group gname PATH
```
...is roughly equivalent to:

```
   find PATH -type d -print0 | xargs -0 chown chmod ug+rwx,o+rx
   find PATH -type f -print0 | xargs -0 chown chmod ug+rw,o+r
   chmod uname:gname -R PATH
```
TODO: allow option to specifically include/exclude setting the directory we point to, not just its entries.



## file-largest

List the largest files under the given path(s).

```
    # file-largest /opt /tmp
    Reading '/opt'...
    Reading '/tmp'...
    Done
      100MB '/opt/google/chrome/chrome'
      101MB '/opt/scipion-2016-01-30/.git/objects/pack/pack-043cf8c2d2435990ca2a8c6d489dd7c454eb29f0.pack'
      104MB '/opt/scipion/.git/objects/pack/pack-f826a4cffb9f8afd8ade60c07d33a349d6ce4c28.pack'
      124MB '/opt/PEET/ParticleRuntime/v84/sys/jxbrowser-chromium/glnxa64/chromium/libjxbrowser-chromium-lib.so'
      177MB '/opt/PEET/ParticleRuntime/v84/bin/glnxa64/libnppi.so.6.0.37'
      239MB '/opt/coot-Linux-x86_64-ubuntu-14.04-gtk2-python/libexec/coot-bin'
      7.7GB '/opt/R2015b_glnxa64.iso'    
```



## file-recent

Reports the most recently altered files under a path (via ctime/mtime).
Helps find what you / other people / programs were recently working on.

```
    # file-recent ~
    Looking in '/root'
       including files where max(ctime,mtime) is in the last 100 days
       will sort results by age
       and show the 5 most recent files
          2mo   2wk :  /root/.mozilla/firefox/Crash Reports/InstallTime20160606114208
          5dy   2hr :  /root/.lesshst
          1hr  24min:  /root/.Xauthority
         40min 33sec:  /root/.bash_history
         35min 33sec:  /root/.local/share/mc/history
```



## ssh-permission-check

You know when ssh refuses to do something because it doesn't like your permissions?
This suggests chmod commands to fix that   (and a few restrictive things, just because)
(TODO: also consider chowns, in case you have the wrong owner)

```
    $ sudo ssh-permission-check
    chmod 750 '/root/.ssh'                  # currently: 770
    chmod 600 '/root/.ssh/bridgetrust.pub'  # currently: 644
    # Checked users: repository, root, tunneler
```






## strace-openedfiles

Runs and straces a given command.
For all open() calls that it does, prints unique existing filenames.

Discards the command's stdout
TODO: allow printing it on stderr instead, or logging it elsewhere.

Also stats these files, and when they're large than 1MB prints LARGE,
because this was written to see which files a grepalike was spending so much time on:
```
# strace-openedfiles  ag work_mem | grep ^LARGE
LARGE /usr/lib/locale/locale-archive
LARGE ./solardata/solar__.csv
LARGE ./solardata/solar.sql
```

TODO: consider things that would be aliases (e.g. alias ag='ag --path-to-agignore ~/.agignore')


## lsof-interesting

lsof with a grep that filters out most not-really-file stuff.
Verrry simple.


## otherpeople

Tries to summarize what various people are up to 
* shows process names -- and filters out boring stuff, particularly for root
* shows working dir for shells (and some others)
* CPU / wait state if worth mentioning

```
    # otherpeople
    root
         4 *  bash
                in '/var/www/coding/github-sys-tools'
                in '/root'
                in '/var/www/coding/github-file-tools'
         3 *  sshd
         3 *  smbd
         2 *  tmux
              fancontrol
              wolforward
              fail2ban-server
              john                99%CPU
                in '/root'
              master
              emacs
              docker

    tunneler
              sshd

    me
              Xtightvnc
              x-window-manage
              xstartup
              dd                  52%CPU  waiting on IO  <--------
              bash
              in '/home/me'
```




## svn-summarize-recent

A script with a few svn calls that shows the commit message and list of files involved for

- the difference between repository HEAD and local code
- the difference between the last revisions we know of

You'll want a passwordless auth for this, or it'll ask you a dozen times, once for each part.



## svn-headdiff

Shows the diff between repository HEAD and local code





## file-caseclash

```
    $ file-caseclash /data
    # Issues under '/data/NerdDocs/learning'
      '/data/NerdDocs/learning/[learn] N-grams, smoothing.pdf'
      '/data/NerdDocs/learning/[learn] N-grams, Smoothing.pdf'

    # Issues under '/data/Installs/Game/Older - C64/unsorted/'
      '/data/Installs/Game/Older - C64/unsorted/README.TXT
      '/data/Installs/Game/Older - C64/unsorted/Readme.txt
      '/data/Installs/Game/Older - C64/unsorted/README.txt
      '/data/Installs/Game/Older - C64/unsorted/readme.txt
```

Checks directories for file and directory entries that will be confusing to case-insensitive filesystem APIs ( such as windows's)





## file-stable

When you want to handle files soon after they come in, but not _while_ still being written,
a bit of backoff time is useful.

Meant to feed things to xargs, in that it checks that files
that match a given shellglob, 
and that sit under a given directory.
have not changed size/ctime in a while (currently hardcoded to ~10 seconds),
and then will print them once and never again.

I use this to compress incoming data on a server after it's fairly certain they are complete:

```
   file-stable '*.dat' /data/archive | xargs -t -n 1 -P 3 pigz -3
```




## file-cleanafter-rsync

If you break off rsync, it can leave behind temporary files, which are lonely (no file alongside)
or stale (much older than the real file alongside).

This looks for them, checks that they're not recent, and reports and optionally removes them.



## file-readin

Reads a file. So that it's in the page cache.
(note: you may care to know about the vmtouch utility)

```
    # readin -w png -w jpg -r /data/images
    Reading: /data/images/eb8f2be459aad473b5dd64680d78b810ddf2a8cf.jpg
    Reading: /data/images/bdff38ce75cbbd76803c4e25478faa64553d0b08.jpg  
```


## file-summarize-extensions

Reports total amount and size, per file extension under a path.

Helps answer questions like 
* "if I delete just the cache files, how much space would that free?"
* "how useful might it be compress stuff in here?"
* "how long would this take to copy just the gzipped version?"

```
    # file-summarize-extensions /home
    Final report (top 10, by count):
      html:          138.5MiB,    6088 files
      None:          10.59GiB,    4696 files
       gif:           41.1MiB,    3005 files
      java:           19.5MiB,    1944 files
       pyc:          11.78MiB,    1070 files
     class:           2.98MiB,    1001 files
       png:           72.1MiB,     922 files
       htm:          12.61MiB,     631 files
       jpg:          295.3MiB,     631 files
       pdf:           1.44GiB,     563 files
```


## file-totalsize

Takes a list of files on stdin, sums their size and count.

I mostly use this as a more targeted variant of file-summarize-extensions

```
    # /tmp files that haven't changed in the last two weeks
    # find /tmp -ctime +14 | file-totalsize
    63.4MiB / 66.5MB  (66491289 bytes)  in 240 files

    # JPGs from cameras
    # find /data/images -iname '*.jp*' | egrep ^DSC | file-totalsize
    121MiB / 127MB  (127194089 bytes)  in 15 files
```



## file-estimate-homediruse

As a sysadmin, it's nice to quickly see who your inactive users and your abusers are.

```
    # file-estimate-homediruse
    User 'myusername'
      INFO: Size:          9.7GB
      INFO: Youngest file:  2mo   5dy
      Some of the larger and recent files:
         3mo   3dy        Dropbox/morework/titan.png
         4mo   7hr        Dropbox/Zeck-PD6.12_8.12_12.16 pwrmix.pdf
         3mo   3wk        Dropbox/relay_h-550_en.pdf
         3mo   1dy        Dropbox/CMaps/project.idx

    User 'repository'
      INFO: Size:          13.2GB
      INFO: Youngest file:  4min 9sec
      Some of the larger and recent files:
         4min 9sec        workscripts/db/rep-cache.db
        29min 13sec       workscripts/db/revs/0/707
        19hr  10min       rep/db/revs/0/930
        19hr  13min       supportingdata/db/revs/0/6

    User 'tunneler'
       INFO: Size:          6KB
       INFO: Youngest file:  7mo   3wk
       No recently modified files.                                                  
```



## TODO

Cleanup for all. 

Option parsing for all.

