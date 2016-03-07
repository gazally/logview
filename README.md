<p><a href="http://www.gnu.org/licenses/gpl-3.0.txt"><img src="https://img.shields.io/badge/license-GPL_3-green.svg" alt="License GPL 3"/></a>
<a href="http://stable.melpa.org/#/logview"><img src="http://stable.melpa.org/packages/logview-badge.svg" alt="MELPA Stable" title=""/></a>
<a href="http://melpa.org/#/logview"><img src="http://melpa.org/packages/logview-badge.svg" alt="MELPA" title=""/></a></p>


# Logview mode

Logview major mode for Emacs provides syntax highlighting, filtering
and other features for various log files.  The main target are files
similar to ones generated by Log4j, Logback and other Java logging
libraries, but there is really nothing Java-specific in the mode and
it should work just fine with any log that follows similar structure,
probably after some configuration.

The mode is meant to be operated in read-only buffer, so all the
command bindings lack modifiers.

Out-of-the-box the mode should be able to parse standard SLF4J (Log4j,
Logback) files as long as they use ISO 8601 timestamps and certain
UNIX files in `/var/log`.


### Installation

Logview is available from [MELPA](http://melpa.org/#/logview).
Assuming your `package-archives` lists MELPA, just type

    M-x package-install RET logview RET

to install it.

Alternatively, installing Logview from source is not difficult either.
First, clone the source code:

    $ cd SOME-PATH
    $ git clone https://github.com/doublep/logview.git

Now, from Emacs execute:

    M-x package-install-file RET SOME-PATH/logview

Alternatively to the second step, add this to your `.emacs` file:

    (add-to-list 'load-path "~/logview")
    (require 'logview)


### Submodes

Since there is no standard log file format, Logview mode has to try
and guess how the log file it operates on is formatted.  It does so by
trying to parse the very first line of the file against various
submodes it has.

If it succeeds in guessing, you will see major mode specifed as
‘Logview/...’ in the modeline, where the second part is the submode
name.

In case it fails, you will see it complain in the echo area.  The
buffer will also not be highlighted or switched to read-only mode, so
you will be able to edit it.

#### What to do if Logview mode fails to guess format

Currently your only option is to customize the mode.  `C-c C-s` will
show you only those options that are relevant to submode guessing.
You will need to customize at least one of those, or maybe all three.
All the variables are well-documented in customization interface.

If you think your log format is standard enough, you can open an issue
and request format addition to the list of mode built-ins.


### Commands

Nearly all commands have some use for prefix argument.  It can be
usually just guessed, but you can always check individual command
documentation within Emacs.

When buffer is switched to read-write mode, Logview automatically
deactivates all its commands so as to not interfere with editing.
Once you switch the buffer back to read-only mode, commands will be
active again.

#### Movement

* All standard Emacs commands
* Move to the beginning of entry’s message: `TAB`
* Move to next / previous entry: `n` / `p`
* Move to next / previous ‘as important’ [*] entry: `N` / `P` 
* Move to first / last entry: `<` / `>`

[*] ‘As important’ means entries with the same or higher level.  For
    example, if the current entry is a warning, ‘as important’ include
    errors and warnings.

#### Narrowing and widening

* Narrow from / up to the current entry: `[` / `]`
* Widen: `w`
* Widen upwards / downwards only: `{` / `}`

#### Filtering by entry level

* Show only errors: `l 1` or `l e`
* Show errors and warnings: `l 2` or `l w`
* Show errors, warnings and information: `l 3` or `l i`
* Show all levels except trace: `l 4` or `l d`
* Show entries of all levels: `l 5` or `l t`
* Show entries ‘as important’ as the current one: `+` or `l +`

#### Filtering by entry’s logger name, thread or message

See [more detailed description below](#filters-explained).

* Edit current name, thread and message filters: `f` (pops up a separate buffer)
* Add name include / exclude filter: `a` / `A`
* Add thread include / exclude filter: `t` / `T`
* Add message include / exclude filter: `m` / `M`

#### Resetting filters

* Reset level filter: `r l`
* Reset name filters: `r a`
* Reset thread filters: `r t`
* Reset message filters: `r m`
* Reset all filters: `R`
* Reset all filters, widen and show all explicitly hidden entries: `r e`

#### Explicitly hide or show individual entries

* Hide one entry: `h`
* Hide entries in the region: `H`
* Show some explicitly hidden entries: `s`
* Show explicitly hidden entries in the region: `S`

In Transient Mark mode `h` and `s` operate on region when mark is
active.

#### Explicitly hide or show details of individual entries

The mode terms all message lines after the first ‘details’.
Oftentimes these contain exception stacktrace, but most logging
libraries let you write anything here.

* Toggle details of the current entry: `d`
* Toggle details of all entries in the region: `D`
* Toggle details in the whole buffer: `e`

In Transient Mark mode `d` operates on region when mark is active.

#### Change options for the current buffer

These options can be customized globally and additionally temporarily
changed in each individual buffer.

* Toggle Auto-Revert mode: `o r`
* Toggle Auto-Revert Tail mode: `o t`
* Toggle ‘copy only visible text’: `o v`
* Toggle ‘search only in messages’: `o m`
* Toggle ‘show ellipses’: `o e`

#### Miscellaneous

* Customize options that affect submode selection: `o S` or `C-c C-s`
* Bury buffer: `q`
* Refresh the buffer (appending, if possible) preserving active filters: `g`
* Append log file tail to the buffer: `x`
* Revert the buffer preserving active filters: `X`
* Universal prefix commands are bound without modifiers: `u`, `-`, `0`..`9`


### Filters<a id="filters-explained"></a>

In addition to level filtering, Logview supports filtering by entry’s
logger name, thread and message.

These filters are regular expression and come in two kinds: _include_
and _exclude_ filters.  If at least one include filter is set, only
those entries where relevant part matches at least one of regular
expressions are shown, all others are filtered out.  If any exclude
filter is set, all entries where relevant part matches its regular
expression are filtered out (regardless of any other filters) and
hidden.

Easiest way to add filters is by using `a` / `A` commands (add
include/exclude name filter correspondingly), `t` / `T` (thread
filters), and `m` / `M` (message filters).  You can reset all filters
of given type: `r a` for name, `r t` for thread and `r m` for message
filters, `R` for all filters at once.

However, oftentimes you need to adjust existing filters, e.g. to fix a
typo or simply change one, while keeping others the same.  For this
purpose use `f` command.  It pops up a separate buffer with all
currently active filters, which you can edit as you like, using any
standard Emacs features.

Lines beginning with ‘#’ character are comments.  They and empty lines
are ignored.  Lines for actual filters must start with certain prefix:
‘a+ ’ (note the single trailing space!) for name inclusion filters,
‘a- ’ for name inclusion, and similart ‘t+ ’, ‘t- ’, ‘m+ ’, ‘m- ’ for
thread and message filters.  Additionally, multiline message filters
must use ‘.. ’ prefix (two dots and a space) for continuation lines.
The buffer mode has some syntax highlighting support, so you should
see if anything goes wrong.  The easiest way to figure it out is to
add a few filters using commands described earlier and then open this
buffer with `f` and see how they are represented.

#### Filter regexp details

Regular expressions can be matched against entry parts either
case-sensitively or case-insensitively, depending on standard Emacs
variable `case-fold-search`.

Filters are matched against relevant entry parts as strings, not
against the whole buffer.  Therefore, you can use `^` and `$` special
characters for the expected meaning.  For example, adding `^org` as
name exclusion filter will hide all entries where logger name _begins_
with string ‘org’.

Unlike name and thread filters, message filters can span multiple
lines.  To enter linefeed in message buffer (after `m` or `M`) use
`C-q C-j`.  When editing a multiline filter with `f`, prefix all
continuation lines with ‘.. ’.

Commands `a`, `A`, `t` and `T` default to the name (or thread) of the
current entry.  You can also use `C-p` (`<up>`) to browse history of
previously entered values and `C-n` (`<down>`) for a few default
values.
