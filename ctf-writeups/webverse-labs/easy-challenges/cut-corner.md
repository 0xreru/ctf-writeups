# Cut Corner

Marcus hit Slack's free-tier message limit for the fourth time and decided to build his own. CutCorner launched the following Monday. It mostly works. The Resource Hub's URL fetcher has a blocklist he wrote in an hour and never revisited. The file server on the unusual port was supposed to get auth before anyone cared.<br>

### Initial Reconnaissance

Reading the challenge description will almost always give you a solid clue. Right away, the mention of a "URL fetcher" and a "file server on an unusual port" gave me the idea that this challenge was going to revolve around an LFI (Local File Inclusion) or SSRF (Server-Side Request Forgery) attack.

\
I started up the challenge and scanned the application. I immediately observed a feature that passes a URL parameter (e.g., `?url=`). Since it treats the passed value directly as a URL, this essentially confirmed the SSRF vector.

\
While browsing the application, I also found a conversation between the team talking about their server configs. Marcus mentions spinning up a quick internal file share that is not on the public interface, explicitly stating that it contains `notes.txt` and `creds.txt`. This was the final hint: we need to access a local file server to read the flag.

<figure><img src="../../../.gitbook/assets/image (25).png" alt="" width="478"><figcaption></figcaption></figure>

### Bypassing the SSRF Filter

I started digging deep into SSRF and trying to access the local server. I fired off standard localhost payloads first to see how the application would react:

```
http://127.0.0.1/
http://0.0.0.0/
http://[::1]/
http://2130706433/ (decimal value for 127.0.0.1)
```

The application blocked these attempts. Testing `http://localhost/` returned an explicit error message stating `Private/loopback addresses not allowed`. The developer definitely put a basic blocklist in place.

<figure><img src="../../../.gitbook/assets/image (26).png" alt="" width="563"><figcaption></figcaption></figure>

\
To bypass this, I used a hex representation of the `127.0.0.1` IP address.

```
http://0x7f000001/
```

This payload successfully bypassed the filter, returning a `200 OK` status and the HTML for the `#general` channel. We now have internal access to the loopback address.<br>

<figure><img src="../../../.gitbook/assets/image (28).png" alt="" width="563"><figcaption></figcaption></figure>

### Port Fuzzing and Flag Retrieval

Now, what's next? We know from the briefing that the file server is hosted on an "unusual port." We need to specify the correct open port to hit the exact server hosting the files.

\
I started testing common development and alternative HTTP ports like `8000`, `8080`, and `3000`.\
I quickly got a hit on port `3000`.&#x20;

Accessing `http://0x7f000001:3000/` returned a directory listing containing `creds.txt`, `notes.txt`, and our target: `flag.txt`.

<figure><img src="../../../.gitbook/assets/image (29).png" alt="" width="563"><figcaption></figcaption></figure>

\
From here, getting the final flag was as simple as appending the filename to our bypassed URL:file

```
http://0x7f000001:3000/flag.txt
```

### References

* [SSRF Attack Guide](https://hackviser.com/tactics/pentesting/web/ssrf)

### 🔑 Key Takeaways

* Always read the challenge lore—it's basically free recon.
* Never trust a security filter written in an hour by a tired developer. 😅
* Alternative IP encoding (Hex, Octal, Decimal) is your best friend for lazy blocklists.

###

If you have any questions feel free to dm me [xreru](https://app.gitbook.com/u/sV63NjWn0kbva4C066LUjfLD3y92 "mention") or in [linkedin](https://www.linkedin.com/in/reru/)
