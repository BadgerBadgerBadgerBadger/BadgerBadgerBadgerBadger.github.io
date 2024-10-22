My girlfriend works in the field of language and education research and sometimes makes lessons for the online Dutch language learning platform [Nedbox](https://www.nedbox.be/). The lessons are often based around real world events and having a way to _gather interesting real world events is invaluable. She asked me for help listing news events from the show [VRT NWS journaal](https://www.vrt.be/vrtmax/a-z/vrt-nws-journaal) on [VRT Max](https://www.vrt.be/vrtmax/).

Their page looks like this.
![image](https://github.com/user-attachments/assets/9e969878-fa0c-4748-ae1e-97703f0c0bfe)

Each day there are two episodes, one for the 13h news and another for the 19h news. Clicking through, we can see that each episode has chapters within it with timestamps and titles. It's a list of these titles with links to that chapter (for quick viewing) that my girlfriend was interested in gathering, without having to manually click on each episode, each day, every day.

![vrt-news-journaal-episode-page](https://github.com/user-attachments/assets/d5fe96b9-dadf-42ed-8784-0089f95ea96c)

I started off by exploring the html source of the landing page to find content already present. There's no need for complicated scraping if I can just grab the relevant html section, perhaps something between `<ul>` or `<ol>` tags.

No such luck. The initial document that loads is pretty barebones.

```html
<!DOCTYPE html>
<html lang="nl">

	<head>
		<meta charset="UTF-8">
		<title>VRT MAX</title>
		<script>var s = document.createElement('script'); s.src = '/vrtnu-resources/loaders/scriptLoader.js'; s.async = false; document.head.appendChild(s);</script>
	</head>
	<body>
	
		<div id="root"></div><noscript>U moet JavaScript inschakelen om deze app te gebruiken.</noscript>
		<script>document.body.onload = () => loadBodyScripts();</script>
	</body>
</html>
```

I have elided much from the head but the body is as you see it: sparse. It looks as if all the actual content is loaded via Javascript. Not the best design, if you ask me. I'm not sure there is enough interactivity on a page like this to justify denying non-Javascript enabled users access to the page.

Okay, so we can't do anything without running Javascript. So let's do something to run Javascript.

I originally wanted to program this in Go since that is my daily driver. But after exploring libraries, I couldn't find anything that beats [puppeteer](https://github.com/puppeteer/puppeteer) for the sheer number of features available, not to mention ease of use, wealth of documentation, and example projects.

 I started off by inspecting the page after all the Javascript had run and found my content.
![vrt-news-journaal-episode-page-html](https://github.com/user-attachments/assets/b3263ca6-8831-4dda-b424-d72e92eda37e)

After playing around a bit I decided on this CSS selector: `#main ul`. Short and gets the job done. Time to extract the list.
```javascript
async function extractEpisodeList(url) {

	try {
		// Launch the browser
		const browser = await puppeteer.launch();
		
		// Create a new page
		const page = await browser.newPage();
		
		// Navigate to the specified URL and wait for the network to be idle
		await page.goto(url, { waitUntil: 'networkidle0' });
		
		// Wait for any client-side rendering to complete
		await new Promise(resolve => setTimeout(resolve, 2000));
		
		// Get the fully rendered HTML content
		const content = await page.content();
		
		// Extract the list items and their links
		const listItems = await page.evaluate(() => {
		
			const list = document.querySelector('#main ul');
			
			if (!list) {
				return [];
			}
		
			return Array.from(list.querySelectorAll('li'))
				.map(li => {
			
					const a = li.querySelector('a');
	
					return a ? {				
						href: a.getAttribute('href'),
						content: a.getAttribute('aria-label') || a.textContent.trim()
					} : null;
		
				})
				.filter(item => item !== null);
		});
		
		// Close the browser
		await browser.close();
		return listItems;
		
	} catch (error) {
		throw error;
	}
}
```

The code should be pretty self-explanatory. I had to fiddle a bit to find the right query selectors to extract the link and title after I got the list. It's important we use [`waitUntil`](https://pptr.dev/api/puppeteer.waitforoptions) when loading the page to ensure _most_ network activity is complete. [`networkidle0`](https://pptr.dev/api/puppeteer.puppeteerlifecycleevent) waits for at least 500 ms of no new network connections.

The end result is a list of episodes.
```json
[
	{
		"href": "...",
		"content": "..."
	}
]
```

> If you spend some time on that page you'll notice the episode list is a sort of carousel within which new episodes get loaded as you scroll horizontally to the right. By default only 13 episodes or so are listed. But this is fine for us since we will check this page every day and we only care about new episodes.

Before we start grabbing the chapters for each episode, let's talk about a very small optimization. We _could_ scrape the entire list of episodes every day, but given that we have only 2 new episodes each day, we could be re-scraping 11 episodes we have already scraped, every single day. Let's perform a very small optimization step.

We will maintain a list of episodes we have already processed, and each day process only new episodes. This is achieved by this function (which itself calls the function above to grab the episode list).

```javascript
async function updateEpisodeList() {

	const url = 'https://www.vrt.be/vrtmax/a-z/vrt-nws-journaal/';
	
	try {
		const renderedContent = await extractEpisodeList(url);
		
		// Read the local file "episode-list.json"
		let localEpisodes;
		
		try {
		
			const rawData = await fs.readFile(episodeListFilePath, 'utf8');
			localEpisodes = JSON.parse(rawData);
		} catch (error) {
		
			console.error('Error reading or parsing local file:', error);
			return;
		}
		
		// Compare extracted episodes with local episodes
		const extractedSet = new Set(renderedContent.map(item => item.href));
		const localSet = new Set(localEpisodes.map(item => item.href));
		
		const newEpisodes = renderedContent.filter(item => !localSet.has(item.href));
		
		const removedEpisodes = localEpisodes.filter(item => !extractedSet.has(item.href));
		
		// Print the diff
		console.log('Diff between extracted and local episodes:');
		
		if (newEpisodes.length > 0) {
		
			console.log('\nNew episodes:');
			
			newEpisodes.forEach(episode => {
				console.log(`- ${episode.content} (${episode.href})`);
			});
		}
		
		if (removedEpisodes.length > 0) {
		
			console.log('\nRemoved episodes:');
			
			removedEpisodes.forEach(episode => {
				console.log(`- ${episode.content} (${episode.href})`);
			});
		}
		
		if (newEpisodes.length === 0 && removedEpisodes.length === 0) {
			console.log('No differences found.');
		}
		
		return {
			newEpisodes: newEpisodes,
			removedEpisodes: removedEpisodes,
			localEpisodes: localEpisodes
		};
	} catch (error) {
		throw error
	}
}
```

Once again, most of the code should be self-explanatory. We read our existing episodes list from a file, compare against the scraped list and generate lists of new episodes (and some extra stuff but those aren't relevant).

Armed with new episodes to scrape, we need to figure how to extract the chapters of the episodes, mainly their title and the timestamp at which they start. And if we can generate a link that will take you to the exact part of the video, even better.

For this I started inspecting the episode page itself. As with the main page, the episode page is sparse and uses JS to load everything, but looking at the XHR calls, I noticed something.

![vrt-nws-journaal-graphql](https://github.com/user-attachments/assets/a7b42447-d434-4dcd-8d12-146f8fdd7f8a)

Is that a Graphql endpoint mine eyes spy?! I definitely know how to use one of those.

This made the whole process much simpler. I inspected the Graphql response, found the part with the episode chapters and the timstamps and bundled them up in a message I could send to a Slack channel. I won't show that code here as it's fairly mundane.

> Afterwards I had the thought that perhaps the main page also loaded its episode list via a Graphql call and I did not need to scrape anything. But I already had so no point thinking about it further.

At this point I had to decide where to run it. This wasn't planned to be a live service, just a script that ran every day so something like Fly.io wasn't necessary. I remembered a tech meetup I had attended earlier in the year at [Theo Technologies](https://www.theoplayer.com/) and the talk on [Zero Budget Solution Engineering Toolbox](https://youtu.be/kabFMYS0TxY?t=3760) by [Daniel Dallos](https://www.linkedin.com/in/danieldallos/). He had talked about using Github actions as a way to run jobs on a schedule, while at the same time storing small amounts of data in the Github repository as a JSON file. I highly recommend watching the talk as he mentions a lot of useful tricks and tools.

So I whipped up a Github a workflow yaml file.

```yaml
name: Scrape  
  
on:  
  workflow_dispatch:  # this allows us to manually run this workflow
  schedule:  
    # Runs at 12:00 UTC every day  
    - cron: '0 12 * * *'  
  
jobs:  
  run-script:  
    runs-on: ubuntu-latest  
  
    steps:  
      # Step 1: Check out the repository  
      - name: Checkout repository  
        uses: actions/checkout@v3  
  
      # Step 2: Set up Node.js (adjust version as needed)  
      - name: Set up Node.js  
        uses: actions/setup-node@v3  
        with:  
          node-version: '20'  
  
      # Step 3: Cache npm dependencies  
      - name: Cache dependencies  
        uses: actions/cache@v3  
        with:  
          path: ~/.npm  
          key: ${{ runner.OS }}-node-${{ hashFiles('**/package-lock.json') }}  
          restore-keys: |  
            ${{ runner.OS }}-node-  
  
      # Step 4: Install dependencies (if you have a package.json)  
      - name: Install dependencies  
        run: npm ci  
  
      # Step 5: Run your JavaScript script  
      - name: Run the JavaScript script  
        run: node scrape.js  
  
      # Step 6: Commit and push changes back to the repository  
      - name: Commit and push changes  
        run: |  
          git config --global user.name 'github-actions[bot]'  
          git config --global user.email 'github-actions[bot]@users.noreply.github.com'  
          git add .  
          git diff --quiet && git diff --staged --quiet || (git commit -m "Update results from scheduled script" && git push)  
        env:  
          # Ensure you have write access to the repository via a GitHub token  
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

There is some stuff in there about loading NodeJS, cacheing dependencies and whatnot, but the meat of the code is in Steps 5 and 6 where we run our scraping and then commit the results back to the repository.

And with that I had a tool that would let me know every day about the episodes of the VRT NWS Journaal.

![vrt-nws-journaal-slack-message](https://github.com/user-attachments/assets/90a280d1-6eb9-4ab6-9d6d-23167fcf8250)

Hope you enjoyed the read.
