# Sway Config
When using [sway](https://wiki.archlinux.org/title/Sway) as your desktop window
manager a configuration file is provided in `/etc/sway/config`.

Many tutorials mention placing settings in `~/.config/sway/config`. Which I did thinking they would be
merged with the global configuration in `/etc/sway/config`. However this config file
is **not** a user override of global configuartion! It is a complete replacement of the config.

Launching sway with an empty config or incomplete config results in a grey
screen of death on login where you can't launch apps or interact with the
window manager in anyway. Trying to figure out what the issue is with logs is 
non-obvious because technically nothing went wrong. Thats just what sway looks like without anything 
configured. TIL.

A close inspection of the [configuration](https://wiki.archlinux.org/title/Sway#Configuration)
section of the arch wiki mentions copying the *example* config. However
I didn't read carfully and I was used to home files modifying or extending `/etc`
configuration not replacing them.

