# neorex

A Rex-Native Neovim frontend built in Zig.

One Neovim runs in the background behind all your panes, so buffers,
registers, undo and language servers are shared. `:split` and `:vsplit` open
real Rex panes where Neovim would put the window, and closing a pane in Rex
closes its window. Floats, the completion menu and statuslines are drawn
inside the pane they belong to.

## Status

Very early and rough, bugs / missing features are to be expected.

### Setup-compatibility fork

This branch adds Neovim tab pages as separate Rex tabs, keeps source windows
in their original tab, and avoids the Neovim 0.12.6 abort triggered when a
plugin queries the previous tab's current window. Closing a standalone
Diffview tab returns to the source editor.

Directional navigation goes through Rex's pane geometry. Normal-mode
Ctrl-H/J/K/L and Ctrl-arrows request pane focus; Insert, command-line and
terminal-mode Ctrl-arrows become ordinary arrows. The companion dotfiles
change forwards Rex navigation keys to neorex so it can make this choice.
External editor panes report a regular window type to Lua plugins, allowing
ToggleTerm to open a horizontal terminal from them.

Tested on macOS with Rex 0.1.0 (1728), Neovim 0.12.6, Zig 0.17.0 and the
existing LazyVim configuration. Live checks covered standalone Diffview
open/switch/close, the horizontal project terminal, floating ToggleTerm
input, fzf-lua's picker and preview, the dotfiles navigation actions, Rex
resizing and pane closure. All 107 tests pass when `REX_SERVER` is supplied;
without it, the live Rex API test skips. The optional golden suite was not
run. Regression
checks were also run with deliberately broken tab, navigation, window-type
and split-placement behavior to confirm that they fail.

The custom `config.diffview_region` same-tab workflow remains incompatible.
Its window moves suppress the events neorex tracks; a live test ended the
editor. Use `:DiffviewOpen` for this trial. Region zoom and plugin-driven
window rearrangement need further work before this can replace daily Neovim.

AI has been used heavily in building neorex due to limits on my available time. A lot of the code needs large improvements, cleanup, better design etc. I will be doing this incrementally - any contributions are welcome.

## Requirements

- macOS (it uses kqueue)
- [Rex](https://www.superlogical.com/rex)
- A recent Neovim. It's developed against 0.13-dev; older versions may show
  no statuslines.
- Zig 0.17 to build

## Build and run

```sh
zig build
zig build test
zig build test -Dgolden   # also the golden tests; downloads Ghostty once
```

The golden tests check every frame neorex draws against
[Ghostty](https://github.com/ghostty-org/ghostty)'s terminal emulator, pinned
to one commit.

Run it in a Rex pane, with any Neovim arguments:

```sh
zig-out/bin/neorex some/file.txt
```

`neorex help` lists the debug commands, which run each layer on its own:
keys, Neovim's redraw events, the renderer, and pane hand-off.

## Current limitations

- The `<C-w>` resize, swap and move commands do nothing. Use Rex's own pane
  controls instead.
- Plugins that arrange windows (sidebars, layout managers) don't get the
  placement they expect.
- Inactive tabs retain their windows, but their temporary native split tree
  does not preserve the Rex layout. Plugins inspecting that tree can see
  different geometry.
- No external command line or messages: long messages still show
  "Press ENTER".
- Mouse clicks on a float go to the window under it.

## Plans

Much of neorex will be split out into standalone Zig modules, and later into
their own packages:

- a Neovim RPC client (msgpack, message framing, requests)
- a Rex API client (WebSocket and the control protocol)
- terminal key input (the kitty keyboard protocol)

neorex should then be easier to extend to other editors that provide similar processes for running external windows.

As rex matures, and there is increased support for things like floating windows, notifications etc we can then start to move more and more things over to rex to display natively.

## License

MIT
