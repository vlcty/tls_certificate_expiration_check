# TLS Certificate Expiration Check for Nagios/Icinga

## What is it

This is a plugin for Nagios / Icinga to monitor the days left before a TLS certificate expires.
An expired certificate isn't good and should never happen. Well, it happened to me and I got many E-Mails from people complaining about that.

## What it does

There are two modes:

1) Fetch a file via the network
2) Read a local file

If you give an address and a port option 1 is choosen.
If you set a file option 2 is choosen.

## How to install
### Step 1: Get the script

Fetch the script and place it in your PluginDir-Folder. Usually it's unter `/usr/lib/nagios/plugins`

### Step 2: Install dependencies

The following applications and modules are needed:

- perl of course
- openssl
- Perl module "Crypt::OpenSSL::X509"
- Perl module "Date::Calc"

If you are on a Debian/Ubuntu based machine you can install the needed perl modules via apt:

```apt-get install libdate-calc-perl libcrypt-openssl-x509-perl```

Another way is to fetch it over CPAN. Your choice!

### Step 3: Make it available for Icinga 2

I don't have Nagios so I don't know how to configure it :-) I tried to developed the plugin according to the "Nagios Plugin Development Guidelines" so there shouldn't be any problem implementing this check in a nagios compatible environment.

#### Step 3.1: Create a CheckCommand object

Navigate on your Icinga 2 server to your config folder (Normally `/etc/icinga2/conf.d`).

I placed the following piece of code in `commands.cfg`:

```
object CheckCommand "tls_certificate_expiration" {
    import "plugin-check-command"

    command = [ PluginDir + "/check_tls_certificate_expiration" ]

    arguments = {
        "--address" = "$tls_address$"
        "--port" = "$tls_port$"
        "--hostname" = "$tls_hostname$"
        "--common-name" = "$tls_common_name$"
        "--file" = "$tls_file"
        "--warn" = "$tls_warn$"
        "--crit" = "$tls_crit$"
        "--openssl" = "$tls_openssl$"
    }

    vars.tls_address = "$address$"
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

        check_command = "tls_certificate_expiration"

        vars.tls_hostname = "blog.veloc1ty.de"
}

```

Of course you can change check_interval to the value you desire. Since the plugin calculates in whole days one check every day is enough.

#### Step 3.3: Add a service group (optional)

If you want to add a service group for this check you can simply add the following snippet in `groups.conf`:

```
object ServiceGroup "tls_certificate_expiration" {
    display_name = "TLS certificate expiration"

    assign where service.check_command == "tls_certificate_expiration"
}
```

If you want to add it to multiple hosts work with `apply`!

That's it. Pretty simple. Reload or restart icinga2 and check the result in your browser.

# Possible arguments

The CheckCommand and the plugin accept various arguments:

CheckCommand Variable | Plugin Argument | Description
tls_address | --address | The address of the server
tls_port | --port | The port to connect to. Default: 443
tls_hostname | --hostname | The hostname which should be sent as SNI
tls_common_name | --common-name | The command name of the certificate
tls_file | --file | Path to the file
tls_warn | --warn | Amount of days left until certificate expires. Default: 21
tls_crit | --crit | Amount of days left until certificate expires. Default: 14
tls_openssl | --openssl | Path to the openssl binary. Default: /usr/bin/openssl

Hint: Not every parameter has to be set.



# Examples

Note: The CheckCommand is always the same.
The host "mineralwasser" is given.

## Check a HTTPS Certificate via SNI

```
object Service "tls-blog-veloc1ty-de" {
    import "generic-service"

    check_command = "tls_certificate_expiration"
    host_name = "mineralwasser"

    vars.tls_hostname = "blog.veloc1ty.de"
    vars.tls_common_name = "blog.veloc1ty.de"
    // Note: address is automatically set to the host's address
    // Note: port is default 443
    // Note: If you skip tls_common_name common name checking is disabled
}
```

## Check an IMAP certificate

```
object Service "tls-mail-veloc1ty-de" {
    import "generic-service"

    check_command = "tls_certificate_expiration"
    host_name = "mineralwasser"

    vars.tls_port = "993"
    // Note: address is automatically set to the host's address
    // Note: no SNI is needed, so we can skip "tls_hostname"
}
```

## Check a local file

```
object Service "tls-veloc1ty-ca" {
    import "generic-service"

    check_command = "tls_certificate_expiration"
    host_name = "mineralwasser"

    vars.tls_file = "/path/to/veloc1tyCA.pem"
    // Note: address is automatically set to the host's address
}
```

# Special thanks

Special thanks to Jan from biocrafting.net for notifying me about the SNI issue!
