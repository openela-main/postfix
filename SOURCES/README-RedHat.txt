This Postfix build behaves differently from the upstream postfix-3.5.8.
It's because in RHEL-8 backward compatibility is kept to postfix-3.3.1.

For the upstream postfix-3.5.8 behavior either run the following commands:

# postconf info_log_address_format=external
# postconf smtpd_discard_ehlo_keywords=
# postconf rhel_ipv6_normalize=yes

Or go through the following steps:

1. Change the configuration option 'info_log_address_format' to 'external'.
In RHEL-8 it's by default set to 'internal' to mitigate [Incompat 20191109].

2. Change the configuration option 'smtpd_discard_ehlo_keywords' to ''.
In RHEL-8 it's by default set to 'chunking' to mitigate [Incompat 20180826].

3. Add RHEL-8 specific configuration option 'rhel_ipv6_normalize' and set it
to 'yes'. In RHEL-8 this option was added to mitigate [Incompat 20190427].

Details from the upstream RELEASE_NOTES:

[Incompat 20191109]
Postfix daemon processes now log the from= and
to= addresses in external (quoted) form in non-debug logging (info,
warning, etc.).  This means that when an address localpart contains
spaces or other special characters, the localpart will be quoted,
for example:

    from=<"name with spaces"@example.com>

Older Postfix versions would log the internal (unquoted) form:

    from=<name with spaces@example.com>

The external and internal forms are identical for the vast majority
of email addresses that contain no spaces or other special characters
in the localpart.

Specify "info_log_address_format = internal" for backwards
compatibility.

The logging in external form is consistent with the address form
that Postfix 3.2 and later prefer for table lookups. It is therefore
the more useful form for non-debug logging.

[Incompat 20180826]
The Postfix SMTP server announces CHUNKING (BDAT
command) by default. In the unlikely case that this breaks some
important remote SMTP client, disable the feature as follows:

/etc/postfix/main.cf:
    # The logging alternative:
    smtpd_discard_ehlo_keywords = chunking
    # The non-logging alternative:
    smtpd_discard_ehlo_keywords = chunking, silent_discard

See BDAT_README for more.

[Incompat 20190427]
Postfix now normalizes IP addresses received
with XCLIENT, XFORWARD, or with the HaProxy protocol, for consistency
with direct connections to Postfix. This may change the appearance
of logging, and the way that check_client_access will match subnets
of an IPv6 address.
