---
layout: post
title: "Bookmarklets Made My Life Easier"
excerpt: "Using tiny pieces of script to speed up your browser usage!"
modified: 2016-12-10T14:17:25-04:00
categories: tips
tags: [javascript, browser]
comments: true
share: true
---

### Update
I've put the bookmarklets that I mostly frequently use in every day life on [this repository](https://github.com/scionofbytes/bookmarklets).

## Being a Developer is Fun

One of the advantages of browsing the web as a web-developer vs. as a layman is the fact that you get to do some pretty cool stuff to make your life easier.

For example, I remember once being on a website where a certain action kept failing with no visible errors. I’d click a button and nothing would happen. It was frustrating and a layman would have been forced to quit at that point or write in to the website’s maintainers. 

And that's what I did too. I wrote in to the site's maintainers.

But before I did that I fired up the [developer console on Chrome](https://developers.google.com/web/tools/chrome-devtools/console/) and started debugging the issue. It turned out to be an error in the script that was trying to handle the button's click event. The event registered to the handler had a typo in the name.

Since I could fix the code on the client, I did so and moved on with my life. I never returned to the site and the developers never wrote back. I hope they found the bug-report useful and it helped better their site.

> Where do bookmarklets come in?

One of the things I take advantage of most are [bookmarklets](http://www.bookmarklets.com/). These are tiny pieces of javascript code that run in the context of whatever page you’re on and have full access to the page’s DOM. They live in the form of a clickable bookmark on your toolbar and do their thing at the click of a mouse button. You can even configure them to ask for input before taking actions making them incredibly versatile.

> So how have you taken advantage of bookmarklets?

### Running Queries on Rockmongo. 

Anyone who has used the tool [Rockmongo](http://rockmongo.com/) knows how irritating it can be to click through multiple pages to get to a collection’s query interface and then run the query. But once I realized that this action could be represented as a query string, I quickly scripted a little querybuilder that would prompt me for an [_id](https://docs.mongodb.com/manual/core/document/#document-id-field) and would perform a lookup against the relevant collection. This improved my productivity a *lot*.

> Show me what it looks like.

Here's the code for one of the Rockmongo query builders:

```javascript
javascript:(function getDocumentById() {
	window.location.href = encodeURI("http://<host>/rockmongo/index.php?db=<db_name>&collection=<CollectionName>&action=collection.index&format=json&criteria=%7B%0D%0A%09_id%3A+ObjectId%28%27" +
	prompt() +
	"%27%29%0D%0A%7D&newobj=%7B%0D%0A%09%27%24set%27%3A+%7B%0D%0A%09%09%2F%2Fyour+attributes%0D%0A%09%7D%0D%0A%7D&field%5B%5D=_id&order%5B%5D=desc&field%5B%5D=&order%5B%5D=asc&field%5B%5D=&order%5B%5D=asc&field%5B%5D=&order%5B%5D=asc&limit=0&pagesize=10&command=findAll");
})();
```

It looks quite messy, unfortunately, but it gets the job done. Just replace the stuff in angle brackets with your own values and this should be up and running (if it isn't, [let me know](mailto:shuvophoenix@gmail.com)).

### Getting Email Records on the Mandrill Dashboard

Another example would be of querying in [Mandrill](http://www.mandrill.com/). If you've used [Mailchimp](https://mailchimp.com/)'s Mandrill service, you'll know just how frustrating it can be to log in to the Mandrill dashboard and query for certain tags on certain dates. The interface is clunky and slow and you get logged out often for no reason.

The only way I could maintain my sanity was to use a handy little bookmarklet to quickly run the queries that I most regularly accessed.

```javascript
javascript:(function getTodaysTag() {
	function getToday() { 
	  return formatDate(new Date());
	}

	function formatDate(date) {
	  return (date.getMonth() + 1) + '/' + date.getDate() + '/' + date.getFullYear();  
	}

	window.location.href = 
		"https://mandrillapp.com/activity?date_format=mm%2Fdd%2Fyy&q=&date_range=custom&start_date=" + 
		encodeURIComponent(getToday()) + 
		"&stop_date=" + 
		encodeURIComponent(getToday()) + 
		"&tag=" +
		"<your-tag>" +
		"&sender=&search-select-q=&api_key=&__csrf_token=<your-token>";
})();
```

The above script builds a query string that would fetch for you all the emails tagged with a particular tag (which you can also accept dynamically by calling [`prompt()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/prompt)), that were sent on the current date. You don't have to click through multiple boxes, you just click on one little bookmarklet.

One caveat is that these two bookmarklets work by building querystrings which are then accessed like any normal page. This only works if you're already logged in to Rockmongo or Mandrill. If you're not, you'll be redirected to the login page, after which you'll come to the dashboard's standard homepage and will have to click on the bookmarklet again. Still better than the alternative.

> That sounds pretty cool but what else can I do? I don't use Mandrill or Rockmongo.

These are just two examples of what you can do with bookmarklets and that's not the end of it. Tumblr's [Post to Tumblr](https://www.tumblr.com/docs/en/lesser_known_features) bookmarklet let's you quickly share the contents of any page to your tumblr blog.

Similarly many other services use bookmarklets to for sharing content or simply providing better readability. [Hongkiat](www.hongkiat.com) has this [beautiful list of bookmarklets](http://www.hongkiat.com/blog/100-useful-bookmarklets-for-better-productivity-ultimate-list/) that you can copy to your bookmarks bar and start using immediately. And since they are just pieces of javascript code, you can customize them as much as you want and have your own version of the bookmarklet tailored to your needs.

So there you go, the power of bookmarklets. I hope this will help you become more productive in your everyday browsing or at least alleviate some of the pains. If you have any questions, send me an [email](mailto:shuvophoenix@gmail.com) or drop me a [tweet](https://twitter.com/scionofbytes).
