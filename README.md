## Overview: Using libcurl with Custom Configuration

Instead of hardcoding all options directly into your C program, you can create a curl.conf file containing your preferred configurations.
This approach keeps your code cleaner and allows you to easily adjust settings without modifying the source code.


---

Step-by-Step Guide

1. Create a Custom curl.conf File

First, create a custom configuration file, typically named curl.conf, with the necessary settings to route traffic through Tor.
Example:
```
# curl.conf

# Specify the SOCKS5 proxy (Tor)
proxy = "socks5h://127.0.0.1:9050"

# Optional: Increase verbosity for debugging
verbose = true

# Optional: Specify a user-agent string
user-agent = "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; Trident/6.0)"

Save this file in a directory of your choice (for example, /etc/curl/).
```

---

2. Modify the C Code to Use curl.conf

libcurl does not load a configuration file by default. However, you can set the options directly in your C code to match what you'd put in curl.conf.

Here’s an example C program:
```
#include <stdio.h>
#include <curl/curl.h>

// Callback function to write data to stdout
size_t write_data(void *ptr, size_t size, size_t nmemb, FILE *stream) {
    size_t written = fwrite(ptr, size, nmemb, stream);
    return written;
}

int main(void) {
    CURL *curl;
    CURLcode res;

    curl = curl_easy_init();
    if(curl) {
        // Use a custom user agent
        curl_easy_setopt(curl, CURLOPT_USERAGENT, "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; Trident/6.0)");

        // Use Tor SOCKS5 proxy
        curl_easy_setopt(curl, CURLOPT_PROXY, "socks5h://127.0.0.1:9050");

        // Specify the .onion URL
        curl_easy_setopt(curl, CURLOPT_URL, "http://hss3uro2hsxfogfq.onion");

        // Set up the callback to handle the response
        curl_easy_setopt(curl, CURLOPT_WRITEFUNCTION, write_data);
        curl_easy_setopt(curl, CURLOPT_WRITEDATA, stdout);

        // Perform the request
        res = curl_easy_perform(curl);

        // Check for errors
        if(res != CURLE_OK)
            fprintf(stderr, "curl_easy_perform() failed: %s\n", curl_easy_strerror(res));

        // Cleanup
        curl_easy_cleanup(curl);
    }

    return 0;
}
```

---

3. Running the Program with the Custom Configuration

When using libcurl in a C program, configurations like proxy and user-agent are set programmatically (as above), so you don't need to pass --config as you would with the CLI.

However, if you want to use curl from the command line and load a configuration file, you can do:

curl --config /etc/curl/curl.conf http://hss3uro2hsxfogfq.onion


---

Benefits of This Approach

1. Separation of concerns: Keeps your C code cleaner and easier to maintain.


2. Ease of modification: Adjust settings without recompiling your C code.


3. Reusable configurations: Share the same curl.conf across different projects or systems for consistent behavior.




---

Summary

Using a custom curl.conf file (or matching settings in C) gives you flexibility and maintainability when interacting with Tor or other proxies.
This approach is especially useful when working in environments where different users or systems might require different settings or for rapid testing and debugging.


---

Setting Up and Building curl from Source with SOCKS5 Support

1. Install Necessary Dependencies

sudo apt-get update
sudo apt-get install build-essential autoconf automake libtool pkg-config libssl-dev libcurl4-openssl-dev

build-essential: Compiler and build tools (gcc, g++, make, etc.)

autoconf, automake, libtool, pkg-config: Required for generating build scripts.

libssl-dev: SSL support for HTTPS.

libcurl4-openssl-dev: Development files for curl.



---

2. Download and Extract curl Source Code

wget https://curl.se/download/curl-8.8.0.tar.gz
tar -xvf curl-8.8.0.tar.gz
cd curl-8.8.0


---

3. Configure curl with SOCKS5 Support

./configure --with-openssl


---

4. Compile and Install curl

make
sudo make install

This will replace the existing curl binary with your newly built version, which includes full SOCKS5 support.


---

5. Verify the Installation

curl-config --features

Check that socks5, SSL, HTTPS, and other desired features are listed.


---

6. Using the New curl with Tor

curl -x socks5://127.0.0.1:9050 https://check.torproject.org

This routes traffic through Tor using the SOCKS5 proxy at 127.0.0.1:9050.


---

Common Errors Fixed by This Setup

The following errors occur when your C compiler or curl build lacks proper SOCKS5 support:

Failed to fetch page http://zqktlwi4fecvo6ri.onion: error sending request for url: failed to lookup address information: Name or service not known
Failed to fetch page http://onionlinksv3bdm5.onion: error sending request for url: failed to lookup address information: Name or service not known
...

By rebuilding curl properly with SOCKS5 support, these errors are resolved.
This is especially important on systems like Nethunter, where SOCKS proxies (e.g., socks5h, socks4) might not be properly configured out of the box.


---

Final Summary

By following these steps, you ensure your C programs and curl commands can fully utilize Tor through SOCKS5 proxying.
This provides secure, anonymized HTTP(S) requests, greater flexibility, and avoids manual proxy or DNS resolution issues.


---


