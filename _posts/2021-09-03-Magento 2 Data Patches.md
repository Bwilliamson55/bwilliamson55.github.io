---
title: Magento 2 Data Patches
date: 2021-09-03 10:19
categories: [Magento, Dev]
author: bwilliamson
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

This section needs some work- but as we get more examples they will be posted.
I have a repository with the full example [files here.](https://github.com/Bwilliamson55/PatchExamples)

-----

## Attributes

Almost all the attribute types will work the same way when creating/updating/removing them. In this case we're talking about [EAV attributes](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/attributes.html#custom), **not** [extension attributes](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/attributes.html#extension).

The classes to inject are `Magento\Eav\Setup\EavSetupFactory` and `Magento\Framework\Setup\ModuleDataSetupInterface`. Then initialize the setup with
```php
$eavSetup = $this->eavSetupFactory->create(['setup' => $this->moduleDataSetup]);
```
Each attribute type has a different set of allowed data (Its fields), so check the devdocs if those fields aren't covered here.

Create/Update with the following methods, the `self::blabla` is just an example. You can hardcode these values if you wish.

```php
$eavSetup->addAttribute(
    self::ENTITY_TYPE_ID, // Text type id like catalog_product
    self::ATTRIBUTE_CODE, // Text code like 'my_attribute'
    self::ATTRIBUTE_DATA  // Array of key-value pairs for attribute fields+values
);
// to update:
$eavSetup->updateAttribute(
    self::ENTITY_TYPE_ID,
    self::ATTRIBUTE_CODE,
    self::ATTRIBUTE_DATA
);
```
I have a gist for entity type IDs and where they are declared [here.](https://gist.github.com/Bwilliamson55/748de11b959afc09322b332a2ae807c5)

### Customer Attributes

I have recently added Customer attribute examples to the example repository linked above (~11/21). There are some gotchas worth noting:
> If your customer attribute is not saving/persisting data on the customer object- be sure it has an `attribute_set_id`, and `default_group_id`. In addition, these need to be added to the attribute- NOT during creation.

*Excerpt from customer attribute patches*
~~~ php
...
$customerEntity = $this->eavConfig->getEntityType('customer');
$attributeSetId = $customerEntity->getDefaultAttributeSetId();

$attributeSet = $this->attributeSetFactory->create();
$attributeGroupId = $attributeSet->getDefaultGroupId($attributeSetId);
...
$attribute = $this->customerSetup->getEavConfig()
    ->getAttribute(Customer::ENTITY, self::ATTRIBUTE_CODE);
if ($attribute) {
    $attribute->setData('used_in_forms', [
        'adminhtml_checkout',
        'adminhtml_customer',
        'customer_account_edit',
        'customer_account_create'
    ]);
    $attribute->setData('attribute_set_id', $attributeSetId);
    $attribute->setData('attribute_group_id', $attributeGroupId);
    $this->attributeResource->save($attribute);
}
...
~~~

> Why are we using `CustomerSetup`? It just extends `EavSetup`..
Yep - but that's how the docs have it, and it has a neat extra function called `updateAttributes` which we are ironically not even using in our examples XD

### Select, Multiselect

Here's the TLDR on how to do select/multiselect from a patch:
- 'input' will be `select`/`multiselect`, and 'type' can be anything, but I generally use `text` - `int` works in many cases as well.
- 'source' will be `Magento\Eav\Model\Entity\Attribute\Source\Table` unless you have a source class.
- For multiselect- 'backend' will be `Magento\Eav\Model\Entity\Attribute\Backend\ArrayBackend` unless you have special requirements there
- 'user_defined' should be `true` and 'system' should be `false` if you want to be able to change/delete the attribute and it's values from the admin
- Option values for each are written the same way
  - For Simple arrays:
  - ~~~php
    'option' => [ 'values' =>
        [
        'Option 1',
        'Option 2',
        'Option C'
        ]
    ],
    ~~~
  - For Scoped (per store view) arrays: ***notice values is now value***
  - ~~~php
    'option' => [ 'value' =>
        [
            'option1'=>[
                0=>'Bla bla!',    // 0 is store id
                1=>'I am store id 1!',
                13=>'I am store id 13!'
            ],
            'option2'=>[
                0=>'Bla bla! 2!',    // 0 is store id
                1=>'I am store id 1! 2!',
                13=>'I am store id 13! 2!'
            ],
        ],
        'order'=> //Sort Order
        [
            'option1'=>1,
            'option2'=>2
        ]
    ],
    ~~~

----
## CMS blocks and pages
See the examples in the [repository](https://github.com/Bwilliamson55/PatchExamples) for a full example.

Pages and Blocks work a little differently from each other. In both cases though, I find it easiest to build your CMS data by hand, then go pull the value of the column `content` from the `cms_block` or `cms_page` tables.

**Blocks** will accept
```php
[
    'title' => 'CMS block title 2',
    'identifier' => 'example-block-2',
    'content' => 'This would be content directly from the cms_block tables content field',
    'is_active' => 1,
    'stores' => [], //the stores this is available to- 0 for default/admin/all
    'sort_order' => 0
]
```
Inject `Magento\Cms\Model\BlockFactory` and save the block with
```php
$this->blockFactory->create()->setData($blockData)->save(); // $blockData is the array above
```

For **Pages** - they will accept
```php
[
    'title' => 'Example cms page 1',
    'page_layout' => 'cms-full-width',
    'meta_keywords' => '',
    'meta_description' => 'lorem',
    'identifier' => 'example-page-1-slug',
    'content_heading' => '',
    'content' => 'This would be content directly from the cms_page tables content field',
    'layout_update_xml' => '',
    'url_key' => 'example-page-1',
    'is_active' => 1,
    'stores' => [],  //the stores this is available to- 0 for default/admin/all
    'sort_order' => 0,
    'meta_title' => 'lorem'
]
```
Inject `Magento\Cms\Model\PageFactory` and save the page with
```php
$this->pageFactory->create()->setData($pageData)->save(); // $pageData is the array above
```
---
## Admin configurations
This is easier than most - inject `Magento\Framework\App\Config\Storage\WriterInterface` and store your config settings like so
```php
$this->configWriter->save(
    $path, // such as web/secure/base_url
    $value, // such as 'https://magento-2.test/'
    self::CONFIG_SCOPE_TYPE, // such as 'websites' or 'stores' or 'default'
    self::CONFIG_SCOPE_ID // The ID related to the scope - website ID or store ID or 0 for default
);
```
I find this easiest to pull these values right from the DB after I've clicked my way to the config I want. You can sort the `core_config_data` table by its `updated_at` column, or `id` desc and presto.

The only gotcha here is when your IDs change across environments. In that case I hope your text codes are consistent so you can retreive your IDs via the code before saving this.

----
## Theme config/assignment
Once you have your theme files in place, you can update its configuration with a patch

Inject `Magento\Theme\Model\Data\Design\ConfigFactory` and `Magento\Theme\Model\DesignConfigRepository`. If you already know your Theme ID great, otherwise inject `Magento\Theme\Model\ResourceModel\Theme\CollectionFactory` too.

Build the config, then save it:
```php
// Get the theme ID
$themeCollection = $this->themeCollectionFactory->create();
    $theme = $themeCollection->getThemeByFullPath('frontend/MyVendorName/MyThemeName');
    $themeId = $theme->getId();
// Put the ID in an array, as the minimum.
$data = [
    'theme_theme_id' => $themeId
];
//Save the configuration to the theme - in this case we're setting its scope and assigned website
$designConfigData = $this->themeConfigFactory->create(
    self::CONFIG_SCOPE_TYPE, // can be 'websites', 'stores', 'default'
    $websiteId, // ID of the scope type- so, websiteId or storeId or 0
    $data
);
$this->designConfigRepository->save($designConfigData);
```
---

