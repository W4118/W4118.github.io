# OS Developer Workflow

From now on, all assignments will need to be done in the class VM, and will involve working within a complete copy of the Linux Kernel. After hearing the frustrations of students in past years who have tried to do their assignments directly in the VMWare console, using a barebones text editor, we have created a guide for building out an improved developer workflow. We present instructions for how to configure remote ssh, set up git shortcuts, and configure a robust Vim dev environment.

By the end of this guide, you will be able to:

 - Use `ssh` to work directly in your host OS
 - Have semantic auto-complete, file grepping, function GOTOs and more in your vim
 - Impress non-OS friends with your vim wizardry


## SSH into vm

Most students are more comfortable working in their host machine than in the VM. We can enable this by configuring remote ssh.

First, in your VM execute the following command to get its remote address:
```
/sbin/ifconfig eth0 | grep 'inet addr' | awk '{print $2}' | cut -d ':' -f 2
```

Copy the address and in the `~/.ssh/config` file on your __host machine__, add:

```
Host vm
  HostName <YOUR-VM-ADDRESS>
  User w4118
```

Now, to ssh into your VM just type `ssh vm`.

__For the remainder of this guide, all changes will be made in the class virtual machine.__

## Gitconfigs:

To optimize your git workflow, you can add shortcuts to the   `~/.gitconfig` file on your VM. For example, here is my config file:

```
[user]
    name = Andrew Aday
    email = aza2112@columbia.edu
[color]
    diff = auto
    status = auto
    branch = auto
[core]
    editor = vim
[alias]
    stash-unapply = !git stash show -p | git apply -R
    co = checkout
    br = branch
    ci = commit
    st = status
    last = log -1 HEAD
    plog = log --pretty=oneline --abbrev-commit
    conflict = diff --name-only --diff-filter=U  # View all files that have merge conflicts
```


# Vim Setup

__Non-vim users__: If you have a cool emacs/atom/etc setup, feel free to send us a guide and we will share it on the course website.

The remainder of this guide will be dedicated to building out a robust vim developer environment, so only continue if are planning to use vim for this class.

## The Philosophy of Vim

Before diving into setup, I need to assert a critical point about working in vim: __don't use tabs__! Instead, use __buffers__ and __windows__.

Many of you taking this class have come fresh out of AP, where you first learned to use vim. When I took OS I was in the same boat; all I had with me was Jae's `.vimrc`, a few keyboard shortcuts, and many bad habits. Unfortunately, no class really teaches *how* to use vim, and as a result its easy to become glued to functional, but inefficient methods.
For instance, if you repeatedly type `:tabn`/`:tabp` to navigate files, or if you have 10 different terminal windows open, each ssh'd to the same remote directory with 10 instances of vim open on 10 different files (aka me in AP)...your workflow is probably suboptimal.

Thus, beyond installing plugins, the aim of this guide is to enforce efficient vim usage patterns that will save you tremendous amounts of time in the long run.

People who are most familiar with IDEs or sublime/atom expect to use tabs with vim because that is how these other editors work: the relationship is 1 tab per 1 file, and opening/closing a file requires opening/closing a tab. Vim however is not meant to be used this way--it was designed to work within a *single* terminal tab by utilizing multiple windows + multiple buffers.

Definitions:
- A __Buffer__ is simply the raw data associated with an open file.
- A __Window__ is a visual display for a buffer. A tab in Vim must have >= 1 window, and each window displays exactly 1 buffer.

Here is an approximate translation from actions in sublime --> actions in vim:

- Open `file.c` in a new tab --> Open `file.c` in a new buffer, and move your window to display this buffer
- Close tab of `file.c` --> Close the `file.c` buffer
- Open `file.c` in a new column --> Open `file.c` in a new window with either `:vsp file.c` or `:sp file.c`
- etc.

Here's an example of what this should look like. Note that I never open a new tab. Instead, I have 3 buffers open (top line) and 1 or 2 windows open. To view a file, I simply navigate the window I want to the appropriate buffer.

![Demo](images/philosophy.gif)

## Install Pathogen

Now let's get to installing plugins.

[Pathogen](https://github.com/tpope/vim-pathogen) is a plugin manager for Vim. We'll be using pathogen to install the remaining plugins, so do this first! Run this:

```
mkdir -p ~/.vim/autoload ~/.vim/bundle && \
curl -LSso ~/.vim/autoload/pathogen.vim https://tpo.pe/pathogen.vim
```

And add the following to your `~/.vimrc`:

```
execute pathogen#infect()
```

Now all plugins can be installed inside `~/.vim/bundle` and they will be automatically added to your vim `runtimepath`.

### Remapping Leader

Vim has the concept of a "leader" key, which is used to program personal keyboard shortcuts. By default, it is mapped to `\`, which is a little difficult to read. I suggest mapping it to either `<space>` or comma `,` (I prefer space). In your `~/.vimrc`, write either

```
let mapleader ="\<Space>"
```

or

```
let mapleader = ","
```

We will be using leader keys through the rest of the guide. When you see something like `nmap <leader>l :bnext<CR>`, it means we are mapping the keyboard shortcut `<leader>` + `l` to the action `:bnext<CR>`.

## Bufferline Display:

[Airline](https://github.com/vim-airline/vim-airline) is a package to display status information such as git branch and what buffer you are currently viewing. To install, run

```
git clone https://github.com/vim-airline/vim-airline ~/.vim/bundle/vim-airline
```

And in your `~/.vimrc` add

```
" Airline
let g:airline#extensions#tabline#enabled = 1 " Enable the list of buffers
let g:airline#extensions#tabline#fnamemod = ':t' " Show just the filename
```

When it's all done, you should have a header that looks like:

![buffer](images/buffer)

To easily maneuver around these buffers, add this to your `~/.vimrc`:

```
" -----------Buffer Management---------------
set hidden " Allow buffers to be hidden if you've modified a buffer

" Move to the next buffer
nmap <leader>l :bnext<CR>

" Move to the previous buffer
nmap <leader>h :bprevious<CR>

" Close the current buffer and move to the previous one
" This replicates the idea of closing a tab
nmap <leader>q :bp <BAR> bd #<CR>

" Show all open buffers and their status
nmap <leader>bl :ls<CR>
```

Now to navigate your window left/right between buffers, just press `<leader>` + `l` or `<leader>` + `r`. To close a buffer, press `<leader>` + `q`.

## Window navigation

To open a new window, enter:

- `:vsp` to open a vertically-split window
- `:sp` to open a horizontally-split window

To navigate between windows, use: `ctrl-w` + `h|j|k|l` to navigate left|down|up|right.
Alternatively, add this to your `~/.vimrc` so that you can move between windows using arrow keys:

```
" Use arrow keys to navigate window splits
nnoremap <silent> <Right> :wincmd l <CR>
nnoremap <silent> <Left> :wincmd h <CR>
noremap <silent> <Up> :wincmd k <CR>
noremap <silent> <Down> :wincmd j <CR>
```

To close a window, press `ctrl-w` + `c`

## ctrl-p and Nerdtree

The Linux kernel is a massive code base, so for easy navigation we'll want to add a filename grepper ([ctrl-p](https://github.com/kien/ctrlp.vim)) and file tree ([Nerdtree](https://github.com/scrooloose/nerdtree))

```
git clone https://github.com/ctrlpvim/ctrlp.vim ~/.vim/bundle/vim-ctrlp
git clone https://github.com/scrooloose/nerdtree.git ~/.vim/bundle/nerdtree
```

Now add the following to your `~/.vimrc`:

```
" ctrl-p
let g:ctrlp_custom_ignore = {
  \ 'dir':  '\v[\/](\.(git|hg|svn)|\_site)$',
  \ 'file': '\v\.(exe|so|dll|class|png|jpg|jpeg)$',
\}

" Use the nearest .git|.svn|.hg|.bzr directory as the cwd
let g:ctrlp_working_path_mode = 'r'

nmap <leader>p :CtrlP<cr>  " enter file search mode

" Nerdtree
autocmd vimenter * NERDTree
autocmd StdinReadPre * let s:std_in=1
autocmd VimEnter * if argc() == 0 && !exists("s:std_in") | NERDTree | endif
autocmd bufenter * if (winnr("$") == 1 && exists("b:NERDTree") && b:NERDTree.isTabTree()) | q | endif
map <C-n> :NERDTreeToggle<CR>  " open and close file tree
nmap <leader>n :NERDTreeFind<CR>  " open current buffer in file tree
```

Now if you want to search for and open a file, press `<leader>` + `p`. This is analogous to `cmd+r` in Sublime.

Nerdtree will show a file tree in the window on the left of your screen. To collapse/expand it, press `ctrl` + `n`. To expose your current working file in the file tree, press `<leader>` + `n`.

Altogether it looks like this:
![File Navigation](images/file_nav.gif)

## YCM

Next we are going to add semantic auto-complete through a plugin called [YouCompleteMe](https://valloric.github.io/YouCompleteMe/). This part is a little involved, so here's a demo first so you can decide if it's worth the setup:

![YCM](images/ycm.gif)

Before installing YCM, we need to rebuild vim from source to get a compatible version.

First install some packages:
```
sudo apt-get update && apt-get install libncurses5-dev libgnome2-dev \
    libgnomeui-dev libgtk2.0-dev libatk1.0-dev libbonoboui2-dev \
    libcairo2-dev libx11-dev libxpm-dev libxt-dev python-dev \
    python3-dev ruby-dev lua5.1 lua5.1-dev libperl-dev git
```
Install clang (make take a while)
```
sudo apt-get install clang
```

Uninstall your current vim:
```
sudo apt-get remove vim vim-runtime gvim
```

Install python2:
```
sudo apt-get install python python-dev
```

And build vim!
```
cd ~
git clone https://github.com/vim/vim.git
cd vim
./configure --with-features=huge \
            --enable-multibyte \
            --enable-rubyinterp=yes \
            --enable-pythoninterp=yes \
            --with-python-config-dir=/usr/lib/python2.7/config \
            --enable-perlinterp=yes \
            --enable-luainterp=yes \
            --enable-gui=gtk2 \
            --enable-cscope \
            --prefix=/usr/local
make VIMRUNTIMEDIR=/usr/local/share/vim/vim80
sudo make install
```

Now set your updated vim as default editor:
```
sudo update-alternatives --install /usr/bin/editor editor /usr/local/bin/vim 1
sudo update-alternatives --set editor /usr/local/bin/vim
sudo update-alternatives --install /usr/bin/vi vi /usr/local/bin/vim 1
sudo update-alternatives --set vi /usr/local/bin/vim
```

Finally, install YCM
```
cd ~/.vim/bundle
git clone https://github.com/Valloric/YouCompleteMe.git
cd YouCompleteMe
git submodule update --init --recursive
`./install.py --clang-completer`
```

In the template for your homework assignments, we will include a file `kernel/.ycm_extra_conf.py` that enables YCM. If you want to see how this works, check out [YCM-Generator](https://github.com/rdnetto/YCM-Generator) and the original YCM github page.


__Important Note__: for YCM in the kernel to work properly you need to be *inside* the kernel directory when you activate vim. e.g. `cd <path-to-homework-assignment>/kernel && vim`

## Cscope

This is the final step: Function GOTOs and efficiently browing code.

[Cscope](http://cscope.sourceforge.net/) is a code browser that works in your terminal and within vim. It is far more powerful than a standard grepper (such as the one at http://elixir.free-electrons.com/linux/v3.10/). For example, Cscope can answer:

- Where is this variable used?
- What is the value of this preprocessor symbol?
- Where is this function in the source files?
- What functions call this function?
- What functions are called by this function?
- Where does the message "out of space" come from?
- Where is this source file in the directory structure?
- What files include this header file?

To install Cscope, we need to build from source. First, download the latest stable version:

```
cd ~ && wget https://sourceforge.net/projects/cscope/files/cscope/15.8a/cscope-15.8a.tar.gz/download -O cscope-15.8a.tar.gz
tar xvzf cscope-15.8a.tar.gz
```

To install:
```
cd ~/cscope-15.8a
./configure
make
sudo make install
```

Verify installation succeeded by running `cscope`. This should open up the Cscope browser in your terminal window. To exit, use `ctrl-d`.

In the template code for all your written assignments, we will include the following files in `<PATH-TO-HW-REPO>/kernel`:

- `cscope.files`: lists all kernel files Cscope will include in its project database
- `cscope.files`, `cscope.in.out`, `cscope.out`: Files comprising the Cscope database. Do not touch these!
  - If you do accidentally delete them, run `cscope -b -q -k` in the kernel directory to regenerate.

To use the Cscope browser:

```
cd <PATH-TO-HW-REPO>/kernel
cscope -d
```

Use `<Tab>` to alternate between the menu and the list of matching lines. See [manpage](http://cscope.sourceforge.net/cscope_man_page.html) for more usage instructions of the Cscope browser.

Demo:

![Cscope](images/cscope.gif)

To enable vim support, we need to add a new `cscope_maps.vim` file. First, if it doesn't already exist, run
```
mkdir ~/.vim/plugin/
```
All `.vim` files in this directory will automatically be sourced into your `~/.vimrc`. Then run:

```
cd ~/.vim/plugin && wget http://cscope.sourceforge.net/cscope_maps.vim
```

See this [tutorial](http://cscope.sourceforge.net/cscope_vim_tutorial.html) for more usage details. For now, I will tell you the single most useful feature we have just added: __Function GOTOs__! Now, when your cursor is over a function call, enter `ctrl + ]` and you will automatically jump to the function declaration in a new buffer. And to move your cursor back and forth, use `ctrl + o` (backwards) and `ctrl + i` (forwards). See demo video below:

![Function GOTO Demo](images/function_goto.gif)

My last recommendation is to use 2 terminal windows when working: one dedicated to vim/writing code, and the other dedicated to cscope/browsing code.

## Fin

And that completes this developer workflow guide. Congrats if you've made it this far; hopefully the time spent reading this will be vastly outweighed by time you'll have saved in this class and beyond.

Please send any suggestions/errata to w4118@lists.cs.columbia.edu
