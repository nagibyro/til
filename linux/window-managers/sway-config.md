# Sway Config
When using [sway](https://wiki.archlinux.org/title/Sway) as your desktop window
manager; the configuration file provided in `/etc/sway/config` is meant as an
example file. While following config examples mentions setting config values in
`~/.config/sway/config`. However this config file is **not** a user override of
the global configuartion! It does not merge the values instead it is a complete
replacement of config.

A close inspection of the [configuration](https://wiki.archlinux.org/title/Sway#Configuration)
section of the arch wiki mentions copying the *example* config. I was used to
home files usually modifying or overriding `/etc` files not replacing them.

Launching sway with an empty config or a one line config results in a grey
screen of death. Logs are also non-obvious because technically nothing went
wrong, thats just what sway looks like without anything configured.

