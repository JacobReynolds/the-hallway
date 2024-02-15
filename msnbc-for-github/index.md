---
title: msnbc for github
layout: default
nav_order: 8
---

# 📰 msnbc for github

I'm in love with technology. Almost pervertedly so. Not in a furry kind of way, but more of a getting married to a roller coaster kind of way. Few things in life excite me as much as technology. Finding new tools to play with, buying new hardware, writing code, solving problems, 😍. It's something I've been lucky to love in life and have the opportunity to do for a living. When thinking about start up ideas, tapping into this love is something that I commonly come back to. How can I build a platform that keeps me on the cutting edge of techology. Should I start a newsletter about emerging tech, create a youtube channel trying out all the latest frameworks, or provide training to enterprises to help them adopt new methodologies? So far nothing has stuck, but this is about one idea I pursued.

## Morning routine

My morning routine consists of many weird little things, like always having to put on deodorant before I brush my teeth. But I do that so it has time to dry before I have to put a shirt on. Anyways, once I'm clothed and in my office I check the front page of [Hacker News](https://news.ycombinator.com), the [Github Trending](https://github.com/trending) page, and unfortunately [Twitter](https://x.com/jreynoldsdev) for a little bit. One thing I'm routinely disappointed by is that the Github Trending page only updates once a day and provides a max of 25 results. My insatiable thirst for more technology is always left unquenched. So I figured, why not try and build something around this?

## Here's the pitch

A platform, a world wide web technology platform you could even say, that tracks all of the available code repositories on the internet (not just Github), aggregates statistics about them and reports them to you. This way you can get a far more in-depth trending page and more interestingly try and draw intelligence about the interactions between these repositories, to answer questions like:

- What is the most common date picker library used by React developers?
- How many apps have been rewritten from React to Vue?
- What dirty little secrets are those C developers hiding?

There are various iterations like this out there that aggregate stargazers and commits and let you search through them, but none quite captured my vision.

## How does it work

So Github definitely doesn't like people using their API too heavily, they famously don't have a pay-for-use API plan, and have nice but [non-suitable API limits](https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api?apiVersion=2022-11-28) for projects like this. That can be a future problem though, what's a small test case I can try out before we go whole hog on this? How about scraping all of the available `package.json` files from Github, so we can try and correlate dependency usage across projects. Their [Code Search API](https://docs.github.com/en/rest/search/search?apiVersion=2022-11-28#search-code) should be good for this. If we use large page sizes and take our time, rate limits shouldn't be a problem. Some Typescript along the lines of:

```typescript
type SearchResponse<T> = {
  total_count: number;
  incomplete_results: boolean;
  items: T[];
};

type CodeSearchMatch = {
  name: string;
  path: string;
  sha: string;
  url: string;
  git_url: string;
  html_url: string;
  repository: any; //`any` for brevity
  score: number;
};

const GITHUB_API_BASE_URL = "https://api.github.com";

async function codeSearchForPackageJson(page: number, pageSize: number) {
  let out = [];
  try {
    const response = await axios.get<SearchResponse<CodeSearchMatch>>(
      `${GITHUB_API_BASE_URL}/search/code?q=${encodeURIComponent(
        "filename:package.json extension:json"
      )}&sort=stars&per_page=${pageSize}&page=${page}`
    );
    console.log(`Code search returned ${response.data.items.length} matches`);

    for (const file of response.data.items) {
      out.push({
        ownerName: file.repository.owner.login,
        repoName: file.repository.name,
        fileUrl: file.html_url
          .replace("/blob", "")
          .replace("github.com", "raw.githubusercontent.com"),
      });
    }
  } catch (error: any) {
    console.error("Error fetching repositories:", error?.response?.data);
    return [];
  }
  return out;
}
```

and you can start iterating through all `package.json` files on Github, in descending order of popularity. Deeper in the code I parse out all of the dependencies and goodness but this was the hard part. Fun fact, getting files from `raw.githubusercontent.com` doesn't count against your rate limit, which is the reason for that substitution. Since I solved that, I should go ahead and build everything else! I went through building out some dependency parsing logic, stuffing it into a DB and displaying it in a UI. Only for me to discover, 2 days later, this note in their docs

> the GitHub REST API provides up to 1,000 results for each search

which I didn't think would be a problem because my search showed millions of results in the total count parameter. Turns out, if you try to paginate pass 1k results, it throws an error...So we're a bit fucked on that one.

## Eminem? No NPM

Despite my issues, I know there's a lot of large websites out there that aggregate results from Github, how do they do it? Likely enterprise agreements, but maybe there's hope. Since we're working with the javascript ecosystem right now, even though future plans include multiple languages, let's try NPM. Maybe they have a nicer API.

As it turns out, they are more liberal with a lot of their data, and if you look hard enough you can find their API documentation in a random [GitHub repo](https://github.com/npm/registry/blob/master/docs/REGISTRY-API.md). Even more weirdly, is a file in that repo called [`follower.md`](https://github.com/npm/registry/blob/master/docs/follower.md). This details a mechanism for creating a NPM registry follower, that will sync all data in their entire database to a remote follower that you own. Holy shit! I've been a good boy, but I don't deserve such luxury.

This follower will sync NPM's CouchDB replica at [https://replicate.npmjs.com](https://replicate.npmjs.com) and stream it from the beginning of time. A lot of this went over my head, as the data it streamed contained various events across the DB, all sorts of different fields, and a lack of documentation for my small brain. Nevertheless, it was pretty interesting to see, I was able to get a lot of the information I wanted, primarily dependencies, and even get them across versions. So I solved this for the Javascript ecosystem at least, but surprisingly life does exist outside of JS.

## Thanks for all the fish

Sadly the issues with Github's API were a big issue and would prevent me from expanding a lot of this research to other languages. I could try going the same route with other package managers for the other languages, but I lost steam and decided to move on to some other ideas. How's that for anticlimactic? 🫡
