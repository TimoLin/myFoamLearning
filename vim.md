Customize Vim for OpenFOAM
==========================
## [Foam Comments](https://github.com/TimoLin/foam-comments)  
A Comment/Un-Comment plugin for OpenFOAM cases' files using short-key [Ctrl-C/X].  
```vim
Plug 'TimoLin/foam-comments'
```
## [vim-openfoam](https://github.com/phresher/vim-openfoam)  
Highlighting OpenFOAM keywords for OpenFOAM cases' files.  
```vim
Plug 'phresher/vim-openfoam'
```
## [YouCompleteMe](https://github.com/ycm-core/YouCompleteMe)  
Environment  
- Ubuntu 18.04
- VIM 8.0 
- Python 3.6.9

### 1. Installation
Add this to `.vimrc`
```vim
Plug 'Valloric/YouCompleteMe', { 'commit':'d98f896' }
```
Install/Compile:
```shell
python3 install.py --clangd-completer
```

### 2. Configuration for OpenFOAM
Use [YCM-Generator](https://github.com/rdnetto/YCM-Generator).
