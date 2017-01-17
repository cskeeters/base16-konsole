# Base16 Konsole

This is a [base16 template repository][tr] for konsole.

Currently it includes two templates and output folders.

## kde4-konsole-vim

`kde4-konsole-vim`.  This is designed to work with 

"Intense" colors (8-15) have been repurposed (similarly to [iterm2/dark.itermcolors.erb][itermtempl]) to work with [base16-vim][bv] with `base16colorspace` unset.  Other applications like prompts and `ls` **will look off** as a consequense.  I.e normally [ansi 9 looks red][wikicolors], but instead it's mapped to *base03* (grey) for base16-vim.

## kde4-konsole

`kde4-konsole` will *not work* with [base16-vim][bv], but colors will show up correctly in prompts and the colored output of `ls`.

# Installation

To install system-wide, run the following commands:

```bash
git clone https://github.com/cskeeters/base16-konsole
cd base16-konsole/kde4-konsole-vim
cp *.colorspace  /usr/share/kde4/apps/konsole/
```

To install for only your user, run:

```bash
git clone https://github.com/cskeeters/base16-konsole
cd base16-konsole/kde4-konsole-vim
cp *.colorspace  ~/.kde4/apps/konsole/
```

Then select the colorscheme you would like to use in konsole through the menus.

## Vim Configuration

Follow the instructions for [installation of base16-vim][bvi].  **Do not set base16colorspace!**

### Airline

In RHEL6, vim shows my airline status with bright yellow forground colors instead of darker colors.  It's using `ctermfg=10`, but 10 seems to be set correctly.  I'm not sure why, but changing term=bold to term=NONE fixes the issue.  The easiest way to do this is to remove `bold` from the string value on [this line][airline-bold].

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

[wikicolors]: https://en.wikipedia.org/wiki/ANSI_escape_code#Color://en.wikipedia.org/wiki/ANSI_escape_code#Colors 
[testcolor]: https://chriskempson.github.io/base16/
[itermtempl]: https://github.com/chriskempson/base16-builder/blob/master/templates/iterm2/dark.itermcolors.erb
[bv]: https://github.com/chriskempson/base16-vim
[tr]: https://github.com/chriskempson/base16#template-repositories
[bvi]: https://github.com/chriskempson/base16-vim#installation
[airline-bold]: https://github.com/vim-airline/vim-airline/blob/45d77ca90953e191e4ac140b964683c2aecef069/autoload/airline/themes.vim#L51
[konbug]: https://bugs.kde.org/show_bug.cgi?id=344181
[osc]: http://invisible-island.net/xterm/ctlseqs/ctlseqs.html#h2-Operating-System-Commands
