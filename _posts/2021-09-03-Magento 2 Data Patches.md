---
title: Magento 2 Data Patches
date: 2021-09-03 10:19
categories: [Magento]
author: Bwilliamson
tags: [magento, data patches]
---

***Data Patches; They're awesome.***

Before 2.3, Magento used Install and Upgrade scripts to modify your schema, and other things. This has been deprecated in favor of a much easier (I think) declaration schema. This new mess of stuff is [outlined here](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/declarative-schema/migration-commands.html). This is closely related, but not the topic at hand. It's worth mentioning, because of the mountain of old information out there using the pre-2.3 methods, which can be terminologically confusing with more current methods. 

Basically, if you find information telling you to use an `InstallData.php` file- it's out of date. 

At the time of this writing, the current method is [outlined here in the dev docs](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/declarative-schema/data-patches.html). In this post, we're focused on data patches, not schema patches.

# How it works

* Create your patch file in `<Vendor>/<Module_Name>/Setup/Patch/Data/<Patch_Name>.php`
    * Better practice is to name your patch in a meaningful way, and name the class the same
        * e.g. `AddCatalogAttributes.php` and `class AddCatalogAttributes`
* Run `bin/magento setup:upgrade` and the patches `apply()` method will be executed
* A row will be written into the `patch_list` table if the patch is successfully executed


## Requirements

* For a data patch you must implement `Magento\Framework\Setup\Patch\DataPatchInterface`
* Due to the interface, we must define the following three methods
  
---
---

## apply(), getDependencies() and getAliases()  
  

### apply()  
is the meat and potatoes. The stuff in here runs when the patch executes. 

### getDependencies()
is where we define any other patches we want to run before this one.
e.g.   
```php
public static function getDependencies(){
    return [
        \Vendor\Module\Setup\Patch\Data\FilenameWithoutExtension::class
    ];}
```

### getAliases()
is where we define other names for this patch. The only use case I know of, is if you change the module name - so technically the path of the patch changes, and triggers a new patch install. This function gives us a way to avoid that. I have never used this but it must be defined. 

## Revert
The devdocs are confusing on this subject. So I'll try to clarify; You can revert patches.  
The problem is, you can't revert a ***single, specific patch*** **among other patches in the same module**.   

This:  
```bash
bin/magento module:uninstall --non-composer Vendor_ModuleName
```
will run the `revert()` method in **every** patch file in the named module.  

There's no magic here, you need to define your own uninstall steps in the `revert()` method:
```php
    /**
     * Delete an Attribute
     */
    public function revert()
    {
        $eavSetup = $this->eavSetupFactory->create(['setup' => $setup]);
        $eavSetup->removeAttribute(
            \Magento\Catalog\Model\Product::ENTITY,
            'attribute_code');
        }
    }
```


## Re-run and versioning  

If you want to re-run the patch, you can delete the corresponding row in your `patch_list` table.  
This is acceptable in dev, but there's a cleaner way to re-run patches in the case of a new patch version.

In addition to `Magento\Framework\Setup\Patch\DataPatchInterface` implement `Magento\Framework\Setup\Patch\PatchVersionInterface` as well.
You then must declare the `getVersion()` method:
```php
public static function getVersion()
{
return '1.0.1';
}
```
If the number it returns is **higher** than the version in the modules **`module.xml`**, then the patch is run.

# Examples

The wonderful thing about patches is they're simple. It's a single file, that's run once. We really don't have many limitations on what we can do with this.  
Everyone loves examples. Especially future me - copy paste ftw. 

This section needs some work- but we have good examples I'll be posting here soon.

## Attributes
### Product
### Category
### Address
### Order
### Customer
---
## CMS blocks
## CMS pages
## Admin configurations
### Theme config/creation
---
## Create stuff
### Product
### Customer
### Category
### Website/Store/StoreView