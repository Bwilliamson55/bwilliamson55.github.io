---
title: How to ... Entities Part 1
date: 2021-12-04 16:13
categories: [Magento, Dev]
author: Bwilliamson
tags: [magento, php, how to entities, entities]
mermaid: true
---

A very common question in Magento development is *How to bla bla entity(ies)*. 

I find myself doing this from time to time as well, either out of laziness or because I've run into something quirky. 
>Have you noticed how the `ProductRepository::Save()` method requires the product exist first? Quirky right? 

This will be the first post in a small series of posts on *How to bla bla* entities. We're mainly refering to CRUD or other common actions. I'll be focusing on the most common entities in Magento 2.4.x, to keep things practical. 

# Which Entities?

Well, basically anything with an `entity_id` column. 

You'll notice in your travels that these are often declared using the constants from their Models. These strings are handy to have on hand, and make this series of posts easy to organize!

We'll be breaking these posts down into a few main categories:
- Intro (We are here)  
- Store Stuff
- Customer Stuff
- Catalog Stuff
- Stock (Inventory) Stuff
- Other Stuff (Like orders)

*I have this list in my gists on my github repository as well.*

| Entity type id | Where it's declared |
|:----------------|:-----------------|
| ---Store Stuff---
| store | \Magento\Store\Model\Store::ENTITY |
| store_website | \Magento\Store\Model\Website::ENTITY |
| store_group | \Magento\Store\Model\Group::ENTITY |
| ---Customer Stuff---
| customer | \Magento\Customer\Model\Customer::ENTITY |
| customer_address | \Magento\Customer\Model\Indexer\Address\AttributeProvider::ENTITY |
| customer_group | \Magento\Customer\Model\Group::ENTITY |
| ---Catalog Stuff---
| catalog_eav_attribute | \Magento\Catalog\Model\ResourceModel\Eav\Attribute::ENTITY |
| catalog_product | \Magento\Catalog\Model\Product::ENTITY |
| catalog_category | \Magento\Catalog\Model\Category::ENTITY |
| ---Stock (Inventory) Stuff---
| cataloginventory_stock_item | \Magento\CatalogInventory\Model\Stock\Item::ENTITY |
| cataloginventory_stock | \Magento\CatalogInventory\Model\Stock::ENTITY |
| ---Other Stuff---
| config_data | \Magento\Framework\App\Config\ValueInterface::ENTITY |
| order | \Magento\Sales\Model\Order::ENTITY |

# How to *bla bla*

The way Magento ties everything together and utilizies these entities is not exactly intuitive.  

What is the *right* way to get, set, update, and delete these things?  
The answer to this changes with the version of Magento- but as of 2.4.x best practices, the following details are pretty close - making exceptions where practical. 

This post aims to put it all together so we can understand the pieces of the puzzle, then apply them in the following posts. 

## Putting it all together  
Here's the stuff we're talking about:
* Interfaces
* Repositories
* Factories
* Models
* Resource Models
* Collections

<details markdown="1">
  <summary>Expand me to see Diagram</summary>

![02](/assets/img/post images/magento/entities/02.svg){: .normal }

</details>  

*****

Ok how do we use this stuff?  

In a perfect world, we could use the repository for everything right? ... ***right***?  
In reality, we end up using a mixture of Repositories, Factories (plus what they create), and Collections. It ***should be*** rare to never that you need to use anything else for entity manipulation, but as always, there are more exceptions than we'd like in the core code.

The best example of the above diagram is the Magento customer module. Everywhere else it gets tricky sometimes, so that's where the examples in this series come in.

### Rough Game Plan
To put these pieces to work, let's look at a pretty standard workflow when you're manipulating an entity. We can generally assume that we are talking about the **Interface** of the puzzle piece where possible. 

1. Get thing(s) with repository 
   1. If thing doesn't exist, create it with a factory
   2. If list of things with filters are needed, use collection instead of repository
2. Update the thing via it's built in methods `$thing->setStuff($data)` 
   1. If necessary use the things resource model
3. Save the thing with its repository->save method 
   1. If we're just updating attributes of the thing, save them with the resource models `updateAttribute` method if it has one. This is much faster than saving the whole object.
   2. If the Repository doesn't have a save method, use the resource model save method for that thing.
      1. You can also use the models save method, but that is generally deprecated.



## Why use interfaces?  
Well, if you want to conform to a '[service contract](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/service-contracts/service-contracts.html)' by Magento standards, you should always be using an interface. There really are some great reasons to use them-
* Extensibility
  * Interfaces, especially repository interfaces, allow you to map things more easily in Magento to the REST API. By using classes in `<module>/Api`, you're using the same methods the REST API can use.
  * Most entity interfaces extend the `\Magento\Framework\Api\ExtensibleDataInterface` which allows other modules to extend this interface with more attributes/properties.
  * This is mostly from the perspective of creating your own interfaces for your own module, but the points stand.
* Maintainability
  * Interfaces, per Magento's 'service contracts' ***will not change*** - therefore, your module will survive any major core updates such as if they rip and replace the ORM.  

> Side note: How do interfaces know which class to use? I mean, if I inject an interface and instantiate it, what's actually instantiated? A model, usually. Check the `etc/di.xml` in the module that holds the interface; that's where interfaces are usually mapped to classes.


## Why use repositories?
Well, if you use a repository**interface**, we can apply the same arguments above in the 'pro' column. Also, a lot of direct `save()` methods have been deprecated on models themselves, in favor of resource models and their repositories (preferably) doing the saving instead. 
This is a good idea, as it de-couples the entities (models) from persisting their own data.  

To search using repositories, you'll use a [Search Criteria interface](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/searching-with-repositories.html#m2devgde-search-criteria). This is different than collections, and will probably earn its own post. 

> ***But the repository doesn't do everything I need***.  
Yea, I get it. Especially for things like websites/store/store_view - you may need to use the resource models directly. I didn't tell you this, but as soon as the repository starts to slow you down, skip it and use stuff directly within reason.

## Models, Resource Models, and Collections oh my
What are these things anyway?  

Here's the summary:
* Resource Models sit on the DB and do direct CRUD actions. 	
* Models are our entities and business logic that use the resource models.
* Collections are groups of models, that we can use in a lot of ways - If you need to use a WHERE clause, GROUP BY or ORDER BY in your search, use a collection, otherwise use a repository
  
Similar to repositories- the way we search collections is a bit out of scope for this. It's a fun topic, but deserves its own post.

>  Side note- repositories are heavy when fetching lists. If you do `getList()` through a repository, and watch the queries it runs, you might start looking for another option. I like to stick with Collections for fetching more than one thing. Otherwise, use get/getById from the repository.

# Stay Tuned

That's the overview of what we're talking about.

In the following posts I will be writing about, and sharing examples of the rough game plan above, per entity category.

If this is all fairly new to you, hang in there- it gets easier as you start to put it to work.


