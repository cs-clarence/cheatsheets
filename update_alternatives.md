# Creating symlink alternatives using `update-alternatives` on Linux
* Install a symlink alternative using `--install` flag.
```
sudo update-alternative --install {path to symlink: string} {name of alternative: string} {path to where to point that symlink alternative: string} {priority: number}
```
    * `Path To Symlink` - usually always located in `/usr/bin/` (ex: `/usr/bin/python3` for creating alternatives to multiple python versions)
    * `Name Of Alternative` - should always be the name of the symlink (ex: `python3` for `/usr/bin/python3`).
    * `Path To Where To Point That Symlink Alternative` - location of the exucable to be set as a symlink alternative
    * `Priority` - in auto mode, the alternative with the higher priority will be chosen
    * Example full command: `sudo update-alternative --install /usr/bin/python python /usr/bin/python3`

* Edit the priority of an existing symlink alternative.
    * Use the same 
    `sudo update-alternative --install {existing path to symlink: string} {existing alternative: string} {existing path to binary: string} {priority: number}` 
    but change the priority and the priority of the exisitng alternative will be changed. (will not be duplicated)
    
* Config an alternative using `--config` flag.
```
sudo update-alternative --config {name of alternative: string}
```
    * You will be prompted choose a number corresponding to the paths listed in an alternative.
    * Selecting anything higher than 0 means you will switch to manual mode.
    * Be default, alternatives are set in auto mode where paths are chosen according to highest priority.
    
* List all paths in an alternative using `--list` flag.
```
sudo update-alternatives --list {name of alternative: string}
```

* Remove all paths in an alternative using `--remove-all` flag.
```
sudo update-alternatives --remove-all {name of alternative: string}
```

* Remove one path in an alternative using `--remove` flag.
```
sudo update-alternatives --remove {name of alternative: string} {path to binary: string}
```
