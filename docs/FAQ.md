
# Frequently Asked Questions

## Does matterhorn support Gitlab authentication?

Matterhorn supports GitLab authentication indirectly. In order to use
Matterhorn with GitLab authentication, see the Authentication section on
Matterhorn details for use of session tokens.

## Does matterhorn support Mattermost Personal Access Tokens?

Yes. See the Authentication section above.

## How can I get Matterhorn to render emphasized Markdown text with an italic font?

In `~/.config/matterhorn/theme.ini`,
```
[other]
markdownEmph.style = [italic]
```
and in `~/.config/matterhorn/config.ini`,
```
themeCustomizationFile: theme.ini
```

This is known to work on `gnome-terminal` version `3.32.2`, VTE version
`0.56.3`; it may work for you, too. Many terminal emulators do not
support italics at all or without various hacks. Let us know what works
for you!

## I enabled italicized text in my theme configuration. Why doesn't it work?

Most terminfo files for typical terminal configurations do not provide
support for italicized text. If your terminal emulator supports italics,
you must enable it in your terminfo database in order to use it in
Matterhorn. For more information, see these links:

* http://www.nerdyweekly.com/posts/enable-italic-text-vim-tmux-gnome-terminal/
* https://medium.com/@dubistkomisch/how-to-actually-get-italics-and-true-colour-to-work-in-iterm-tmux-vim-9ebe55ebc2be
* https://github.com/tmux/tmux/blob/2.1/FAQ#L355-L383

## I am seeing malformed characters or display corruption when I run matterhorn in my terminal. What could be causing this?

Some terminal emulators cannot handle the extra escaping that occurs
when the URL hyperlinking mode is enabled. Try setting `hyperlinkUrls =
False` in your `config.ini` file.

## Does Matterhorn support graphical emoji?

At present Matterhorn does not reliably support graphical emoji due to
the lack of consistent support for wide Unicode characters in various
terminal emulators. Results may vary, and use of emoji characters may
cause terminal rendering issues depending on the terminal emulator in
use.

## I'm running Matterhorn in Tmux. How can I paste tmux buffers into Matterhorn's editor?

By default, `tmux`'s `paste-buffer` binding, `prefix-]`, pastes buffer
text by replaying it as terminal input. This will cause unwanted
behavior when Matterhorn receives that text and sends out each input
line as a separate message. But `tmux` supports bracketed paste mode
to make a `tmux` paste a block of text as *one* Matterhorn message.
It can be enabled by changing the behavior of `prefix-]` in the Tmux
configuration:

```
unbind-key -T prefix ]
bind-key -T prefix ] paste-buffer -p
```

## I'm using Mattermost through an Nginx proxy and I keep getting disconnected.

You might need to adjust your Nginx proxy settings.
For context, see [this potentially related report](https://github.com/matterhorn-chat/matterhorn/issues/578).

## Why doesn't Matterhorn support my desired keybindings?

In many terminal emulators, some key and modifier combinations do not
work at all or do not behave as expected. In these cases it will be
necessary to configure Matterhorn to intepret the appropriate escape
sequences in order to translate them into the desired key and modifier
combinations.

To configure Matterhorn to support a particular key combination, some
steps must be followed. Here are the steps, using `Control-Backspace` as
the example key combination (which is commonly not supported):

1) Determine the escape sequence that your terminal sends for the
   desired key combination. This can usually be done by consulting
   the terminal settings or by running `cat >/dev/null`, pressing the
   desired key combination, and observing the escape sequence that is
   printed. For example, for `Control-Backspace` on my terminal I get:
   ```
   $ cat >/dev/null
   ^[[3;5~
   ```
   This indicates that the sequence is `\ESC[3;5~`.
2) Create a file `~/.vty/config` and add to it the line `map _
   "\ESC[3;5~" KBS [MCtrl]`. This configuration file is loaded by
   the `vty` library, the library that Matterhorn uses to translate
   terminal input into key events. This configuration line tells
   `vty` to translate the input sequence `\ESC[3;5~` to the `KBS`
   (backspace) key with the modifier list `[MCtrl]` (control) regardless
   of the value of `TERM` (underscore). The list of valid keys and
   modifiers usable in `map` lines can be found [in the vty Haskell
   documentation](http://hackage.haskell.org/package/vty-5.28.2/docs/Graphics-Vty-Input.html).
3) Set up the desired event in `~/.config/matterhorn/config`'s
   `[keybindings]` section to be triggered by `C-Backspace`.

Another way to get the input byte sequence associated with your
keybinding is to run `matterhorn` with Vty debug logging enabled:

```
VTY_DEBUG_LOG=/tmp/matterhorn-keys-debug matterhorn
```

While `matterhorn` is running, the Vty debug log will contain output
about each received key combination like:

```
input bytes: "\b"
```

that will indicate what should go into the `map` entries in
`~/.vty/config`.

## Will you add support for my favorite GNU `readline` key to the message editor?

Please open a ticket to discuss support for any editing features that
you would like to see added. But please bear in mind that while we do
want Matterhorn's built-in text editor to provide a reasonable feature
set for editing messages, we do not intend to support all features
supported by other applications, and Matterhorn's editor is *not*
intended to provide the full feature set provided by `readline` even
though some of its features and behaviors are informed by `readline`.
This is also why we provide the ability to invoke an application of your
choosing to edit messages with `M-k` (`invoke-editor`).
