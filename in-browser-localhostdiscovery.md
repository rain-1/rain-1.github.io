This might be well known and boring to many of you, but it was a big shocker to me learning it for the first time. So I want to share what I learned!

Basically any old webpage can perform local network host discovery on you. To implement this I made a webpage which attempts to load images from addresses `192.168.1.x`. If you watch in the browser console it'll show either `net::ERR_CONNECTION_REFUSED` for a host that's up or `net::ERR_ADDRESS_UNREACHABLE` for a host that doesn't exist.  This is a CORS error which the javascript on the webpage is not allowed to differentiate by catching. But one error takes 3 ms to happen and the other takes 3 seconds!

So here's my example: [https://jsfiddle.net/pwb0dkyh/](https://jsfiddle.net/pwb0dkyh/) it just checks the first 20 local ip addreses. I've only tried it on my own computer but it was able to correctly detect my laptop and phone and list the rest of the ips as down. I don't know if it'll work perfectly for others, would be interesting to hear.

A related thing a webpage in your browser might do is connect to localhost and control any unauthenticated local services. Taviso used this to great effect here [https://github.com/spesmilo/electrum/issues/3374](https://github.com/spesmilo/electrum/issues/3374)

**Conclusion** Just loading up an arbitrary web link that someone (suspicious) gives you is a LOT more dangerous than I previously realized. It's possible to scan locally and even open connections and control local services.

-------------

copy of the code from the pastebin, in case the pastebin disappears.

```
// in testing it took:
// * 5ms to error on a host that is not up
// * 3055ms to error on a host that is up
//
// if your network has different timing characteristics you may need to modify this value.

var cutoff = (5 + 3055)/2.0;

function img_load_time(link) {
	var startTime = new Date().getTime();
	var img = new Image();
  img.onerror = function() {
  	var loadtime = new Date().getTime() - startTime;
    var result = (loadtime < cutoff) ? "UP" : "DOWN";
  	var node = document.createElement("li");
		var textnode = document.createTextNode(link + " " + result);
		node.appendChild(textnode);
    document.getElementById("hosts").appendChild(node);
    next();
  }
  img.src = link;
}

var i = 0;
function next() {
  if(i > 20) return;
	img_load_time("https://192.168.1."+i+"/");
  i++;
}

next();
```

----

# Update: Jan 1 2019

Giacomo brought to my attention an instance of this kind of attack used in the wild:

"Recently it was discovered that a Russian Governement's agency is using these exploits to detect some of the tools that can be used to detect these attacks despite the HTTP cache control directives on the home page of rkn.gov.ru: the beautified script shows from line 2615 on which security tools where scanned."

This was brought up on telegram and the spyware script was subsequently blocked by AdGuard plugin

* https://t.me/usher2/702
* https://t.me/rknapocalypsetime/157
* https://github.com/AdguardTeam/AdguardFilters/issues/27243

Including the telegram posts here for reference:

```
Escher II
On the main site of Roskomnadzor http://rkn.gov.ru in the code of the page there is such a Javascript: 
https://rkn.gov.ru/5a71c95863f38b1dcfcbb763.js For
some reason, it goes (by itself) through different ports to 127.0.0.1 (t ... e. on your computer or mobile phone, or tablet, or a cosmetic bag - what you use there for going to the sites) and sends something to the Roskomnadzor server (if there is something there).

What it “breaks” to is some kind of debugging utility and / or security test proxy. It is not clear why Roskomnadzor needs this. Maybe this is the developers' self-defense, which they forgot to remove from the working version. Maybe this is a collection of information about suspicious machines on the network. But the fact is that information about your computer is not going to be with your knowledge. And not that public information that you would like to make public.

I hope everyone enjoyed the news: https://t.me/usher2/702
Really similar to the detection utilities running on the user machine, here are just some of the lines that puzzled me: 

t.src = " http://127.0.0.1:8182/favicon. ico ", t.onload = function () { 
var e =" Tool: Acunetix; Open Port: 8182 "; 
d.push (e), u (e) 

In short - if you have a favicon.ico (site icon) on port 8182, the script thinks that you are a terrible hacker. So if all of a sudden you are a web developer and you are not lucky enough to have a locally deployed saytik on 8182 and it has ABOUT GOD an icon - the RKN thinks you are a mamkin hacker. This is a success.
```

So it seems that anybody who would access this particular russian webpage would be scanned to check if they had certain services running on their local computer judged to be "hacker tools" and I can only assume this information would be logged and kept long term.

It really seems to be like we have a problem here, if I access an external webpage it shouldn't be able to access my local pages and network. I know that IPFS takes advantage of this functionality. But other than that am I missing some reason why this feature is needed?
