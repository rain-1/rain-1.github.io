This might be well known and boring to many of you, but it was a big shocker to me learning it for the first time. So I want to share what I learned!

Basically any old webpage can perform local network host discovery on you. To implement this I made a webpage which attempts to load images from addresses `192.168.1.x`. If you watch in the browser console it'll show either `net::ERR_CONNECTION_REFUSED` for a host that's up or `net::ERR_ADDRESS_UNREACHABLE` for a host that doesn't exist.  This is a CORS error which the javascript on the webpage is not allowed to catch. But one error takes 3 ms to happen and the other takes 3 seconds!

So here's my example: https://jsfiddle.net/pwb0dkyh/ it just checks the first 20 local ip addreses. I've only tried it on my own computer but it was able to correctly detect my laptop and phone and list the rest of the ips as down. I don't know if it'll work perfectly for others, would be interesting to hear.

A related thing a webpage in your browser might do is connect to localhost and control any unauthenticated local services. Taviso used this to great effect here https://github.com/spesmilo/electrum/issues/3374

**Conclusion** Just loading up an arbitrary web link that someone (suspicious) gives you is a LOT more dangerous than I previously realized. It's possible to scan locally and even open connections and control local services.

