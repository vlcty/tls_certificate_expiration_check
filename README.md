# TLS Certificate Expiration Check for Nagios/Icinga

## What is it

This is a small plugin for Nagios / Icinga to monitor the days left before a TLS certificate expires.
An expired certificate is not good and should never happen. Well, it happened to me and I got dozens of E-Mail about people complaining about that.

## What it does

The plugin uses OpenSSL's s_client to connect to a given hostname and port. Afterwards the date of the expiration is extracted and the delta time calculated.

You can give custom parameters for warn- and critical-values. Defaul warn-value is 30 days and default critical-value is 10 days.

## How to install
### Step 1: Get the script

Fetch the script and place it in your PluginDir-Folder. Usually it's unter `/usr/lib/nagios/plugins`

### Step 2: Install dependencies

I assume you have `perl` and `openssl` already installed. Only the perl module `Date::Calc` is required but maybe already installed.

On Debian based distributions you can install it via

```apt-get install libdate-calc-perl```

Or you install it over CPAN.

### Step 3: Make it available for Icinga 2

I don't have Nagios so I don't know how to configure it :-)

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
                "--port" = "$tls_port$"
                "--warn" = "$tls_warn$"
                "--crit" = "$tls_crit$"
        }

}
```

#### Step 3.2: Add check to a host

To add the check to a single host use the following in a host configuration file:

```
object Host "mineralwasser" {
    [...]
}

object Service "tls_blog.veloc1ty.de" {
        host_name = "mineralwasser"
        display_name = "TLS expire blog.veloc1ty.de"
        check_interval = 1d

        check_command = "tls_certificate_expiration_check"

        vars.tls_hostname = "blog.veloc1ty.de"
}

```

Possible vars are:

* tls_hostname = The hostname of the server (mandatory)
* tls_port = The port of the server. Default: 443
* tls_warn = Warning limit in days before expiry date. Default: 30
* tls_crit = Warning limit in days before expiry date. Default: 10

That's it. Pretty simple and small.

If you want to add it to multiple hosts work with `apply`

