# Naxsi web application firewall

This mod adds a web application firewall based on [naxsi](https://github.com/nbs-system/naxsi) to the docker-swag container.


# Installation instructions
In the container's docker arguments, set an environment variable `DOCKER_MODS=XXXXX`

If adding multiple mods, enter them in an array separated by `|`, such as `DOCKER_MODS=XXXXX|linuxserver/mods:swag-mod2`


# Configuration of naxsi
In order to use naxsi in your reverse proxy you need to perform the following basic installation steps. This will allow you to use some predefined rules which match most known attack vendors. Additionally it is possible to add new rules manually or with an integrated learning engine based on the nxtool.

## Add core rules to nginx.conf
You will need to add some core rules whith some basic blocking patterns for most known attacks. This rules will apply to all reverse proxy locations where you enable naxsi later. By default they will not affect any of your existing proxy configurations. Sample core rules based on the official documentation of naxsi are included in the image. To use this you will need to rename the file `/config/naxsi/rules/naxsi_core.rules.sample` to `/config/naxsi/rules/naxsi_core.rules`.

Now you need to tell nginx where this default rules are located. Therefore open the conf file `/config/nginx/nginx.conf` and add the line `include /config/naxsi/rules/naxsi_core.rules;` inside your http block. Do **not** add `/config/naxsi/rules/naxsi_core.rules.sample` as this file will be overwritten during each restart. The result should look like this:
```
...
http {
    ...

    ##
	# Naxsi Settings
	#
    # Include naxsi core rules. This will only mount them to the config, they are not active by default.
    # You will need to enable naxsi on a per location base inside your proxy server block to avoid breaking changes.
    ##

    include /config/naxsi/rules/naxsi_core.rules; 
    
    ...
}
...
```
If you have a custom configuration you will probably need to update this as well.

## Enable naxsi for a proxy location block
In order to use naxsi for your proxy files you will need to activate it in each location block where the WAF should listen or perform actions. You can use a predefined config file for the location blocks, cusomize the predefined one for yourself if needed or create an individual one for specific locations. This default file is located here `/config/naxsi/naxsi.conf`. The latest sample version is always located in the same folder. In case of any update inside the repository you will be informmed and we will update the sample file. The location block should look like this:
```
...
server {
    listen ...
    servername ...
    ...

    loction / {
        ...

        # Include naxsi default configuration. This will enable naxsi with all configs provided in the conf file
        include /config/naxsi/naxsi.conf;

        # Enable naxsi learning mode only for this location enable also this setting (Remove "#" in front of the next line)
        #LearningMode; 
        
        ...
        proxy_pass ...
    }
}
...
```

## Enable fail2ban integration
This mod ships a preconfigured fail2ban filter which will be copied in the fail2ban `filter.d` directory. In order to use this you will need to enable the fail2ban jail. Therefore open the file `/config/fail2ban/jail.local` and add the below content at the end of the document.
```
[nginx-naxsi]

enabled = true
port = http,https
filter = nginx-naxsi
logpath = /config/log/nginx/error.log
maxretry = 5
```


# Full documentation of naxsi
All steps described here are based on a research on the official wiki page. If you need to customize your naxsi further, please have a look at this wiki to find more information: https://github.com/nbs-system/naxsi/wiki

## Rules and Whitelisting documentation

If you like to create rules manually you can follow the following documentation guides.
- Explaining rules: https://github.com/nbs-system/naxsi/wiki/rules-bnf
- Rule examples: https://github.com/nbs-system/naxsi/wiki/rules-examples
- Explaining whitlisting: https://github.com/nbs-system/naxsi/wiki/whitelists-bnf
- Whitelisting examples: https://github.com/nbs-system/naxsi/wiki/whitelists-examples 