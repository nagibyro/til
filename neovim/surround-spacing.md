# Surround Spacing
When using the nvim-surrond plugin, the plugin has different behavior depending
on if you change a surrounding braces, paraens, or brackets using the left
opening surround or the closing right surrond key.

If you use the left key it will (in some cases depending on filetype) add an
extra whitespace between the surround pair and word you are surrounding.
However if use the right or end key it will not add this whitespace.

## Example
If you run with cursor on the word `foo`

- `ysiw[` -> `[ foo ]`
- `ysiw]` -> `[foo]`

This behavior is there for compatability with the `vim-surround` plugin. It's
possible to disable this [entirely](https://github.com/kylechui/nvim-surround/issues/264)
but if you are still learning the keys like me then it's probably better to
just learn the semantics. While this behavior doesn't feel obvious it is
documented in `:h nvim-surround.default_pairs`
