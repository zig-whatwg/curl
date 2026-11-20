---
c: Copyright (C) Daniel Stenberg, <daniel@haxx.se>, et al.
SPDX-License-Identifier: curl
Title: CURLOPT_HTTP_LENIENT
Section: 3
Source: libcurl
Protocol:
  - HTTP
See-also:
  - CURLOPT_HTTP09_ALLOWED (3)
  - CURLOPT_IGNORE_CONTENT_LENGTH (3)
Added-in: 8.19.0
---

# NAME

CURLOPT_HTTP_LENIENT - parse HTTP/1 response headers the way browsers do

# SYNOPSIS

~~~c
#include <curl/curl.h>

CURLcode curl_easy_setopt(CURL *handle, CURLOPT_HTTP_LENIENT, long bitmask);
~~~

# DESCRIPTION

Pass a long with a bitmask of what an HTTP/1 response may carry that libcurl
otherwise rejects with CURLE_WEIRD_SERVER_REPLY. Each bit makes libcurl handle
that case the way web browsers do:

## CURLHTTPLENIENT_NOCOLON

A response header line that has no colon is ignored: it is not parsed, not
passed to the header callback and not available through curl_easy_header(3).
It still counts toward the size limit of the response headers.

## CURLHTTPLENIENT_BADCLEN

A Content-Length header whose single value is not a number (for example
`Content-Length: none`, or a negative number) is ignored, as if the response
had no Content-Length: the body ends when the connection closes. Two
different lengths, a list of values with anything but one repeated number in
it, or a second Content-Length after an ignored one still fail the transfer.

This matches Fetch's "extract a length" (https://fetch.spec.whatwg.org/), and
what Chrome and Firefox do.

# DEFAULT

0: both cases fail the transfer.

# %PROTOCOLS%

# EXAMPLE

~~~c
int main(void)
{
  CURL *curl = curl_easy_init();
  if(curl) {
    curl_easy_setopt(curl, CURLOPT_URL, "https://example.com");

    /* accept what a browser accepts */
    curl_easy_setopt(curl, CURLOPT_HTTP_LENIENT,
                     CURLHTTPLENIENT_NOCOLON | CURLHTTPLENIENT_BADCLEN);

    curl_easy_perform(curl);
  }
}
~~~

# %AVAILABILITY%

# RETURN VALUE

curl_easy_setopt(3) returns a CURLcode indicating success or error.

CURLE_OK (0) means everything was OK, CURLE_BAD_FUNCTION_ARGUMENT means the
bitmask has an unknown bit, non-zero means an error occurred, see
libcurl-errors(3).
