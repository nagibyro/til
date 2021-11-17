# Sway Config
When using [sway](https://wiki.archlinux.org/title/Sway) as your desktop window
manager; the configuration file provided in `/etc/sway/config` is meant as an
example file. Following examples for setting up various config in sway many examples
mention setting config values in `~/.config/sway/config`.
However this config file is **not** a user override of the global configuartion!
It does not merge the values instead it is a complete replacement of the config.

A close inspection of the [configuration](https://wiki.archlinux.org/title/Sway#Configuration)
section of the arch wiki mentions copying the *example* config. I was used to
home files modifying or extending `/etc` configuration not being a replacement.

Launching sway with an empty config or incomplete config results in a grey
screen of death. Logs are also non-obvious because technically nothing went
wrong, thats just what sway looks like without anything configured. TIL.

