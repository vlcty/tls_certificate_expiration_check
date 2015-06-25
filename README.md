# TLS Certificate Expiration Check for Nagios/Icinga

## What is it

This is a plugin for Nagios / Icinga to monitor the days left before a TLS certificate expires.
An expired certificate isn't good and should never happen. Well, it happened to me and I got many E-Mails from people complaining about that.

## What it does

The plugin uses OpenSSL's s_client to connect to a given hostname and port. Afterwards the expiration date is extracted and the delta time in days calculated.

You can give custom parameters for warning- and critical-values. Defaul warning-value is 30 days and default critical-value is 10 days.

## How to install
### Step 1: Get the script

Fetch the script and place it in your PluginDir-Folder. Usually it's unter `/usr/lib/nagios/plugins`

### Step 2: Install dependencies

I assume you have `perl` and `openssl` already installed. The only additional dependency is the `Date::Calc` perl module. It was already installed on my Ubuntu 15.04 system but not ony my Debian 8 servers.

If you use `apt` you can simply install it via

```apt-get install libdate-calc-perl```

Or you install it over CPAN. Your choice!

### Step 3: Make it available for Icinga 2

I don't have Nagios so I don't know how to configure it :-)

I tried to developed the plugin according to the "Nagios Plugin Development Guidelines" so there shouldn't be any problem with implementing this check in Nagios. You just have to figure out how.

To make it available to Icinga 2 you have to do the following steps.

#### Step 3.1: Create a CheckCommand object

Navigate on your Icinga 2 server to your config folder (Normally `/etc/icinga2/conf.d`).

I placed the following piece of code in `commands.cfg`:

```
object CheckCommand "tls_certificate_expiration_check" {
    import "plugin-check-command"

    command = [ PluginDir + "/check_tls_certificate_expiration" ]
    
    arguments = {
        "--hostname" = {
            required = true
            value = "$tls_hostname$"
        }
        "--servername" = {
            required = true
            value = "$tls_servername$"
        }
        "--port" = "$tls_port$"
        "--warn" = "$tls_warn$"
        "--crit" = "$tls_crit$"
    }

    vars.tls_hostname = "$address$"
}

```

#### Step 3.2: Add check to a host

To add the check to a host use this snippet in a host configuration file:

```
object Host "mineralwasser" {
    [...]
}

object Service "tls_blog.veloc1ty.de" {
        host_name = "mineralwasser"
        display_name = "TLS expire blog.veloc1ty.de"
        check_interval = 1d

        check_command = "tls_certificate_expiration_check"

        vars.tls_servername = "blog.veloc1ty.de"
}

```

Possible vars are:

* tls_hostname = The hostname of the server (mandatory, default: hosts address)
* tls_servername = The FQDN for the server name indication (SNI, mandatory)
* tls_port = The port of the server. Default: 443
* tls_warn = Warning limit in days before expiry date. Default: 30
* tls_crit = Critical limit in days before expiry date. Default: 10

Of course you can change check_interval to the value you desire. Since the plugin calculates in whole days one check every day is enough.

#### Step 3.3: Add a service group (optional)

If you want to add a service group for this check you can simply add the following snippet in `groups.conf`:

```
object ServiceGroup "tls_certificate_expiration" {
    display_name = "TLS certificate expiration checks"

    assign where service.check_command == "tls_certificate_expiration_check"
}
```

If you want to add it to multiple hosts work with `apply`!

That's it. Pretty simple. Reload or restart icinga2 and check the result in your browser.

# Special thanks

Special thanks to Jan from biocrafting.net for notifying me about the SNI issue!