---
title: Migrate and Sync Craft CMS data in minutes with n8n
date: 2023-02-26 11:00
categories: [ETL, n8n]
author: bwilliamson
tags: [n8n, etl, videos, api, craftcms, feedme]
---
Using

- Craft CMS
	- Built in Graphql explorer
	- Feed-Me Craft Plugin
- n8n

We can move almost any data between environments, public or not, in just minutes.

Recently I had some tasks at work that inspired this.

Primarily - How do you sync entries between stage and prod, or stage and QA, when stage is behind an IP restricted firewall? In other words- I need to be on a vpn to use this staging environments control panel. So how the heck will the other environments get its data? ***Export/Import? Eew.***


# The Plan


There's actually a few plans:
1. Use [`graphiQl`](https://craftcms.com/docs/4.x/graphql.html#using-the-graphiql-ide) (note the i) to manually get some data and echo it to the other environments with n8n
2. OR If we can publicly access the data source- then use graphql to access it directly and echo it to other environments with n8n
Then,
1. Use feed-me on the receiving end to digest the JSON feed
2. Profit


## Plan 1 (Indirect)
---


Go to your source environment, and go to the GraphiQl explorer (YourDomain/admin/graphiql).
From here- use the 'explorer' to find the data you're after.

### A little about the GraphiQL explorer


Linkie - [https://craftcms.com/docs/4.x/graphql.html#using-the-graphiql-ide](https://craftcms.com/docs/4.x/graphql.html#using-the-graphiql-ide)

I'm sure you're capable of reading the docs on this, but I'll point out a few things:

#### Explorer side panel

By clicking the 'Explorer' button we can see our schema options in a click adventure:

![01](/assets/img/post%20images/etl/n8n/20230222/01-graphiql.png){: .normal }

In the Explorer we have a few adventure types- Query and Mutation. Using the bottom left dropdown we can start a new type in the ISE area:

![02](/assets/img/post%20images/etl/n8n/20230222/02-graphiql.png){: .normal }


The purple things are `filters`, the blue things are `return values`, the yellow are `types`


For example, here's a query with a few common filters and return values for our greyhound org:

![03](/assets/img/post%20images/etl/n8n/20230222/03-graphiql.png){: .normal }


You can create what's shown here with just clicking in the Explorer panel, followed by clicking the play button.

#### Query panel

You can also just start typing in the query panel, inside a valid `query` or `mutation`, and it will do a lot of work for you. For instance, `CTRL+Space` or `CMD+Space` gives you auto-complete options:

![04](/assets/img/post%20images/etl/n8n/20230222/04-graphiql.png){: .normal }


If you enter something that's invalid, it generally just doesn't show up in the results:

![05](/assets/img/post%20images/etl/n8n/20230222/05-graphiql.png){: .normal }

Refer to the documentation for more information and guidance: https://craftcms.com/docs/4.x/graphql.html#getting-started


### Get this data into n8n

Plan 1, which is what I did at work recently in a pinch, is to simply copy the resulting `JSON` into n8n.

Start by placing a default code node: (The code in here is all getting removed)

![06](/assets/img/post%20images/etl/n8n/20230222/06-graphiql.png){: .normal }


Copy the raw output from your GraphiQL results, and modify the code node to return that data:

![07](/assets/img/post%20images/etl/n8n/20230222/07-graphiql.png){: .normal }


(It's easiest to collapse the results, then select and copy them)

![08](/assets/img/post%20images/etl/n8n/20230222/08-n8n.png){: .normal }


Now let's hook this up to a webhook for processing elsewhere.

### Webhook configuration

![09](/assets/img/post%20images/etl/n8n/20230222/09-n8n.png){: .normal }


The webhook itself can be left as default if you like. A simple `GET` request to the URL shown in the webhook node will be enough to trigger this.

>If you have a more complex set of requirements, such as to limit who/what can trigger this- then consider configuring the `Authentication` on the webhook, and/or using an `IF node` to inspect the webhook headers and/or IP address/domain the request came from to prevent accidental or unauthorized hits.

Now the response needs a little tweak, because if we run this it will only return the first item, by default:

![10](/assets/img/post%20images/etl/n8n/20230222/10-n8n.png){: .normal }

So we'll use a common expression to wrap up everything that's incoming into a single response:
{% raw %}
```javascript
{{ JSON.stringify($input.all()) }}
```
{% endraw %}
The downside to this expression is it will give you the raw object from n8n which includes a couple wrapper elements:
{% raw %}
```json
[
    {
        "json": {      //             <----
            "id": "8494",
            "title": "Moe",
            "slug": "moe",
            "age": 2.8,
            "clickupId": "396kh10",
            "gdataLink": "https://www.greyhound-data.com/d?i=2481604"
        },
        "pairedItem": {//             <----
            "item": 0,
            "input": 0
        }
    },
    {
        "json": {      //             <----
            "id": "6868",
            "title": "Lucas",
            "slug": "lucas-3",
            "age": 2.5,
            "clickupId": "32rr46e",
            "gdataLink": "https://www.greyhound-data.com/d?i=2487522"
        },
        "pairedItem": {//             <----
            "item": 1,
            "input": 0
        }
    },
    {
        "json": {      //             <----
            "id": "6844",
            "title": "Mickey",
            "slug": "mickey",
            "age": 1.4,
            "clickupId": "35zpq6x",
            "gdataLink": null
        },
        "pairedItem": {//             <----
            "item": 2,
            "input": 0
        }
    }
]
```
{% endraw %}
To ***just grab the json keys*** we can use [`JMESPATH`](https://docs.n8n.io/code-examples/expressions/jmespath/), in another common expression I use:
{% raw %}
```javascript
{{ JSON.stringify($jmespath($input.all(),"[].json")) }}
```
{% endraw %}
Which gives us a much nicer:
{% raw %}
```json
[
    {
        "id": "8494",
        "title": "Moe",
        "slug": "moe",
        "age": 2.8,
        "clickupId": "396kh10",
        "gdataLink": "https://www.greyhound-data.com/d?i=2481604"
    },
    {
        "id": "6868",
        "title": "Lucas",
        "slug": "lucas-3",
        "age": 2.5,
        "clickupId": "32rr46e",
        "gdataLink": "https://www.greyhound-data.com/d?i=2487522"
    },
    {
        "id": "6844",
        "title": "Mickey",
        "slug": "mickey",
        "age": 1.4,
        "clickupId": "35zpq6x",
        "gdataLink": null
    }
]
```
{% endraw %}
Ok great now we have a feed!

![11](/assets/img/post%20images/etl/n8n/20230222/11-n8n.png){: .normal }

### Test it before moving on

Go to the webhook node, and click on the test url to copy it:

![12](/assets/img/post%20images/etl/n8n/20230222/12-n8n.png){: .normal }

> Alternatively just active and save the workflow, and click on Production URL and copy that one. Production URLs only work when the workflow is active!

Click the Execute Workflow button, so it starts listening for the test call:

![13](/assets/img/post%20images/etl/n8n/20230222/13-n8n.png){: .normal }


Visit the test URL, and note the response:

![14](/assets/img/post%20images/etl/n8n/20230222/14-n8n.png){: .normal }


This is because we didn't set the webhook to respond using the last node, so let's fix that:

![15](/assets/img/post%20images/etl/n8n/20230222/15-n8n.png){: .normal }


After selecting the Using Respond to Webhook Node setting, repeat the test.

***Response in browser:***

![16](/assets/img/post%20images/etl/n8n/20230222/16-results.png){: .normal }


Great!

Save your workflow, and switch over to creating a new feed in feed-me.


## Feed me config

### Detailed, official instructions: https://docs.craftcms.com/feed-me/v4/guides/importing-entries.html#setup-your-feed

### Quick version-
- Go to your feed-me panel at yourCraftSite/admin/feed-me/feeds
- Click New feed
- For this demo, the configuration looks like this:

![17](/assets/img/post%20images/etl/n8n/20230222/17-feedme.png){: .normal }


Click `'save and continue'` after you've configured this how you like.

> NOTE: This will now ping your URL so if it's not active you need to have it listening!
{: .prompt-warning }

*It's easiest to just active your workflow and use the production url during configuration*

Assuming n8n responds, we move on to mapping the root element:

![18](/assets/img/post%20images/etl/n8n/20230222/18-feedme.png){: .normal }


> NOTE: I ran into an exception error here because my Feed-Me was a version behind, and the newest version solves an issue with custom sources. See: https://github.com/craftcms/feed-me/pull/1224
{: .prompt-warning }


`Save and Continue` to begin field mapping, and use the nice interface to map your stuff:

![19](/assets/img/post%20images/etl/n8n/20230222/19-feedme.png){: .normal }

Because this is a demo, I've mapped the `Entity ID` - and selected it as one of the unique fields to be sure I don't clobber anything, and it only gets updated. Your map will probably NOT do this, so you can create new entries.

![20](/assets/img/post%20images/etl/n8n/20230222/20-feedme.png){: .normal }


Save, continue, and click `"Run Feed"` if you're ready.

Click over to the `"Logs"` tab in the feed-me menu, and review what it's done:

![21](/assets/img/post%20images/etl/n8n/20230222/21-feedme.png){: .normal }


That's it!!


## Plan 2 - Direct from another environment

Let's say you're copying/syncing `production` down to other `environments`. In that case we can skip the copy/paste stuff and just ***query Craft directly!***

Follow the docs to make sure your graphql endpoint works and can be reached:  https://craftcms.com/docs/4.x/graphql.html#getting-started

Now back in n8n let's add a graphql call to mimic what we'd done in *Plan 1:*

![22](/assets/img/post%20images/etl/n8n/20230222/22-n8n.png){: .normal }

Mind your credentials here- header auth will generally be in the format of `Authorization: Bearer randomstring`

eg:

![23](/assets/img/post%20images/etl/n8n/20230222/23-n8n.png){: .normal }


![24](/assets/img/post%20images/etl/n8n/20230222/24-n8n.png){: .normal }



The new flow looks like this- and will behave just as in `Plan 1` but dynamically based on your query!

![25](/assets/img/post%20images/etl/n8n/20230222/25-n8n.png){: .normal }

# Conclusion

And THAT is how we can sync/migrate data in Craft CMS in mere minutes using n8n.

How cool is that?!

As always, I hope you found this interesting🙂
