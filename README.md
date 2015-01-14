NAME
    cyrus2dovecot - convert Cyrus folders to Dovecot

SYNOPSIS
    cyrus2dovecot [-cdmq] [-C *cyrus-inbox*] [-D *dovecot-inbox*]
    [-E *edit-foldernames*] [-F *dovecot-uidlist-format*]
    [-H *dovecot-host*] [-N *default-quota*] [-O *cyrus-quota-format*]
    [-Q *cyrus-quota*] [-S *cyrus-seen*] [-U *cyrus-sub*] [*user* ...]

    cyrus2dovecot -h | -v

DESCRIPTION
    cyrus2dovecot converts the e-mails of one or more *user*s from Cyrus
    format to Dovecot Maildir++ folders. If no *user* is specified, the
    *user* names are read from the standard input, one per line. Message
    "UID"s, "INTERNALDATE"s, IMAP folder subscriptions, the "UIDVALIDITY"
    and "UIDNEXT" values for each folder, as well as all IMAP flags
    (including the first 26 user-defined keywords) are preserved during the
    conversion. The generated e-mail filenames include the Maildir++
    extensions "S=<size>" and "W=<vsize>" (which are used by Dovecot for
    better performance). Optionally, Maildir++ maildirsize files are
    created. Expunged messages of the cyrus message store are not migrated
    to dovecot.

OPTIONS
    Within the specified *PATH*s, any occurrence of %u will be replaced by
    the current *user* name, any occurrence of "%*n*u" will be replaced by
    the *n*'th character of that *user* name, any occurrence of %h will be
    replaced by Cyrus' directory "hash" character for that *user* name
    (i.e., %h is equivalent to %1u if the first character of the *user* name
    is a lowercase letter), and any occurrence of %x will be replaced by
    Cyrus' "fulldirhash" character for that *user* name. When you have
    virtual domains in your cyrus installation then these additional
    substitutes can be made: %d will be replaced by the domain part of the
    username %U is the user part of the username %g is the first character
    of the domain

    However, within the specified --cyrus-quota *PATH* (if any), these
    replacements will only be done if the --cyrus-quota-format *VERSION* is
    set to 1.

    The default settings can be found (and modified) at the top of the
    cyrus2dovecot script.

    -C, --cyrus-inbox=*PATH*
            Use this *PATH* to the *user*'s INBOX folder in Cyrus.

    -c, --dovecot-crlf
            Store e-mails with "CR+LF" instead of plain "LF". This flag
            should be specified if the "mail_save_crlf" option is set to
            "yes" in the Dovecot configuration.

    -D, --dovecot-inbox=*PATH*
            Use this *PATH* to the *user*'s INBOX folder in Dovecot.

    -d, --debug
            Print information which is usually only useful for debugging to
            the standard output.

    -E, --edit-foldernames=*SUBSTITUTION*
            Apply the specified *SUBSTITUTION* to the name of each Maildir++
            folder and subscription using Perl code such as
            "eval('$name=~'.$substitution)", where $name holds either the
            string INBOX (which denotes the main Maildir) or the full
            Maildir++ folder name (e.g., .sub.folder), and $substitution
            holds the specified *SUBSTITUTION*. The resulting $name will be
            used as the Maildir++ folder's name. This option may be
            specified multiple times, in which case each of the
            *SUBSTITUTION*s will be applied to each Maildir++ folder name in
            the order specified on the command line. Note that while Dovecot
            stores the subscribed folder names without the leading "." of
            Maildir++ subfolders, cyrus2dovecot adds a leading "." to each
            subscribed subfolder name before applying the specified
            *SUBSTITUTION*(s) and removes it afterwards (if it still exists)
            in order to simplify the matching.

    -F, --dovecot-uidlist-format=*VERSION*
            Create the dovecot-uidlist files using this format *VERSION*.
            For Dovecot releases older than 1.0.2, *VERSION* 1 must be
            specified; otherwise, *VERSION* 3 can be used.

    -H, --dovecot-host=*NAME*
            Use this host *NAME* for the Maildir++ e-mail file's basename.

    -h, --help
            Print usage information to the standard output and exit.

    -m, --dump-meta
            Print a dump of the data structure which holds the metadata
            gathered from scanning the Cyrus folders of a user to the
            standard output.

    -N, --default-quota=*BYTES*
            Create a Maildir++ maildirsize file for each *user*, and set the
            quota limit to the specified number of *BYTES* unless
            --cyrus-quota is also specified, in which case a *user*-specific
            quota would override the --default-quota limit. Specifying 0
            *BYTES* disables the creation of maildirsize files unless
            --cyrus-quota is also specified.

    -O, --cyrus-quota-format=*VERSION*
            Expect the quota database file specified via --cyrus-quota to be
            present in this format *VERSION*, where *VERSION* 1 denotes the
            "quotalegacy" format and *VERSION* 2 denotes the "skiplist" or
            the "flat" text format (cyrus2dovecot will autodetect which of
            those two formats is used if *VERSION* 2 is specified). This
            option is ignored if --cyrus-quota is not specified.

    -Q, --cyrus-quota=*PATH*
            Use this *PATH* to the quota database file in Cyrus, and create
            a Maildir++ maildirsize file for each *user* whose quota limit
            is found in that file.

    -q, --quiet
            Suppress the line usually printed to the standard output for
            each *user* whose e-mails were successfully converted. Error
            messages, if any, will still be printed to the standard error
            output.

    -S, --cyrus-seen=*PATH*
            Use this *PATH* to the *user*'s seen database file in Cyrus. If
            cyrus.seen is specified as the *PATH*, cyrus2dovecot expects an
            old-style cyrus.seen file in every Cyrus folder.

    -U, --cyrus-sub=*PATH*
            Use this *PATH* to the *user*'s subscription database file in
            Cyrus.

    -v, --version
            Print version information to the standard output and exit.

RETURN VALUE
    cyrus2dovecot exits 0 on success. If a non-fatal error occurs,
    cyrus2dovecot prints a message to the standard error output and then
    tries to convert the e-mails of the remaining *user*s (if any), but it
    exits >0 regardless of whether or not those conversions succeed. If a
    fatal error occurs, cyrus2dovecot exits >0 immediately.

EXAMPLES
    Given that the default settings specified at the top of the
    cyrus2dovecot script are correct and that /tmp/users holds the names of
    all *user*s whose e-mails should be converted (one per line), the
    following command would convert all e-mails of those *user*s from Cyrus
    to Dovecot:

            cyrus2dovecot < /tmp/users

    Given that the path to the INBOX in Cyrus is /var/spool/imap/user/%u
    (where %u denotes the *user* name), that Cyrus stores the seen and
    subscription databases within the directory /var/imap/user/%h, and that
    Cyrus stores "quotalegacy" files within the directory /var/imap/quota/%h
    (where %h denotes Cyrus' directory "hash" character for that *user*
    name, respectively), the following command would convert all e-mails of
    the *user*s "bill" and "george" from Cyrus to Dovecot, and the result
    would be stored below /tmp/dovecot (including maildirsize files for both
    users if their quota limits are found):

            cyrus2dovecot --cyrus-inbox /var/spool/imap/user/%u     \
                          --cyrus-seen /var/imap/user/%h/%u.seen    \
                          --cyrus-sub /var/imap/user/%h/%u.sub      \
                          --cyrus-quota /var/imap/quota/%h/user.%u  \
                          --cyrus-quota-format 1                    \
                          --dovecot-inbox /tmp/dovecot/%u/Maildir   \
                          bill george

    A script such as the following could be used in order to convert all
    e-mails of all *user*s (of course, the pathnames and the desired quota
    limit may have to be adjusted, and if the "hashimapspool" option is
    enabled in the Cyrus configuration, /? must be appended to the $in
    path):

            #!/bin/sh

            in=/var/spool/imap/user         # Cyrus INBOXes.
            db=/var/imap/user/?             # Cyrus seen/subscription files.
            out=/tmp/dovecot                # Dovecot Maildirs.
            log=/tmp/conversion.log         # Log of successful conversions.
            err=/tmp/error.log              # Log of conversion errors.
            quota=2147483648                # 2 GiB quota (for maildirsize).

            for u in `find $in/. \! -name . -prune -exec basename \{\} \;`
            do
                    cyrus2dovecot --cyrus-inbox $in/$u              \
                                  --cyrus-seen $db/$u.seen          \
                                  --cyrus-sub $db/$u.sub            \
                                  --default-quota $quota            \
                                  --dovecot-inbox $out/$u/Maildir   \
                                  $u 2>&1 >>$log | tee -a $err >&2
            done

    In order to create all folders (except for the INBOX) as subfolders of
    the INBOX in Dovecot, the following argument could be added to the
    cyrus2dovecot command line:

            --edit-foldernames 's/^\./.INBOX./'

    Cyrus transparently replaces any "." character in folder names with a
    "^" character. Dovecot supports "." characters in Maildir++ folder names
    if the "listescape" plugin is used, which replaces any "." character in
    folder names with the string "\2e". The following argument could be
    added to the cyrus2dovecot command line in order to replace any "^"
    character in Cyrus folder names with "\2e" for the Maildir++ folder
    name:

            --edit-foldernames 's/\^/\\2e/g'

    Dovecot 1.1 and newer support using folders such as Maildir/sub/folder
    (as opposed to Maildir/.sub.folder) if ":LAYOUT=fs" was added to the
    "mail_location" in the Dovecot configuration. The following
    cyrus2dovecot arguments could be specified in order to create such
    folders by removing the leading dot from Maildir++ subfolder names and
    then substituting any following dots with slashes:

            --edit-foldernames 's/^\.//'    \
            --edit-foldernames 's/\./\//g'

    If the seen states, subscriptions, or quotas are stored in Berkeley
    databases, they must first be converted for cyrus2dovecot using a
    command such as the following:

            cvt_cyrusdb /var/imap/user/b/bill.seen berkeley \
                        /tmp/imap/user/b/bill.seen skiplist

CAVEATS
    cyrus2dovecot assumes that the user has no e-mails in Dovecot yet and
    that neither his Cyrus folders nor his Dovecot folders will be accessed
    by another process during the conversion.

    If "%*n*u" is specified within any *PATH* on the command line, all
    *user* names must have a length of at least *n* characters. Otherwise,
    cyrus2dovecot will die with an exception.

    If folder name substitutions are specified via --edit-foldernames, the
    resulting Maildir++ folder names must be unique.

RESTRICTIONS
    Cyrus' seen and subscription databases must be present either in the
    "skiplist" format or in the "flat" text format, and Cyrus' quota
    database(s) (if any) must be present either in one of those formats or
    in the "quotalegacy" format, as cyrus2dovecot doesn't support Berkeley
    databases. However, Berkeley databases can be converted to one of the
    supported formats using cvt_cyrusdb(8), see the "EXAMPLES".

    In maildirsize files created by cyrus2dovecot, no limit for the number
    of messages is specified (as such a limit does not seem useful).

    Cyrus' ACL settings are not converted.

COMPATIBILITY
    cyrus2dovecot is supposed to work with all Cyrus releases up to (at
    least) version 2.3.x. So far, it has been tested with Cyrus 1.4, 2.1.18,
    2.2.12, and 2.3.12p2.

SEE ALSO
    Other tools for converting e-mails from Cyrus to Dovecot can be found at
    <http://wiki.dovecot.org/Migration/Cyrus>.

AUTHOR
    Written by Holger Weiß <holger@ZEDAT.FU-Berlin.DE> at Freie Universität
    Berlin, Germany, Zentraleinrichtung für Datenverarbeitung (ZEDAT).

COPYRIGHT AND LICENSE
    Copyright (c) 2008 Freie Universität Berlin. All rights reserved.

    This program is free software; you can redistribute it and/or modify it
    under the same terms as Perl itself. See perlartistic. This program is
    distributed in the hope that it will be useful, but WITHOUT ANY
    WARRANTY; without even the implied warranty of MERCHANTABILITY or
    FITNESS FOR A PARTICULAR PURPOSE.

HISTORY
            $Log: cyrus2dovecot,v $
            Revision 1.2  2008/09/24 09:52:33  holger
            Message seen states are now parsed more efficiently with regard
            to performance and memory usage.  Apart from that, minor code
            cleanups have been applied.

            Revision 1.1  2008/09/22 08:36:44  holger
            Initial release.

            Revision 1.3 2012/11/11 midnight        alex
            added basic virtual domain support and unixhirarchiesep

            Revision 1.4 2012/12/06 midnight  alex
            Fixed subscriptions for virtual domains

            Revision 1.5 2012/12/08 01:26:00 alex
            added support for sieve rules

            Revision 1.6 2015/01/14 22:40:00 a-schild
            added support for delayed expunge (Don't copy expunged messages)

