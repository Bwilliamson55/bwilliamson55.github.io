---
title: Extract GraphQL calls from a browser for debug in Postman
date: 2023-03-05 19:00
categories: [General, Dev]
author: bwilliamson
tags: [api, graphql, postman, general]
---

>TLDR: You can right-click->copy as curl right from devtools, and paste into postman for on the spot debugging!

*For this all you need is a browser with devtools, [Postman](https://www.postman.com/downloads/), and a GraphQL call.*

# So,

you have a moody or misbehaving GraphQL call in one of your web apps, and you wish you could just change ***one thing*** about the call, on the fly, just to see what happens.
Or maybe the call really does need to be debugged and reworked in a major way but you want to test it first.
Or maybe you just want a faster way to get the call in front of you.

So let's use this little tool to emulate an app making a GraphQL call: [https://lucasconstantino.github.io/graphiql-online/](https://lucasconstantino.github.io/graphiql-online/)

Here's my query:
```js
query($filter: CountryFilterInput) {
  countries(filter: $filter) {
    name
    code
    phone
    continent {
      code
      name
    }
    capital
  }
}
```
Variables:
```js
{
  "filter": {"code": {"in": ["CA","US","GB"]}}
}
```

# Copy the query from a browser

Open your browsers dev-tools, usually you can right click anywhere on the page and select `Inspect`.
Click over to the `Network` tab of the tools,
Filter if you like, for Fetch/XHR requests,
Then run the query to see the call as shown in the screenshot below.

![01](/assets/img/post%20images/general/03052023/01-graphql-online.png){: .normal }

Copy the call with a few clicks:

![02](/assets/img/post%20images/general/03052023/02-copy-as-curl.png){: .normal }

# Paste into Postman via Import

Over in Postman, find and click your `Import` button:

![03](/assets/img/post%20images/general/03052023/03-postman-import.png){: .normal }

Paste your call via the `Raw text` tab:

![04](/assets/img/post%20images/general/03052023/04-raw-text-paste.png){: .normal }

![05](/assets/img/post%20images/general/03052023/05-paste-continue.png){: .normal }

Confirm the import - the defaults are fine:

![06](/assets/img/post%20images/general/03052023/06-confirm-import.png){: .normal }

# Click Send

Magic!
Now we can play around with the variables and other parameters of the query after only a dozen clicks!
![07](/assets/img/post%20images/general/03052023/07-postman-body-tab-send.png){: .normal }

That's all for this one, I hope you found this interesting!
