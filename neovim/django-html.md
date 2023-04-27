# Django Html
I've have been working on a django project and ran into the situation where
I needed to be working on django templates. Which have the `.html` extension.
Neovim would set filetype for HTML files but working with django html files
are actually a superset of html and jinja templating (or something jinja
like not sure if it's actually considered jinja or not). 

Turns out vim / neovim already has a builtin highlighter for django templates.
Specifically `setfiletype htmldjango` and `setfiletype django` the latter does
highlighting for django templating language but not html whereas the former
does both html and django templating. With Vim being vim we'll need to do
some config to make sure the right filetype gets loaded.

## My imperfect solution
Because the html in python projects I'm working with is always django and when
I'm working in python I'm always in a virtualenv. I went
with a imperfect but works for me approach. Using `filetype.lua` you can use
the lua API to add extensions for filetypes but also can override existing
definitions and invoke a lua function to return the type.

```lua
-- add below to filetype.lua in your nvim config directory
vim.g.do_filetype_lua = 1

vim.filetype.add({
  extension = {
    html = function()
      if vim.env.VIRTUAL_ENV then
        return "htmldjango"
      else
        return "html"
      end
    end,
  },
})
```

While this works because of my specific work conditions. It could be
endlessly iterated on to be more generic perhaps looking through the filesystem
to try and identify a django project or whatever. For now this works
well for me and is simple addition to my project.
