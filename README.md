# nvim-spider 🕷️🕸️
Use the `w`, `e`, `b` motions like a spider. Move by subwords and skip insignificant punctuation.

This is a fork chrisgrieser/nvim-spider with some changes and additions that were rejected by the author.

Lua implementation of CamelCaseMotion, with extra consideration of punctuation. Works in normal, visual, and operator-pending mode. Supports counts and dot-repeats.

> __Note__  
> If you installed the plugin before March 31, you should change your
> keymappings to call the motions via Ex-commands to make them dot-repeatable: `"<cmd>lua require('spider').motion("w")<CR>"`. [See the example here.](#installation)

<!--toc:start-->
- [Features](#features)
	- [Subword Motion](#subword-motion)
	- [Skipping Insignificant Punctuation](#skipping-insignificant-punctuation)
- [Installation](#installation)
- [Configuration](#configuration)
- [Notes on Operator-pending Mode](#notes-on-operator-pending-mode)
- [Subword Text Object](#subword-text-object)
- [Credits](#credits)
<!--toc:end-->

## Features
The `w`, `e`, `b` (and `ge`) motions work the same as the default ones by vim, except for two differences:

### Subword Motion
The movements happen by subwords, meaning it stops at the sub-parts of a CamelCase (or SCREAMING_SNAKE_CASE or kebab-case) variable.

```lua
-- positions vim's `w` will move to
local myVariableName = FOO_BAR_BAZ
--    ^              ^ ^

-- positions spider's `w` will move to
local myVariableName = FOO_BAR_BAZ
--    ^ ^       ^    ^ ^   ^   ^
```

### Skipping Insignificant Punctuation
A sequence of one or more punctuation characters is considered significant if it is surrounded by whitespace and does not include any non-punctuation characters.

```lua
foo == bar .. "baz"
--  ^      ^    significant punctuation

foo:find("a")
-- ^    ^  ^  insignificant punctuation
```

This speeds up the movement across the line by reducing the number of mostly unnecessary stops.

```lua
-- positions vim's `w` will move to
if foo:find("%d") and foo == bar then print("[foo] has" .. bar) end
-- ^  ^^   ^  ^^  ^   ^   ^  ^   ^    ^    ^  ^  ^ ^  ^ ^  ^  ^ ^  -> 21

-- positions spider's `w` will move to
if foo:find("%d") and foo == bar then print("[foo] has" .. bar) end
-- ^   ^      ^   ^   ^   ^  ^   ^    ^       ^    ^    ^  ^    ^  -> 14
```

If you prefer to use this plugin only for subword motion, you can disable this feature by setting `skipInsignificantPunctuation = false` in the `.setup()` call.

> __Note__  
> This plugin ignores vim's `iskeyword` option.

## Installation

```lua
-- packer
use { "chrisgrieser/nvim-spider" }

-- lazy.nvim
{ "chrisgrieser/nvim-spider", lazy = true },
```

No keybindings are created by default. Below are the mappings to replace the default `w`, `e`, and `b` motions with this plugin's version of them.

```lua
vim.keymap.set({"n", "o", "x"}, "w", "<cmd>lua require('spider').motion('w')<CR>", { desc = "Spider-w" })
vim.keymap.set({"n", "o", "x"}, "e", "<cmd>lua require('spider').motion('e')<CR>", { desc = "Spider-e" })
vim.keymap.set({"n", "o", "x"}, "b", "<cmd>lua require('spider').motion('b')<CR>", { desc = "Spider-b" })
vim.keymap.set({"n", "o", "x"}, "ge", "<cmd>lua require('spider').motion('ge')<CR>", { desc = "Spider-ge" })
```

> __Note__  
> For dot-repeat to work, you have to call the motions as Ex-commands. When calling `function() require("spider").motion("w") end` as third argument of the keymap, dot-repeatability <!-- vale Google.Will = NO -->will *not* work.

## Configuration
The `.setup()` call is optional. Currently, its only option is to disable the skipping of insignificant punctuation:

```lua
-- default value
require("spider").setup({
	skipInsignificantPunctuation = true
})
```

## Notes on Operator-pending Mode
<!-- vale Google.FirstPerson = NO -->
In operator pending mode, vim's `web` motions are actually a bit inconsistent. For instance, `cw` will change to the *end* of a word instead of the start of the next word, like `dw` does. This is probably done for convenience in vi's early days before there were text objects, but in my view taught people bad habits.

In this plugin, such small inconsistencies are deliberately not implemented. This is also because with of subword motions, vim's behavior would create unexpected results when used in subwords or near punctuation. If you absolutely want to, you can map `cw` to `ce` though. (Remember to add `remap = true` as option.)
<!-- vale Google.FirstPerson = YES -->

## Subword Text Object
This plugins supports `w`, `e`, and `b` in operater-pending mode, but does not include a subword-variant of the `iw`. For a version of `iw` that considers CamelCase, check out the `subword` text object of [nvim-various-textobjs](https://github.com/chrisgrieser/nvim-various-textobjs).
