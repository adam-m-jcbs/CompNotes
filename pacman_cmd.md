# `pacman` notes

These notes are for the Arch Linux package manager utility `pacman`.

The primary database operations can be summarized as:  
`-Q` is for the local package database, `-S` is for the sync database,
`-F` for files database, `-D` is for operating on the database/metadata itself

### Search packages

Append `-s` to `-S` to search sync database, `-Q` for local, and 
`-F` for files in remote packages

You're supposed to be able to use ERE's in `-s` queries, but wiki suggests this
may lead to undesired results.  Yet manpage suggests I should be able to use
regexes for `-S` and `-Q`.  To use regexes for `-F` you need to add `-x`.

Examples:
```
$ pacman -Qs "ERE" #Search local database package names/descriptions for regex ERE
$ pacman -Ss "ERE" #Search sync (remote) database package names/descriptions for regex ERE
$ pacman -Fxs "ERE" #Search remote filenames for regex ERE
$ pacman -Fs "name" #Search remote filenames for file "name"
```

### Downgrade to a cached package

<find package in `/var/cache/pacman/pkg`>  
`pacman -U /var/cache/pacman/pkg/package-old_version.pkg.tar.xz`  
<optionally add to IgnorePkg in `/etc/pacman.conf`>  

### List installed packages

`pacman -Q`

### Search installed packages and their metadata fitting a pattern

(can't do this for just package names :( )  
`pacman -Qs '^some p.ttern'`

### List all orphans 

Q: query, t: not required by any packages, d: installed as dependency  
`pacman -Qtd`

### List all packages in a group

`pacman -Qg` list all grouped packages  
`pacman -Qg aur` list packages in group 'aur'  

### Remove all orphans 

R: remove, 
n: only native pkgs that are in sync DB (helps avoid removing
manually installed pkgs),
s: search patterns provided by the query command,
q: added to previously described list cmd, I think makes the output more 
digestible as arguments  
`pacman -Rns $(pacman -Qtdq)`

### Mark a package's installation reason as explicit

This removes the package as a candidate for removal using above cmds  
`pacman -D --asexplicit pkg_name`  
Or, you can mark as a dependency to _make_ it a candidate  
`pacman -D --asdeps pkg_name`

### Clean the cache

`paccache -r`    Remove all but 3 most recently cached versions  
`paccache -rk`   Same, but do # instead of 3  
`paccache -ruk0` Remove all caches for uninstalled pkgs  

### List files installed by a package

`pacman -Ql <pkg>`  List files belonging to <pkg>  
- caution: if a package appears to have no files, check if it's a metapackage
  (e.g. jupyter, which is a metapackage including things like jupyter-notebook).
  metapackages do not own files -- only the individual packages do)

