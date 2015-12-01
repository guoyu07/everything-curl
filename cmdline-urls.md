## URLs

curl is called curl exactly because a substring in its name is URL (Uniform
Resource Locator). It operates on URLs. URL is the name we casually use for
the web address strings, like the ones we usually see prefixed with http:// or
starting with www.

URL is strictly speaking the former name for this. URI (Uniform Resource
Identifier) is the more modern and correct name for it. It is defined in
RFC3986.

Where curl accepts a "URL" as input, it is then really a "URI". Most of the
protocols curl understands also have a corresponding URI syntax document that
describes how that particular URI format works.

curl assumes that you give it a valid URL and it only does limited checks of
the format in order to extract the information it deems necessary to perform
its operation. You can for example most probably pass in illegal letters in
the URL without curl noticing or caring and it will just pass them on.

### Scheme

URLs start with the "scheme", which is the official name for the "http://"
part. That tells which protocol the URL uses. As a convenience, curl also
allows users to leave out the scheme part from URLs. Then it guesses which
protocol to use based on the first part of the host name.

### Name and password

After the scheme, there can be a possible user name and password embedded.
The use of this syntax is usually frowned upon these days since you easily
leak this information in scripts or otherwise. For example, listing the
directory of an FTP server using a given name and password:

    $ curl ftp://user:password@example.com/

The presence of user name and password in the URL is completely optional. curl
also allows that information to be provide with normal command line options,
outside of the URL.

### Host name or address

The host name part of the URL is of course simply a name that can be resolved
to an numerical IP address, or the numerical address itself. When specifying a
numerical address, use the dotted version for IPv4 addresses:

    $ curl http://127.0.0.1/

... and for IPv6 addresses the numerical version needs to be within square
brackets:

    $ curl http://[::1]/

When a host name is used, the converting of the name to an IP address is
typically done using the system's resolver functions. That normally lets a
sysadmin provide local name lookups in the `/etc/hosts` file (or equivalent).

### Port number

Each protocol has a "default port" that curl will use for it, unless a
specified port number is given. The optional port number can be provided
within the URL after the host name part, as a colon and the port number
written in decimal. For example, asking for a HTTP document on port 8080:

    $ curl http://example.com:8080/

With the name specified as an IPv4 address:

    $ curl http://127.0.0.1:8080/

With the name given as an IPv6 address:

    $ curl http://[fdea::1]:8080/

### Path

Every URL contains a path. If there's none given, "/" is implied. The path is
sent to the specified server to identify exactly which resource that is
requested or that will be provided.

The exact use of the path is protocol dependent. For example, getting a file
README from the default anonymous user from an FTP server:

    $ curl ftp://ftp.example.com/README

For the protocols that have a directory concept, ending the URL with a
trailing slash means that it is a directory and not a file. Thus asking for a
directory list from an FTP server is implied with such a slash:

    $ curl ftp://ftp.example.com/tmp/

### Fragment

URLs offer a "fragment part". That's usually seen as a hash symbol (#) and a
name for a specific name within a web page in browsers. curl supports
fragments fine when a URL is passed to it, but the fragment part is never
actually sent over the wire so it doesn't make a difference to curl's
operations whether it is present or not.

### Browsers' "address bar"

It is important to realize that when you use a modern web browser, the
"address bar" they tend to feature at the top of their main windows are not
using "URLs" or even "URIs". They are in fact mostly using IRIs, which is a
superset of URIs to allow internationalization like non-latin symbols and
more, but it usually goes beyond that too as they tend to for example handle
spaces and do magic things on percent encoding in ways none of these mention
specifications say a client should do.

The address bar is quite simply an interface for humans to enter and see
URI-like strings.

Sometimes the differences between what you see in a browser's address bar and
what you can pass in to curl is significant.

## Many options and URLs

As mentioned above, curl supports hundreds of command line options and it also
supports an unlimited number of URLs. If your shell or command line system
supports it, there's really no limit to how long command line you can pass to
curl.

curl will parse the entire command line first, apply the wishes from the
command line options used, and then go over the URLs one by one (in a left to
right order) to perform the operations.

For some options (for example -o or -O that tell curl where to store the
transfer), you may want to specify one option for each URL on the command
line.

curl will return an exit code for its operation on the last URL used.

## Separate options per URL

In previous sections we described how curl always parses all options in the
whole command line and applies those to all the URLs that it transfers.

That was a simplification: curl also offers an option (-;, --next) that
inserts a sort of boundary between a set of options and URLs that it will
apply the options for. When the command line parses finds a --next option, it
applies the following options to the next set of URLs to operate with. The
--next option thus works as a *divider* between a set of options and URLs. You
can use as many --next options as you please.

As an example, we do a HTTP GET to a URL and follow redirects, we then make a
second HTTP POST to a different URL and we round it up with a HEAD request to
a third URL. All in a single command line:

    $ curl --location http://example.com/1 --next
      --data sendthis http://example.com/2 --next
      --head http://example.com/3

Trying something like that _without_ the --next options on the command line
would generate an illegal command line since curl would attempt to combine
both a POST and a HEAD:

    Warning: You can only select one HTTP request method! You asked for both POST
    Warning: (-d, --data) and HEAD (-I, --head).

## Connection reuse

Setting up a TCP connection and especially a TLS connection can be a slow
process, even on high bandwidth networks.

It can be useful to remember that curl has a connection pool internally which
keeps previously used connections alive and around for a while after they were
used so that subsequent requests to the same hosts can reuse an already
established connection.

Of course they can only be kept alive for as long as the curl tool is running,
but it is a very good reason for trying to get several transfers done within
the same command line instead of running several independent curl command line
invokes.