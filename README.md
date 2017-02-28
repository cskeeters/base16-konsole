# Base16 Konsole

This is a [base16 template repository][tr] for konsole.

Currently it includes three templates and corresponding output folders.

`*.colorscheme` files are designed to work with the version of konsole included with KDE4 and KDE5.  `*.schema` files are designed to work with the version of konsole included with KDE3.

## colorscheme-vim

The files in `colorscheme-vim` are designed to be compatible with [base16-vim][bv].  The means the "intense" colors (8-15) have been repurposed (similarly to [iterm2/dark.itermcolors.erb][itermtempl]) to provide shades of grey and yellow colors needed for a good looking vim interface.  Normally [ansi 9 looks red][wikicolors], but instead it's mapped to *base03* (grey) for base16-vim.  To make the vim colorscheme compatible with this mapping, you must leave `base16colorspace` unset in `.vimrc`.  Other applications like prompts and `ls` **will look off** as a consequence - although this can be mitigated as described below.

## colorscheme

The files in `colorscheme` will *not* work with [base16-vim][bv], but colors will show up correctly in prompts and the colored output of `ls`.

## schema

The files in `schema` will work with [base16-vim][bv].  Only use these if you use KDE3 (RHEL5)

# Installation

To install system-wide, copy the desired files (not the folder) into the appropriate folder.  You can download and install only one file if you know what theme you want to use.

KDE Version | Available To |Location
------------|--------------|-----------------------
KDE 5       | System Wide  | `/usr/share/konsole`
KDE 5       | User Only    | `~/.local/share/konsole`
KDE 4       | System Wide  | `/usr/share/kde4/apps/konsole`
KDE 4       | User Only    | `~/.kde4/apps/konsole/`
KDE 3       | System Wide  | `/usr/share/apps/konsole/`
KDE 3       | User Only    | `~/.kde/apps/konsole/`

You must restart konsole, then you can select the theme you would like to use through the menus.

## Vim Configuration

Follow the instructions for [installation of base16-vim][bvi].  **Do not set base16colorspace!**

### Airline

In RHEL6, vim shows my airline status with bright yellow forground colors instead of darker colors.  It's using `ctermfg=10`, but 10 seems to be set correctly.  I'm not sure why, but changing term=bold to term=NONE fixes the issue.  The easiest way to do this is to remove `bold` from the string value on [this line][airline-bold].

### junegunn/fzf

If you use [fzf][fzf] and want colorization to look right, you'll need something like this in your `.bashrc`.

```sh
if [[ $TERM =~ konsole.* ]]; then
    export FZF_DEFAULT_OPTS='--color fg+:5,hl+:6'
fi
```

This is due to fzf using intense versions of colors.  Since most are dark, the selected item winds up being hard to see.  With `FZF_DEFAULT_OPTS` as mentioned, the selected item will be white and matching text will be orange.  Those are the only good colors to use.

### ls colorization

With kde4-konsole-vim, `ls --color` shows many items in a dark color that is hard to see.  To correct this, you need to set the `LS_COLORS` environment variable to not use bright intensity colors.  You can do this most easily with `dircolors`.  See the [dir_colors manpage][dir_colors] for additional details.  The process for removing intensity code (01) from `LS_COLORS` is:

1. Copy /etc/DIR_COLORS to ~/.dir_colors
1. Edit ~/.dir_colors and modify entries to remove `01;`.  For example, `DIR 01;34` should become, `DIR 34`.
1. Logout and log back in, or run `eval $(dircolors ~/.dir_colors)`

A [base16-vim compatible file][dcgist] is available for you to download if you don't need any further customization.

```sh
cd ~
curl -OL https://gist.githubusercontent.com/cskeeters/aacd10c075d3c7092a5e4e36db34e62d/raw/.dir_colors
```

### Mercurial Configuration

If you use the [color extension][hgc] for hg, you'll need to customize the colors so that bold is not used.  Add this to your `~/.hgrc`.

    [color]
    status.modified = blue
    status.added = green
    status.removed = red
    status.deleted = cyan
    status.unknown = magenta
    status.ignored = yellow

    diff.extended = cyan
    diff.file_a = red
    diff.file_b = green
    diff.trailingwhitespace = red_background

    qseries.applied = blue underline
    qseries.unapplied = yellow
    qseries.missing = red

### Advanced Configuration

If you are like me and want a single `.vimrc` to function for konsole (in 16 color mode) and another terminal which supports 256 colors, you can configure konsole's environment to set `TERM=konsole` and then only set base16colorspace when TERM does not start with konsole

```vim
if $TERM !~# "konsole.*"
    " As a work around for the following bugs in kde4's konsole:
    "   use the output of 16.colorscheme.rb and don't set base16colorspace.
    "   base-shell script will not be called
    " https://github.com/chriskempson/base16-shell/issues/31
    " https://bugs.kde.org/show_bug.cgi?id=344181
    let base16colorspace=256
endif
```

# Building

Most users won't need to rebuild this repository as the color files you need for konsole are generated and committed to this template repository.

This template repository can only be built with [cskeeters/base16-builder-php](https://github.com/cskeeters/base16-builder-php) with [this fixed width modification](https://github.com/cskeeters/base16-builder-php/commit/f13a88c9a460c9377b8b27d401996e4f41ebf748).  This enables KDE3's schema format to align properly.

# Technical Details

ANSI colors can be set in a terminal in two ways.

1. Using the menus to load some settings file, or through `.Xresources`.
2. Set by sending ANSI operating system commands to the terminal.

If

* bright colors (8-15) are set to the same or similar color with the standard ansi colors (0-7),
* Colors 16-20 are set using either method above to dark/orange colors.
* `let base16colorspace=256` is set in `.vimrc`

then everything works great.

Unfortunately, most terminal's only allow you to set colors 0-15.  Konsole is no exception.  Since konsole [doesn't support xterm's operating system command to set those colors][konbug] (ala base16-shell), there is no point in using it and base16-vim can not operate in a 256 colorspace.  Fortunately, base16-vim can function with base16colorspace not set if the black and orange characters are set in colors 8-15.  This template repository provides colorscheme files where colors 8-15 have the black/orange colors needed.

Fortunately, base16-vim is designed to be able to To use vim with vim-base16 in konsole, we must use this 16-color pallet version  and be sure to NOT set base16colorspace=256 in vimrc.


## Further reading/references on ANSI codes

For those interested, you can check out the spec [xterm's documentation on operating system commands][osc]:

```text
    OSC    Ps   ;    Pt                             ST
    ESC ]  4    ;    <ColorNumber> ; rgb:FF/00/00   ESC \
````

Under *Ps = 4*, you can read.

> This can be a name or RGB specification as per XParseColor.

[man 3 xparsecolor][xpc] shows under *Color Names* supporting the `rgb:<red>/<green>/<blue>` format.

[xpc]: https://linux.die.net/man/3/xparsecolor

base16-shell uses this corresponding template to set the color:

    printf_template='\033]4;%d;rgb:%s\033\\'

[wikicolors]: https://en.wikipedia.org/wiki/ANSI_escape_code#Colors
[testcolor]: https://chriskempson.github.io/base16/
[itermtempl]: https://github.com/chriskempson/base16-builder/blob/master/templates/iterm2/dark.itermcolors.erb
[bv]: https://github.com/chriskempson/base16-vim
[tr]: https://github.com/chriskempson/base16#template-repositories
[bvi]: https://github.com/chriskempson/base16-vim#installation
[airline-bold]: https://github.com/vim-airline/vim-airline/blob/45d77ca90953e191e4ac140b964683c2aecef069/autoload/airline/themes.vim#L51
[konbug]: https://bugs.kde.org/show_bug.cgi?id=344181
[osc]: http://invisible-island.net/xterm/ctlseqs/ctlseqs.html#h2-Operating-System-Commands
[fzf]: https://github.com/junegunn/fzf
[dir_colors]: https://linux.die.net/man/5/dir_color://linux.die.net/man/5/dir_colors
[dcgist]: https://gist.github.com/cskeeters/aacd10c075d3c7092a5e4e36db34e62d
[hgc]: https://www.mercurial-scm.org/wiki/ColorExtension
