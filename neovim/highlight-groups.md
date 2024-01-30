# Highlight Groups
I was having an issue in neovim, when my cursor went onto a parenthesis or
bracket it would seem to disappear and jump to the corresponding
closing paren. However the cursor had not really jumped and if I continued off
the paren my cursor would seem to reappear. This was incredibly disorienting to
my workflow. I came to find out that this matching paren highlighting was
builtin feature for vim/neovim based on the [Pi_paren](https://neovim.io/doc/user/pi_paren.html)
standard plugin (Also referred to as matchparen.vim). With this plugin when you go over `{}` or `()`
it'll highlight them. 

**NOTE**: I actually am using [sentiment.nvim](https://github.com/utilyre/sentiment.nvim)
which is a dropin replacement for the builitn Matchparen plugin

This is where I learned about Highlight Groups in Vim. Basically a highlight
group represents a set of things that should be highlighted together. For the
matchparen plugin it had a corresponding `MatchParen` highlight group which
represents the two parens that the matchparen plugin has found that your cursor
is in. There are many [builtin highlight groups](https://neovim.io/doc/user/syntax.html#highlight-groups)
but they can also be created by plugins and be defined in a namespace which is
represented by an id number (the builtin highlights are in a global namespace
with id 0).

These highligh groups like most things in Vim/Neovim are configurable. You can
see their configuration by using the `hi {highlight-group}` command (or just
`hi` to get all groups printed out) for Example the Matchparen highlight group
looked like this:

```
:hi MatchParen
MatchParen     xxx cterm=bold gui=bold guifg=#f44336 guibg=Grey
```

These configs control how highlighting is displayed for that group and
generally has a config for nvim in a gui app or in the terminal. Read more
about highlight-args with [`:h highligh-args`](https://neovim.io/doc/user/syntax.html#highlight-args)

What I learned was my Matchparen was using the `gui=reverse` option which with
my dark colorscheme made the cursor straight up disappear. By changing
foreground and background colors I was able to make my cursor not appear to
jump. 

I was able to modify the `MatchParen` highlight group in my config via the
neovim lua api. I changed it to the above example by adding

```lua
vim.api.nvim_set_hl(0, 'MatchParen', {
  bold=true,
  fg="#f44336",
  bg="Grey",
  cterm="bold"
})

```

in my config. I had never dived deep into the highlighting system in neovim but now it feels
much more approachable understanding how the system works.

If you're interested in seeing all the highlights you have defined and what
they look like rendered you can run `:so $VIMRUNTIME/syntax/hitest.vim` From
there you can look through and invesitage any highlighting you might want to
customize.
