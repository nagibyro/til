# Proxy Pass HTTPS and Static Config
I don't find myself configuring webservers too often anymore these days. However, I try to
keep enough base knowledge to be able to write a decent production ready reverse proxy either
for a nodejs api (for off loading ssl) or nginx + phpfpm setup. Side note since nginx config
can be a bit daunting to get started from scratch [Digital Ocean's nginx Config tool](https://www.digitalocean.com/community/tools/nginx)
is fantastic and gives a good base template to customize as needed. 

Today though I found myself needing to configure nginx as a reverse proxy for a whole website. So needing something like:

```
location / {
    proxy_pass https://www.foobar.com;
}
```

There were a couple of gotcha's that I had not expected / never had to deal with in my simpler reverse proxy setups. The
first being `HTTPS` and as far as I can find there is no clear guides for reverse proxying https sites. The first gotcha
was that by default nginx does not verify the cert of the proxied url. Not sure why -- maybe makes sense for microservice 
architectures doing self signed certs for end to end encryption? Regardless setting `proxy_ssl_verify on;` will make sure
you are validating downstream services which I feel should be the default as it is for tools like curl. However,
setting this option will also require you to tell nginx which certs to use for verification through the `proxy_ssl_trusted_certificate /path/to/certs.crt`
config option In my case I was proxying a public website so I can just set it to the public `ca-certificate.crt` bundle 
installed on whichever linux I'm deploying my app in. For debian flavors this is `/etc/ssl/certs/ca-certificates.crt`. 
The last ssl config that tripped me up was [SNI](https://www.cloudflare.com/learning/ssl/what-is-sni/) since the site I
was proxying had multiple certs for it's given IP. You have to specifically enable proxy SNI with `proxy_ssl_server_name on;`

While all of this was discoverable by reading some error messaging and googling it was non the less a bit of a PITA. I
feel like since nginx compiles against openssl it should be able to default to the system's cert bundle if none is explicitly
given and SNI being on by default makes sense these days.

The second set of gotcha's were around using a hostname url in the `proxy_pass` directive. Essentially if you use a domain name in the url
aka `https://google.com` instead of an ip `https://55.55.55.1` nginx resolves the domain name to an IP address only once
at application startup. This apparently has tripped people up all the way back in [2011](https://forum.nginx.org/read.php?2,215830,215832#msg-215832)
Which is not great since IP address for domains can change pretty regularly. Also on top of this since its resolved at nginx
startup or config reload it uses the host OS's dns resolver instead of any `resolver` directive you have in your config.
This can be bad for example if the host os resolves an IPV6 address instead of IPV4 and your network isn't configured to
work with IPV6. You're only option is  to fix it on the host. Which is impossible in cases where you're using  serverless
ECS or don't have access to the host.

However, there is a solution if you set the url to a nginx config variable, nginx will resolve dns for the domain name based on
the resolver you've configured in nginx (which bonus points you can turn of ipv6 if needed). Though you have to be careful
and make sure you append the `$request_uri` otherwise you're proxy path may not work (not sure about this point saw it in
a stackoverflow comment). The config would look something like:
```
location / {
     resolver 8.8.8.8 ipv6=off; #use google dns resolver since we're proxying a public site
     set $backend "foobar.com$request_uri";
     proxy_pass https://$backend;
}
```

While not a ton of config if you're not a nginx guru it can take some figuring out to know all the options that are relevant
to your use case and ends up being a bit more work than it may initially seem. so in conclusion to proxy a public https
website 

```
location / {
    proxy_pass https://www.foobar.com;
}
```

needs to actually turn into something more like

```
location / {
     resolver 8.8.8.8 ipv6=off; #use google dns resolver since we're proxying a public site
     set $backend "foobar.com$request_uri";
     proxy_pass https://$backend;

     proxy_ssl_verify                   on;
     proxy_ssl_trusted_certificate      /etc/ssl/certs/ca-certificates.crt;
     proxy_ssl_server_name              on;

     #And don't forget your proxy headers
     proxy_set_header Upgrade           $http_upgrade;
     proxy_set_header Connection        $connection_upgrade;
     proxy_set_header Host              $host;
     proxy_set_header X-Real-IP         $remote_addr;
     proxy_set_header Forwarded         $proxy_add_forwarded;
     proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
     proxy_set_header X-Forwarded-Proto $scheme;
     proxy_set_header X-Forwarded-Host  $host;
     proxy_set_header X-Forwarded-Port  $server_port;

     # and timeouts
     proxy_connect_timeout              60s;
     proxy_send_timeout                 60s;
     proxy_read_timeout                 60s;
}
```