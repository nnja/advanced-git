# Advanced Git
## Final Exercise - Using the GitHub API


### Overview

In this exercise, we'll use the GitHub REST API to get information about ourselves, our repositories, and other repositories on GitHub.

**Note:** GitHub allows 60 unauthenticated requests per hour. If you hit your limit, and would like to keep exploring proceed to **Bonus Exercise 1** to learn how to create an API token. 

In these examples I'll be using `cURL`, which is a command line tool that allows you to make requests to a server from the terminal.

If your skill-set is more advanced, you can try a developer library for your language instead. There are GitHub developer libraries for many languages,  such as python, javascript, PHP, and more! [GitHub Developer Libraries](https://developer.github.com/v3/libraries/)

If you're unsure about which integration to use, try the exercises using `cURL` first.

This GitHub API has many uses, from getting simple profile and repository information to creating Issues programatically. 

### Prerequisite - Learn to make unauthenticated requests

We can use the GitHub API unauthenticated for many operations, but GitHub limits us to 60 requests an hour.

To get information about my user profile without authenticating, run:

```
curl https://api.github.com/users/nnja
```

Try it with your username and check out the results.
This corresponds to the [Users endpoint](https://developer.github.com/v3/users/).

### Exercise

Using the [GitHub API documentation](https://developer.github.com/v3/) answer the following questions:

1. How many GitHub users are you following?
2. Which of your repositories has the most stars?
3. What languages are present in your favorite repository?

Hints:

1. [Get a single user](https://developer.github.com/v3/users/#get-a-single-user)
2. [Search Repositories](https://developer.github.com/v3/search/#search-repositories). Stars are in the `stargazer_count`, pass in `stars` as the `sort` parameter. 
3. [List languages for a Repository](https://developer.github.com/v3/repos/#list-languages)

## Solutions

### Step 1 - Get a Github User
Let's query the Github single user API endpoint to see how many people we're following. Replace the username with your Github username to see personalized results.

```
$> curl "https://api.github.com/users/nnja"
{
  "login": "nnja",
  "id": 2030983,
  "avatar_url": "https://avatars1.githubusercontent.com/u/2030983?v=4",
  "gravatar_id": "",
  "url": "https://api.github.com/users/nnja",
  "html_url": "https://github.com/nnja",
  "followers_url": "https://api.github.com/users/nnja/followers",
  "following_url": "https://api.github.com/users/nnja/following{/other_user}",
  "gists_url": "https://api.github.com/users/nnja/gists{/gist_id}",
  "starred_url": "https://api.github.com/users/nnja/starred{/owner}{/repo}",
  "subscriptions_url": "https://api.github.com/users/nnja/subscriptions",
  "organizations_url": "https://api.github.com/users/nnja/orgs",
  "repos_url": "https://api.github.com/users/nnja/repos",
  "events_url": "https://api.github.com/users/nnja/events{/privacy}",
  "received_events_url": "https://api.github.com/users/nnja/received_events",
  "type": "User",
  "site_admin": false,
  "name": "Nina Zakharenko",
  "company": null,
  "blog": "nnja.io",
  "location": "Portland, OR",
  "email": null,
  "hireable": null,
  "bio": "Developer, pythonista, & speaker.\r\nTeam emacs.\r\nCurrently @venmo, previously @reddit & @recursecenter",
  "public_repos": 46,
  "public_gists": 21,
  "followers": 208,
  "following": 69,
  "created_at": "2012-07-24T01:53:42Z",
  "updated_at": "2017-08-27T09:43:19Z"
}
```
It looks like I'm following 69 other users.

### Step 2 - What's Our Most Popular Repo?
We'll query the `/search/repositories` endpoint to find out our most starred repo. We want to search by user (`q=user:nnja`), sort by number of stars (`sort=stars`), and for simplicity we'll set `per_page=1` so that we only get the top result. This is a large response so we'll just grep for the line we're looking for:

```
$> curl -s "https://api.github.com/search/repositories?q=user:nnja&sort=stars&per_page=1" | grep "stargazers_count"
      "stargazers_count": 114,
```

### Step 3 - What Languages are in your Favorite Repo?
Github has a handy List Languages endpoint that will return, for a given repo, the number of bytes of code written in any number of languages:

```
$> curl "https://api.github.com/repos/nodejs/node/languages"
{
  "JavaScript": 5678164,
  "C++": 2072649,
  "C": 376581,
  "HTML": 163390,
  "POV-Ray SDL": 88241,
  "Python": 85037,
  "DTrace": 37659,
  "Makefile": 33997,
  "Batchfile": 23763,
  "Roff": 15312,
  "R": 5359,
  "Shell": 1944
}

```
Surprise, surprise - JavaScript makes up a large majority of the `nodejs` repo.

### Additional Resources
We've just scratched the surface of what the GitHub API is capable of.

If you have extra time, continue to the bonus exercises below.

If you want a deeper dive into creating real-world GitHub API applications, review the [Development Guides](https://developer.github.com/v3/guides/).

### Bonus Exercise 1

Using the GitHub API with a personal access token provides a few benefits:

 - Your rate limit is bumped to 5,000 requests an hour, instead of 60.
 - Tokens are revokable. If your token is compromised, you can cancel it.
 - Tokens can be given limited scope, to only perform the operations you explicitly authorize them for.

**Note:** An GitHub personal token is a password! Don't share it with anyone or commit it to a public repository. When you're done using your personal token for testing, a best practice is to revoke it when you're done.

The GitHub API also supports oAuth, for authentication via applications. 

**Follow these steps to create a personal access token:**

 1. Visit [https://github.com/settings/tokens](https://github.com/settings/tokens)
 2. Click Generate a New Token
 3. Enter a token description
 4. Select the 'repo' scope, leave all others unchecked. 
 5. Store the access code in a safe place, you won't be able to retrieve it again. If you lose it, you'll have to generate a new one.
 

**To use an access token to make CURL Requests:**

Running the following command with a personal token will return the user details for the authenticated user. 
 
```
curl -i -H 'Authorization: token <YOUR_TOKEN>' \
    https://api.github.com/user
```

### Bonus Exercise 2

If you're a pro at using APIs, and know basic HTML and CSS, you can give the last bonus exercise a try.

Create a simple webpage that displays your git profile information, as well as some simple statistics.

Include:

 - Your GitHub profile picture
 - Your GitHub bio
 - The amount of followers you have
 - Your most 'starred' repo
 