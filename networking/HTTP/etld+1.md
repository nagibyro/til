# ETld + 1
When working with mulitenant systems for things like cross site cookies or providing a service for [safe redirects](https://cheatsheetseries.owasp.org/cheatsheets/Unvalidated_Redirects_and_Forwards_Cheat_Sheet.html).
You find yourself acting on user input for redirects or cookies and wanting to make settings
based on user info and site they are interacting with. For example QA users have login cookies set for
`qa.myapp.com` but not `uat.myapp.com` set from a system hosted at `login.myapp.com`
(this can be common situation if using something like [Auth0](https://auth0.com/)).

What you find yourself wanting to do is parse urls `etld+1`. `etld` stands for *effective TLDs*, basic etld's are the same
as regular tld's like `.com` or `.org`. However there are many real world examples where tlds are not enough for decisions
for setting cookies or verifying domains for url redirects. Think of `github.io` or `.co.uk`. Illustrated example of etlds:

#### Simple ETLD
![simple-etld](./same-site.png)

#### Complex ETLD
![complex-etld](./eTLD+1.png)
###### images lifted from https://web.dev/same-site-same-origin/

etld's are maintained at [https://publicsuffix.org/](https://publicsuffix.org/) a good js library for parsing [tldts](https://github.com/remusao/tldts)

Also was mentioned in [Humans can't read URLs. How can we fix it? - HTTP 203](https://youtu.be/0-wB1VY3Nrc) youtube