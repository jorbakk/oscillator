# oscillator

Vim / neovim plugin to access the system clipboard using OSC52 terminal escape
sequences.

Honoring OSC52 terminal sequences usually needs to be activated in your terminal
emulator, if it supports them. Other than that, no further configuration or
software is needed to have full access to your system clipboard, even if your
vim / neovim session runs on a remote server or in a virtual environment. The
clipboard text is transferred using stdin and stdout, so no additional
communication channel needs to be set up.

As off starting to write this plugin other solutions with a similar
functionality provided only yanking to the clipboard, but not pasting from it.
This plugin fills this gap and provides complete access to the system
clipboard. In neovim, this works fully transparent using the '+' and '*'
registers. However, neovim needs a small patch to directly communicate from the
plugin with the terminal emulator without any processing of stdin from the
event loop. The patch is rather small and adds to functions to start and stop
terminal input processing to the neovim API. It can be applied to e.g. neovim
release version 0.7, but should work with other versions, too. The patch is
supplied in this repository (nvim.patch). The neovim implementation is much
faster the vim one, though.

## Installation

Copy the plugin folder to an appropriate subdirectory in ~/.vim so that the
build in vim plugin manager can find it.

## Configuration

Sample vimrc entries to configure oscillator and enable transparent clipboard
access for neovim. Possible options:

    g:oscillator_silent = v:true
    " v:true  - be more verbose
    " v:false - surpress messages

    g:oscillator_yank_limit = 1000000
    " limit the size of text that can be yanked to the clipboard

    g:oscillator_base64decoder = 'base64 -d'
    " use this shell command to decode the clipboard text to base64. Default is a
    " VimL implementation of the base64 algorithm. This applies only to vim.
    " For neovim a lua implementation is used.

    g:oscillator_base64encoder = 'base64'
    " use this shell command to encode the clipboard text to base64. Default is a
    " VimL implementation of the base64 algorithm. This applies only to vim.
    " For neovim a lua implementation is used.

    g:oscillator_osc52_default_selection = 'clipboard'
    " type of clipboard that is used by the OscillatorYank and OscillatorPaste
    " ex-commands. If set to a different value, the 'primary' selection is used.
    " 'clipboard' and 'primary' are clipboard types defined by the OSC52
    " documentation, see also https://www.xfree86.org/current/ctlseqs.html. 

Mappings for vim / neovim:

    if has('nvim')
      let g:clipboard = {
        \ 'name': 'oscillator',
        \ 'copy': {
        \     '+': {lines, regtype -> OscillatorWriteRegToClipboard(lines, regtype, 'clipboard')},
        \     '*': {lines, regtype -> OscillatorWriteRegToClipboard(lines, regtype, 'primary')},
        \     },
        \ 'paste': {
        \     '+': {-> OscillatorReadRegFromClipboard('clipboard')},
        \     '*': {-> OscillatorReadRegFromClipboard('primary')},
        \     },
        \ }
    else
      vnoremap <leader>y :OscillatorYank<CR>
      nnoremap <leader>p :OscillatorPaste<CR>
    endif

## Related projects

- Yanking to the clipboard using OSC52 terminal sequences: [OSCYank](https://github.com/ojroques/vim-oscyank)
- Base64 encoding / decoding in VimL: [Vital](https://github.com/vim-jp/vital.vim)
