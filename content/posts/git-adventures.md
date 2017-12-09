+++
date = "2016-10-19T06:47:39+00:00"
draft = true
title = "Git adventures"
slug = "git-adventures"

+++

## using visual studio code as default git editor

* make sure code is accessible via the PATH environment variable
* run from command line :
```
git config --global core.editor "code --wait"
```
* edit now the .gitconfig file (sure, via visual studio code):
```
git config --global -e
```
* add following:
```
[diff]
    tool = default-difftool
[difftool "default-difftool"]
    cmd = code --wait --diff $LOCAL $REMOTE
```
