## What is this, and how does it work?

It is a demonstration of the fact that cross-domain caching, as currently implemented in most or all major browsers, is a significant privacy hole, leaking information about the user's browsing history.

### A similar, recently fixed attack involving link styling

Until a few years ago, a similar attack was possible based upon the differences in styling for visited and unvisited links. Visited links can be styled differently using the `:visited` CSS selector, and indeed are styled differently by default - hence visited links showing up in purple in most browsers. The attack was simple: create an attack page full of links (that is, `<a>` elements) whose `href`s were URLs of interest. Then you use JavaScript to check the link styling - for example, checking whether the links are displayed in purple - for each link. Now you know which of those pages the user has visited.

There were also more sophisticated variants of the attack that could thwart the most obvious security measures browsers could take. One example was using `:visited` styling that would make the page more computationally difficult (and thus slow) to render, and then timing how long the pageload takes.

### How was that fixed?

See http://blog.mozilla.org/security/2010/03/31/plugging-the-css-history-leak/. The fix had a couple of major components: reducing the set of CSS properties that can be used from within a `:visited` selector to only color-related properties, and always giving unvisited style values to JavaScript code trying to read the CSS properties of an element.

### And how does this attack work?

The attack that this repo demonstrates doesn't involve CSS, nor does it look at the browser's history *per se*. Instead, it looks at the browser's *cache*.

A simplified summary of the attack looks something like this:

* Load a known cached image and time how long it takes to load. (The timing can be done using the `onload` event of the `img` element.)
* List the sites that you want to determine if the user has visited. For each site:
  * Create an `img` element pointing to some image that will reliably have been cached by an ordinary user visiting the site. (Favicons, banner images and logos are good choices that will usually change rarely.)
  * Time how long it takes to load
  * If it takes less time than the known cached image, or not much more time, then deduce that the image was cached and that the user has previously visited the site of interest. Otherwise, assume that they have not.

### How long has this been around?

Security researchers knew about it at least 14 years ago. See [this paper](http://sip.cs.princeton.edu/pub/webtiming.pdf) by Edward Felten and Michael Schneider.

### What's the fix?

Rather than have the keys for browser cache entries be merely the URL of the resource being requested, have the key be a tuple of the URL of the resource and the domain from which the request originated. Felten and Schneider call this "Domain Tagging" and propose it in section 8 of [Timing Attacks on Web Privacy](http://sip.cs.princeton.edu/pub/webtiming.pdf).

As noted by Felten and Schneider, there are other caching-related timing attacks on privacy that this fails to prevent (namely, timing attacks on the operating system's DNS cache, and timing attacks on intermediate proxy servers that perform caching). The latter cannot be solved purely by browser vendors; the former may be soluble by the same method.

Additionally, there is a case of CDNs to consider. A significant part of the value of public CDNs comes from cross-domain caching - from the fact that my first visit to a website can be sped up by virtue of some of its scripts or stylesheets having been cached on visits to unrelated sites that requested them from the same CDN.

TODO: write up proposed fix that retains cross-domain caching for CDNs, involving extra header
