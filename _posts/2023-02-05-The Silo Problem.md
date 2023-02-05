---
title: The Silo Problem
date: 2023-02-05 08:00
categories: [General, Dev, Sysadmin, devops]
author: bwilliamson
tags: [devops, general, silos, projects]
---
# What is The silo problem?
> *The silo problem is a name I'm using for any knowledge or experience based bottlenecks in projects.*

    It's all too common these days to be expected to know everything. If you're a nerd, you can fix my email, right? Or my website? No?
How many of us dread family encounters for this reason?
The same idea applies in modern technology stacks. Unless you're one of the people that understands the technology, it's just more acronyms to you. This can be seen in the hilarious examples found about the internet where job postings require more years of experience than the age of the technology in question. This isn't so far from the truth though; When's the last time a skills requirement list had less than five acronyms in it? We are all expected to know a whole lot of stuff!

I love knowing a whole lot of stuff though.

It's why I'm in the development field. Abundant complex problems to itch that drive to create something useful. This notwithstanding, there is limit to what we are capable of.

     Knowledge, skill, and experience are finite resources.
These resources are called silos because often times they relate to other things in their "vertical" - ie Javascript and Typescript, Powershell and Windows, Bash and Unix. Rarely are these vertical relationships absent of each other. Hence, Silos.

# The problem
> How many silos can be required for a product or solution before it's just too complex to handle or before it's completely unrealistic?
>
The answer varies wildly depending on context of course, but how could we quantify this data? Or how could we at least qualify the data?

The problem is there are too many silos. Usually.
Usually a project is over complicated to fit the capacity of the environment it's built in.
Like water, or greedy regex, it will take on the shape and size of the bounds it's put in. Just because you can doesn't mean you should, but the project team doesn't know that. Usually.
It's generally not difficult to identify a process, product, or project that was or is over complicated. I'm almost certain you're working on one right now.

# The hypothesis
> Less silos will make a better product, process, or project. (Or anything, really)

If the overall complexity of a project is reduced by removing silos, while maintaining its goals, then the product will be better.
"Better" - right, how?
To me, less complexity means (Not exclusively):
- More support
	- Less silos means more skill overlap which means support resources cover more of the product per person
- Less iteration time
	- How fast could you push changes to your project if there was only a single team involved?
- Lower costs
	- Not just money - opportunity cost of labor and skill should not be forgotten.
		- The search for that unicorn team member never ceases because the combination of silos is ever changing.
- Smaller scopes
	- Less variety of stuff, less scope. Usually.
- More available talent
	- Similar to more support,  if you can triple instead of double your skilled resources because of silo consolidation, your project is more resilient to all sorts of risks.
- More time to accomplish goals
	- Dropping a few rows from a timeline of a project, or putting them in parallel, can have a big impact.
- Better user and developer experience
	- Less silos means less complexity in coupling and cohesion, which leads to a lower rate of issues. Knowledge and skill gaps become smaller.
	- Less silos means less resources necessary to address user feedback.

This is certainly not an exhaustive list, but these are some pros that are easy to demonstrate.

## Story Time
I once knew a range safety officer that had a very interesting military background.
He served in the first Gulf war, when we didn't really have any (or have many) explosive ordinance disposal (EOD) units yet in the field. The high demand for these specialists created an interesting program inside the armed forces where electricians ( like my friend ) were trained in explosive disposal. This was to accelerate the rate of producing these specialists because, as the leadership learned, it's easier and faster to train an electrician to work with explosives, than to train an explosives expert to work with electronics.

> The moral of the story is at the heart of the silo problem - bespoke specialties requiring cross training are not all created equal.

Sometimes the silos are too big to take on from the silos you're currently in. That's why different departments and careers exist of course, but as technology marches forward, we shouldn't hesitate to capitalize on our existing, or required silos.

## Example
You have a new eCommerce store you want to host.
You have a development team, and an operations team. Your operations team is already running some projects for you in AWS, built with .net.
Your development team is strong in Javascript and front end styling, but a bit weak in .net / IIS. A lot of them have come from the newer front end technologies like react and vue.
Locally, they develop with a docker based solution so their environments are consistent.
Their personal computers are a mixture of make and model, but they all use and know linux because that's what the docker solution uses, as most modern web stuff does.

Now let's look at some options for this new eComm project
1. Build it in AWS, leveraging existing skills there and in .net. Use the devs to make the front look pretty, requiring some backend dev help you may not have yet, to get that whole stack talking.
2. Build it in AWS, using the dev teams prefferred stack, and train up the ops team to support it.
3. Build it in an IaaS 3rd party managed system, where all the devs need to do is push to their repository.

If the product is the same in all three cases,  which do you think would be the best option?

# Conclusion

There is no conclusion.
This is an ongoing thought experiment of mine, wherein we can apply almost any project and identify large benefits through silo consolidation.
The driver of silo separation in my experience has been the application of previous or existing projects and infrastructure to new projects or ideas. This is partially due to a fear of change. We can't just switch things! We would need to train people! Or worse, RETRAIN people!
There's no magic bullet of course, but we can do something right now which is to ***consider the silos.***
