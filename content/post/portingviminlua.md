---
title: "Portingviminlua"
#description: <descriptive text here>
date: 2021-04-27T17:06:45+05:30
draft: true
toc: false
image: ""
tags: [Neovim, Vim, Lua, Tutorial]
categories: []
---

# Porting Neovim config in Lua ... Why and How ?
<!--more-->

at some point of time , we all have to tweak our configs , and time by time , we all make our configs more complex . 

## So today , we are going to port our neovim Config in Lua language ...

### But , Why ?

well the most simple answer would be , to make more complex and functional config . When Bram Moolenaar made vim , he made vimscript to write configs for vim . But the main fact is , that it is not functional language so we cann't write complex code to do certain tasks.

**Lua in the rescue**

Neovim supports Lua from default and coolest part is it is a functional programming language , an example would be , ***you can write games in Lua with Love2d game engine***.
So we can write multi-functional and complex code with ease. So let's jump into it .

### How ?

Now comes up the real part , tide your seatbelts and let's dive into it ...

#### The Base

Neovim supports `init.lua` file instead of `init.vim` , so we are gonna make a `init.lua` file inside `~/.config/nvim/`
> Remember to backup or delete your `init.vim` if existed

#### Other Lua files

To manage your config with ease , you should break down your config with naming conventions for different , tasks , like for mappings , `mappings.lua` etc.

The Basic File structure of nvim config is like that:

```text
📂 ~/.config/nvim
├── 📁 after
├── 📁 ftplugin
├── 📂 lua
│  ├── 🌑 myluamodule.lua
│  └── 📂 other_modules
│     ├── 🌑 anothermodule.lua
│     └── 🌑 init.lua
├── 📁 pack
├── 📁 plugin
├── 📁 syntax
└── 🇻 init.vim
```
so lua files are typically located inside `Lua/` and it can have submoules as well.

To load `myluamodule.lua` we have to call:
```lua
require('myluamodule')
```

similarly to load `other_modules/anothermodule.lua` we have to call:
```lua
require('other_modules.anothermodule')
```
Notice that while requiring these files, we are not calling `.lua` extension.

another thing to Remember if you are calling any `lua` file in your `init.vim` , you have to append `lua` before everyline:

```vim
lua require('myluamodule')

lua require('other_modules.anothermodule')
```
just like this.

Another thing to remenber if you want to call the whole `other_modules/` at once , you can simply call:
```lua
require('other_modules')
```
but in that case you have to have a `init.lua` under that directory.

##### Tip

to call a bigger chunk of `Lua` code inside your `init.vim` , just call like this:
```vim
lua << EOF
local mod = require('myluamodule')
local chars = {'\', '|', '~'}

for s, m in ipairs(chars) do
	mod.method(v)
end

print(chars)
EOF
```

To source a lua file , just call in command mode:
```vim
:luafile % # for the current file

:luafile some/thing/file.lua # for a certain file
```

##### luafile vs require():

You might be wondering what the difference between `lua require()` and `luafile` is and whether you should use one over the other. They have different use cases:

- `require()`:
    - is a built-in Lua function. It allows you to take advantage of Lua's module system
    - searches for modules in `lua` folders in your `runtimepath`
    - keeps track of what modules have been loaded and prevents a script from being parsed and executed a second time. If you change the file containing the code for a module and try to `require()` it a second time while Neovim is running, the module will not actually update
- `:luafile`:
    - is an Ex command. It does not support modules
    - takes a path that is either absolute or relative to the working directory of the current window
    - executes the contents of a script regardless of whether it has been executed before


#### The sets

The most common part of any `vim configuratins` is sets, like:
```vim
set smarttab 
set number
set relativenumber
set cursorline
set termguicolors
set nobackup
set filetype plugin on
set indent on
set syntex on
```

To achieve the same thing in `lua` we have a different but more verbose approach:
- `vim.o.{option}`: global options
- `vim.bo.{option}`: buffer-local options
- `vim.wo.{option}`: window-local options
- `vim.b.{name}`: buffer variables
- `vim.w.{name}`: window variables
- `vim.t.{name}`: tabpage variables
- `vim.v.{name}`: predefined Vim variables
- `vim.env.{name}`: environment variables

So now the sets will be like:
```lua
-- Ignorecases
vim.o.ignorecase = true
vim.o.smartcase = true

-- Search
vim.o.hlsearch = true
vim.o.incsearch = true
vim.o.inccommand = "split"

-- Vim UI
vim.o.smartindent = true
vim.o.wrap = false
vim.o.colorcolumn = '120'
vim.o.showcmd = true
vim.o.tabstop = 4
vim.o.softtabstop = 4
vim.o.signcolumn="yes"
vim.o.scrolloff = 8
vim.o.showmode = false
vim.o.completeopt = "menuone,noinsert,noselect"
vim.o.hidden = true
-- vim.o.include
vim.o.display = "lastline"
vim.o.backspace = "indent,eol,start"

-- Visuals
vim.o.syntax = "enable"
-- vim.o.filetype = "plugin on"
vim.o.t_Co="256"
-- vim.api.nvim_set_option('t_Co',256)
vim.o.termguicolors = true
-- vim.o.encodingvim.o.hidden = true
vim.o.include = ""
vim.o.display = "lastline"
vim.o.encoding="utf-8"
-- vim.wo.winblend = 100
-- vim.api.nvim_exec([[set winblend=100]], true)
-- Numbers
vim.wo.number = true
vim.wo.relativenumber = true
vim.wo.cursorline = true
-- vim.wo.cursorcolumn = true

-- Utils
vim.o.compatible = false
vim.o.mouse='a'
vim.o.autoindent = true
vim.o.backup = false
vim.o.undofile = true
vim.o.undodir="~/.vim/undodir"
vim.o.swapfile = false
vim.o.shada="!,'1000,<50,s10,h"
vim.o.viminfo="'100,n$HOME/.vim/files/info/viminfo"
vim.o.clipboard = "unnamedplus"

-- Times
vim.o.ttimeoutlen = 50
vim.o.updatetime = 100
vim.o.shortmess = "I"
vim.o.laststatus = 2
```
just like that ...

#### The Keymappings

In vimscript we write , the Keymappings like:
```vim
set Leader = " "

nnoremap <Leader>tg :echo "tg"<CR>
```

But in `Lua` , we can call it like this:
```lua
vim.g.mapleader = " "

vim.api.nvim_set_keymap('n', '<Leader>tg', ':echo "tg"<CR>', { noremap = true , silent = false , expr = false})
```
Tutorial ,

the `vim.api.nvim_set_keymap` is a Meta-accessors , to make a keymap in lua . Inside , the first parenthesis `()`
- 'n' , calls for normal mode . same thing for , 'i' for insert mode, 't' for terminal mode
- '<Leader>tg' or the second parameter is the trigger that , you are calling to do a task
- ':echo "tg"<CR>' or the third parameter is the actual function or task you are gonna call after make the trigger.
- The fourth parameter { noremap = true , silent = false , expr = false} is gonna take your accessors to make it is , call `true` for those things which will gonna work while calling the trigger.
>- call , the first, second and third parameter in quotes. 
>
>- if any of `noremap, silent, expr` is false , you don't need to call it though .

#### Calling `vim command` inside `Lua`
To call vim command in lua, do:
```lua
vim.cmd("the vim command ...")
```
To call a multiline vim command in lua, call:
```lua
vim.cmd([[
this is a
multiline
vim
command ...
]])
```
#### Autocommands
Neovim still doesn't support calling Autocommands in lua ,
To  achieve autocommand in `lua`

- first approach
```lua
vim.cmd([[
augroup highlight_yank
    autocmd!
    autocmd TextYankPost * silent! lua require'vim.highlight'.on_yank({timeout = 40})
augroup END
]])
```
> but for some reason, this doesn't work everytime.

- second approach
```lua
vim.api.nvim_exec([[
augroup highlight_yank
    autocmd!
    autocmd TextYankPost * silent! lua require'vim.highlight'.on_yank({timeout = 40})
augroup END
]], true)
```
the `vim.api.nvim_exec` is another Meta-accessor for calling a nvim command inside lua.
> you have specify `true` after the ]], inside `vim.api.nvim_exec`
#### Functions, loops, if
To call a function or loops or if inside `lua`

- first approach
```lua
vim.cmd([[
function! GitStatus()
  let [a,m,r] = GitGutterGetHunkSummary()
  return printf('+%d ~%d -%d', a, m, r)
endfunction
set statusline+=%{GitStatus()}
]])

vim.cmd([[
if (empty($TMUX))
    if (has("nvim"))
	let $NVIM_TUI_ENABLE_TRUE_COLOR=1
    endif
    if (has("termguicolors"))
	set termguicolors
    endif
endif
]])
```
> but for some reason, if this doesn't work .

- second approach
```lua
vim.api.nvim_exec([[
function! GitStatus()
  let [a,m,r] = GitGutterGetHunkSummary()
  return printf('+%d ~%d -%d', a, m, r)
endfunction
set statusline+=%{GitStatus()}
]], true)

vim.api.nvim_exec([[
if (empty($TMUX))
    if (has("nvim"))
	let $NVIM_TUI_ENABLE_TRUE_COLOR=1
    endif
    if (has("termguicolors"))
	set termguicolors
    endif
endif
]], true)
```

#### Source any file `.vim` file in lua
just call:
```lua
vim.cmd("source the/file/path/here.lua")
```
## Some References
- [A verbose and full guide for lua Tutorial](https://github.com/nanotee/nvim-lua-guide)
- [A great talk by Nvim core developer **Tj Devries**](https://youtu.be/IP3J56sKtn0)
- [List of some plugins written in Lua by **rockerBOO**](https://github.com/rockerBOO/awesome-neovim)
- [My Nvim Dotfiles](https://github.com/arpangreat/dotfiles/tree/master/nvim)
- [The greatest file picker ever existed](https://github.com/nvim-telescope/telescope.nvim)

### If you have any, question or confusion about anything , connect with me , any of my social links below
