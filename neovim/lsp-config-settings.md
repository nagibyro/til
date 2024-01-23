# LSP Config Settings
Using [lspconfig]() it's important to understand how the options you pass to
the `.setup()` functions are actually passed to the underlying LSP. One thing
that tripped me up in the following config was the difference between
`init_options` and `settings` and also the difference between the lspconfig
plugins `settings` and the neovim clients `settings`

Lets look at the following example using the lspconfig plugin:

```lua
lspconfig.jedi_language_server.setup({
  init_options = {
    workspace = {
      extraPaths = {
          "./beta/vendors"
      },
    }
  }
})
lspconfig.pyright.setup({
    settings = {
        python = {
            analysis = {
                extraPaths = {
                    "./beta/vendor"
                }
            }
        }
    },
})
```

In the above example we are configuring two different python LSP's but why do
we need to set the `extraPaths` option in `init_options` for jedi but we can
set it in the `settings` option for pyright? This comes down to implementation
details of the underlying LSP Server. If you read the lspconfig documentation]() you'll find 

> The `settings` table is sent in `on_init` via a
  `workspace/didChangeConfiguration` notification from the Nvim client to
  the language server. These settings allow a user to change optional runtime
  settings of the language server. 

and

> {init_options} `table <string, string|table|bool>`
  a table passed during the initialization notification after launching
  a language server. Equivalent to the `initializationOptions` field found
  in `InitializeParams` in the LSP specification.
 
The LSP spec defines events that are passed between a client like your editor
and the LSP server or that the lsp server might send to the client(your editor). 
**initializationOptions** is passed as the LSP starts, the options are sent once.
Where as **workspace/didChangeConfiguration** can be sent at anytime from the client
to the lsp server and the server will update it's configuration. **IF** the server has
implemented the **workspace/didChangeConfiguration** event. As you get deeper into the
weeds you find that the LSP spec is large and language servers don't implement all of 
the LSP spec. Some of this is because that parts of the spec don't apply to the language
but others is simply that the work has not been done. In this case the Jedi language server
has not implemented the [**workspace/didChangeConfiguration** event](https://github.com/pappasam/jedi-language-server/blob/dff0f122f06e8ce3b5ade55f039dec951057edbd/jedi_language_server/server.py#L629).
It has it defined and it accepts the event but it does nothing with it (not even a log message). 
Whereas the pyright does implement this [event](https://github.com/microsoft/pyright/blob/cfb1de0cc4117095752d0c0c9ba1193402f971a6/packages/pyright-internal/src/languageServerBase.ts#L794)
This can lead to confusion but it is worth understanding who lsp client
interacts with the underlying server as well at the capabilities of the lsp
servers you are using.

A quick side note. If you are not using the lspconfig plugin and are trying to
use the lsp client direcly from the neovim api the `settings` value is used
slightly differently. From the lsp neovim docs:

> settings: Map with language server specific settings.
  These are returned to the language server if requested via
  `workspace/configuration`. Keys are case-sensitive.

The `workspace/configuration` event is an event the LSP Server send to the
client asking for configuration - a pull model in contrast to the `**workspace/didChangeConfiguration**`
event above which is pushed by the client to the server. But again you need to
check if your lsp server uses this. At the time of this writing I could not
find implementation detail of the jedi language server using this event. It
seems the only way to configure jedi is to use the `initializationOptions` when
the lsp starts.
