Website Performance Optimization
--------------------------------

#Introduction
The **critical rendering path** is the sequence of steps that the browser goes through to render the page. The browser does a lot of work by constructing the DOM, CSS object model (CSSOM), layout, and paint. Optimizing the critical rendering path will reliabilty help you render your pages in less than a second.
Basically, the CPR is the process a browser goes through to convert HTML, CSS, and JS into actual pixels on the screen.

##What are your goals for taking this course?

There are 5 elements to making effective goals: clarity, challenge, commitment, feedback, and complexity.

I have two goals: one is educational and the other is practical
(1) I've always wanted to gain a fundamental understanding the mechanism of how browsers and webpages work together and how performance is affected by that mechanism. The understanding will fill in the gaps of my knowledge whenever I read technical blogs or design my own apps. Success is measured when: (a) I complete this course, and (b) write my summary notes for personal reference in my local nanodegree directory. 

(2) The practical goal is exactly that - I want to practice the different techniques I'll learn in this course to reliably build performant apps that result in great user experiences. Success is measured by scoring a minimum of 90 when analyzing my personal projects using PageSpeed Insights.

#How does the browser render a page?
Learn more about [the Critical Rendering Path](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/) with Google's Web Fundamentals (see sources). 

The browser first grabs HTML to generate the DOM. It then grabs the CSS to create the CSSOM. Then the browser combines the two to create the **Rendering Tree** and figure where everything goes on the page which is the **Layout** step. Finally, you can **paint** pixels on the actual screen.

##HTML to DOM
Check out the [demo page](http://udacity-crp.herokuapp.com/cssom.html). 

The browsers follows a specific instruction to process HTML of a webpage. It parses the HTML by tokenizing the characters into tags. A separate process consumes those tags into node objects. After consuming all the tokens, the browser creates a DOM which is a tree structure that captures the content and properties of the HTML and relationship between all the nodes. The DOM is the full-parsed representation of the HTML markup. The browser creates the DOM incrementally. We can take advantage of this to speed up rendering of these pages.
Here we arrive at our first technique: **incremental HTML delivery**. Flushing is when the server sends the initial part of the HTML document to the client before the entire response is ready. All major browsers start parsing the partial response, which when done correctly results in a page that loads and feels faster. This depends on when to flush the partial HTML document response. The flush should occur before the expensive parts of of the bakckend work, but after the initial response has enough content to keep the browser busy.  Read more about [flushing the document early](http://www.stevesouders.com/blog/2009/05/18/flushing-the-document-early/). 
The important part is that the server does not have to wait to render the full response before returning it to the client. The sooner you can flush some data, the sooner the browser can start building the DOM, and discover and dispatch requests for other critical resources.
See [Timeline Chrome Devtools](https://developer.chrome.com/devtools/docs/timeline).

###How the DOM is built
- HTML response > Tokens > Nodes > DOM Tree
- A single DOM node will start with a **startTag** token and end with an **endTag** token. Between the **startTag** token and **endTag** token come other tokens which define the DOM node.
- Nodes contain all relevant information about the HTML element
- Nodes are connected into a DOM tree based on token hierarchy

By the way, DOM construction is incremental!

##CSSOM
- Characters > Tokens > Nodes > CSSOM

CCSOM is like a DOM in that it is a tree structure. But CSSOM differs in the CSS rules cascade down. Furthermore, you can't use a partial CSSOM tree because it may render the style of a page incorrectly. The browser blocks page rendering until all it received and has processed all of the CSS. In other words, **CSS is render blocking**.

###Why does the browser match CSS selectors from right to left?
Read a [StackOverflow discussion](http://stackoverflow.com/questions/5797014/why-do-browsers-match-css-selectors-from-right-to-left). Basically, by matching the rightmost part of the selector (which is the child) the browser avoids retraversing the tree depth which results in huge savings of work spent looking for the element. The moral of the story is, the more general selector is easier to evaluate.

###Measure first, optimize later
You can use Chrome Devtools to save a timeline trace and load a timeline trace to measure how long it takes for the CSS to fully process. The traces are just JSON files. They will come in handy to get a baseline for the performance of your CSS and then optimize as needed.

##Render Tree
So the DOM contains all the content of a page and the CSSOM contains all the style of the page. How do we take the content and style and turn them into pixels onto the screen? One of the most important properties of the **render tree** is that it only captures visible content. 

According to the [Render Tree construction](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-tree-construction), the browser starts at the root node of the DOM and equivalently looks at the root node of the CSSOM. If there is a style applied to the DOM node (as specified by the CSSOM), the browser constructs the first node of the Render tree. The browser traverses deeper through the DOM and CSSOM and constructs more nodes of the Render tree. Note that since the Render tree only captures visible content, it can prune off the traveral of whole sections of the DOM and CSSOM tree.


##Layout

###Calculating positions and dimensions

The **Layout** step figures out where and how all the elements are positioned on the page. Learn more about [layout viewport and the basics of responsive web design](https://developers.google.com/web/fundamentals/layouts/rwd-fundamentals/).

	<meta name="viewport" content="width=device-width">

This meta tag is very important to include for responsive web design. It tells the browser that the width of the layout viewport is equal to the device width. Without using the viewport meta tag, the browser defaults to a viewport width of 980px which is optimized for large screens. For example, if you've ever encountered a page on your smartphone where you have to zoom many times to read the text, that is an example where that page did not specify a viewport width explicity.

###Analyzing layout in Devtools
Layout can be triggered by device orientation change on mobile, a window resize, or any other action that modifies the content of the DOM, e.g. adding or removing content from the DOM tree, toggling CSSOM properties on a node, and so on!
So how do you go about optimizing for rerunning layout whenever the page is adding new content or modifying styles? It depends on the site, but a good rule of thumb is to batch updates.

##Paint

###Putting pixels on the page
Let's review the step order of rendering an HTML document.

1. Begin constructing the DOM by parsing the HTML.
2. Request CSS and JS resources.
3. Parse CSS and construct the CSSOM tree (this needs to be done ASAP to unblock the JS engine).
4. Execute JS.
5. Merge DOM and CSSOM into the Render tree.
6. Run layout, paint

##Devtools

###You can't optimize what you can't measure
You can use [CSS minification](https://developers.google.com/speed/pagespeed/service/MinifyCSS) and [inlining and prioritizing "critical CSS"](https://developers.google.com/speed/pagespeed/service/PrioritizeCriticalCss) to optimize performance.


#Optimizing the CRP

##Optimizing the DOM
###Fewer bytes...faster renders

- Learn more about [resource minification](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/optimize-encoding-and-transfer#minification-preprocessing--context-specific-optimizations)
- Learn more about [text compression with gzip](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/optimize-encoding-and-transfer#text-compression-with-gzip)
- Learn more about [HTTP caching](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching)

We focus on optimizing the DOM and CCSOM (the steps that come before the Render tree) because these are usually the worst performance offenders. For the DOM, we want to minimize the size of the HTML file aka minification. Once that's done you want to compress the file and cache it.

While these are great techniques to start with first, remember that CSS blocks rendering. So another optimization strategy is to avoid Rendering Blocking CSS. Since CSS is a render blocking resource, get it down to the client as soon as as quickly as possible to optimize the time to first render.

So why does CSS block rendering? That is because without the CSSOM fully loaded, the browser cannot fully render a page.

##Unblock CSS With Media Queries
CSS allows us to scope styles to particular conditions, using media queries. Thus sometimes it is useful to split CSS into multiple files in order to not have to block rendering. For example in a situation like so:
	<link rel="stylesheet" href="style.css">
	<link rel="stylesheet" href="style-print.css" media=print>
The browser still downloads both stylesheets, but the second sheet does not block rendering of the page. This improves performance in two ways: (1) the second file does not have to block rendering, and (2) we've got fewer bytes to wait for before we can render the page. Again, you can read more about media queries from [Responsive Web Design Basics](https://developers.google.com/web/fundamentals/layouts/rwd-fundamentals/).

#Optimizing Javascript

##All the things that block rendering.
Minify, compress, and cache it. However, Javascript is parser blocking because it blocks DOM construction when the browser hits the `<script>` tag. The browser blocks the DOM from being fully constructed while waiting for the script file to be downloaded. This is particularly bad when your `<script>` points to another Javascript file, which causes the browser to have to fetch the file and blocking construction of the DOM - it takes a while. So why not just use inline Javascript throughout your code? Well, there are tradeoffs. Inlining Javascript may help limit requests, but the downside is having redundant code.

###External Javascript Dependencies
Consider an example like below:

	<style src="style.css" /> <!-- p {color: black}
	<p>
		Awesome page 
		<script>
			var e = document.getElementsByTagName("p")[0];
			e.style.color = "red";
		</script>
		is awesome
	</p>
The browser requests the HTML. As soon as it gets the response it starts building the DOM. It then discovers the CSS and sends the request. Then the parser continues and finds the `<script>` tag, at which point it has to block. It doesn't know what the script is going to do, because the script may want to try accessing the CSS properties, so it blocks script execution until the CSS arrives and builds the CSSOM. Only then can we run the Javascript and then finish building the DOM.

CSS blocks rendering and blocks Javascript execution, which is why it is crucial to optimize your CSS.

###Async Javascript
Notice that some scripts don't modify the DOM and, really they shouldn't be blocking rendering. Analytics is a great example. We have two strategies to tell the browser exactly that. 

One strategy is to load the script after the page has loaded. The browser fires an onload event (see `window.onload` event) when the page is finished loading. You can wait for that and then execute your script then.

The second strategy is simpler using the `async` attribute to the script tag. It has two properties: 
- it does not block DOM construction
- it does not block **on** CSSOM
Async does not block the CPR. But it doesn't work for inline scripts which always block on the CSSOM. Exception is when you run the script before the CSS.

###Bonus - TCP Slow Start
See the formula that models how long it takes to download a file given the initial CWND, packet size, and roundtrip time (RTT) at [the secotion on TCP slow-start](http://chimera.labs.oreilly.com/books/1230000000545/ch02.html#SLOW_START).

#General Strategies
##Recap
###Minify, Compress, Cache
- HTML, CSS, Javascript

###Minimize use of **render blocking** resources (CSS)
1. Use media queries on <link> to unblock rendering
2. Inline CSS

###Minimize use of **parser blocking** resources (JS)
1. Defer Javascript execution
2. Use `async` attribute on `<script>`

All these ideas is all about reducing the amount of data that needs to get sent down the wire - **minimize bytes**. Furthermore, the we want to **reduce critical resources**. Finally, we want to **shorten the CRP length**.

###Note
PageSpeed Insights encourage having small HTML file sizes like < 14kb in order to reduce the number of roundtrips it takes to get HTML responses. This is low level detail which you can learn more from reading the *High Performance Browser Networking* book.
Keeping your eyes on CRP metrics will help you optimize performance

- number of critical resources
- total critical KB
- minimum critical path length (roudtrips)

The browser can download both the CSS and Javascript in parallel

##Strategies
1. Inline CSS and inline JS.
2. Media query for CSS and async JS.

##The Preload Scanner
Read [How the Browser Pre-loader Makes Pages Load Faster](http://andydavies.me/blog/2013/10/22/how-the-browser-pre-loader-makes-pages-load-faster/)
The preload scanner in the browser peeks ahead in the document and tries to discover critical CSS and JS files and requests them while the parser is blocked.



# Sources
- [Udacity Wiki on Perf Resources](https://www.udacity.com/wiki/ud884)
- [Chrome Device Mode](https://developer.chrome.com/devtools/docs/device-mode)
- [Android Remote Debugging](https://developer.chrome.com/devtools/docs/remote-debugging)
- [Modern Web Development](https://developers.google.com/web/fundamentals/)
- [ChunkScatter](http://blog.cowchimp.com/chunk-scatter-http-chunked-response-analysis-tool/)
- [CSSOM Construction](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/constructing-the-object-model#css-object-model-cssom)
- [CSS Blocks Rendering](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-blocking-css)
-[Critical Rendering Path Perf Patterns](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/analyzing-crp#performance-patterns)
-[PageSpeed mobile analysis - less than 1 second rendering](https://developers.google.com/speed/docs/insights/mobile)
-[TCP Slow Start from "High Performance Browser Networking"](http://chimera.labs.oreilly.com/books/1230000000545/index.html) 
