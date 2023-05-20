---
title: How to Extend Craft CMS html purifier with html5 elements
date: 2023-05-20 08:00
categories: [CraftCMS, Dev]
author: bwilliamson
tags: [craftcms, html5, htmlpurifier, redactor]
---

# TLDR

To extend the out of the box htmlpurifier element definitions, use a custom module ([Follow this official guide](https://craftcms.com/docs/4.x/extend/module-guide.html) to get a empty module quickly), and tap into the event `EVENT_MODIFY_PURIFIER_CONFIG`, utilizing the following snippet in your module's `init()` method.
To make this easier, we'll download most of the tricky definitions: `composer require xemlock/htmlpurifier-html5`, so we can simply say things like:
``` php
$pictureContent = new HTMLPurifier_ChildDef_HTML5_Picture();
// ...
$def->addElement('picture', 'Flow', $pictureContent, 'Common');
```

The official html purifier docs, aren't super intuitive: [http://htmlpurifier.org/docs](http://htmlpurifier.org/docs)
[The HTML purifier customization guide](http://htmlpurifier.org/docs/enduser-customize.html) may explain some of what we're doing, mainly why adding an element like `<video>` ***looks*** complicated.. Scroll way down to `"Putting it all together"` where the structure and parameters for creating a field is drawn out. Or just copy the code below, I'm not your boss.

Snippet below adds the following HTML5 elements (and common child attributes) to htmlpurifier (allowing them):
- `<video>`
- `<audio>`
- `<source>`
- `<track>`
- `<picture>`
- `<img>` (empty tag)

<details markdown="1">
  <summary><strong>Code for your modules main file </strong></summary>

### ***Note***:
Compare this to the boilerplate produced by `craft make module`, the `attachEventHandlers()` is basically all that's changed here

```php
namespace modules\YOURMODULE;

use Craft;
use HTMLPurifier_ChildDef_HTML5_Media;
use HTMLPurifier_ChildDef_HTML5_Picture;
use yii\base\Module as BaseModule;
use craft\redactor\Field;
use craft\htmlfield\events\ModifyPurifierConfigEvent;
use yii\base\Event;

/**
 * purifier-config module
 *
 * @method static Module getInstance()
 */
class Module extends BaseModule
{
  public function init(): void
  {
    Craft::setAlias('@modules/purifiermodule', __DIR__);

    // Set the controllerNamespace based on whether this is a console or web request
    if (Craft::$app->request->isConsoleRequest) {
      $this->controllerNamespace = 'modules\\purifiermodule\\console\\controllers';
    } else {
      $this->controllerNamespace = 'modules\\purifiermodule\\controllers';
    }

    parent::init();

    // Defer most setup tasks until Craft is fully initialized
    Craft::$app->onInit(function() {
      $this->attachEventHandlers();
    });
  }

  private function attachEventHandlers(): void
  {
    Event::on(
    Field::class, // When the craft\redactor\Field class sends the following event
    Field::EVENT_MODIFY_PURIFIER_CONFIG,
       static function(ModifyPurifierConfigEvent $event) {
         if( $def = $event->config->getHTMLDefinition(true) ) {
    // xemlock/htmlpurifier-html5 library used to assist extending the HTML5 definitions
    // modifying config/htmlpurifier/Default.json wont cut it for this
           $mediaContent = new HTMLPurifier_ChildDef_HTML5_Media();
           $pictureContent = new HTMLPurifier_ChildDef_HTML5_Picture();
           // https://html.spec.whatwg.org/dev/media.html#the-video-element
           foreach (['video', 'audio'] as $element) {
             $def->addElement($element, 'Flow', $mediaContent, 'Common', [
               'controls' => 'Bool',
               'height' => 'Length',
               'poster' => 'URI',
               'preload' => 'Enum#auto,metadata,none',
               'src' => 'URI',
               'width' => 'Length',
             ]);
             $def->addElement($element, 'Inline', $mediaContent, 'Common', [
               'controls' => 'Bool',
               'height' => 'Length',
               'poster' => 'URI',
               'preload' => 'Enum#auto,metadata,none',
               'src' => 'URI',
               'width' => 'Length',
             ]);
           }

           // https://html.spec.whatwg.org/dev/embedded-content.html#the-source-element
           $def->addElement('source', false, 'Empty', 'Common', [
             'media' => 'Text',
             'sizes' => 'Text',
             'src' => 'URI',
             'srcset' => 'Text',
             'type' => 'Text',
           ]);

           // https://html.spec.whatwg.org/dev/media.html#the-track-element
           $def->addElement('track', false, 'Empty', 'Common', [
             'kind' => 'Enum#captions,chapters,descriptions,metadata,subtitles',
             'src' => 'URI',
             'srclang' => 'Text',
             'label' => 'Text',
             'default' => 'Bool',
           ]);

           // https://html.spec.whatwg.org/dev/embedded-content.html#the-picture-element
           $def->addElement('picture', 'Flow', $pictureContent, 'Common');
           $def->addElement('picture', 'Inline', $pictureContent, 'Common');

           // https://html.spec.whatwg.org/dev/embedded-content.html#the-img-element
           $img = $def->addBlankElement('img');
           $img->attr = [
             'srcset' => 'Text',
             'sizes' => 'Text',
           ];
         }
       }
    );
 }
}
```
</details>

# Why

When working with the very common [Redactor field](https://plugins.craftcms.com/redactor) in Craft CMS, by default, the HTML (if any) put in the field is purified before saving.
This helps prevent malicious and broken HTML from entering your content, and breaking or compromising your frontend.

## Just diable "Purify HTML" on the field
Right? ... Right??
That **would** allow you to put whatever you want in your content and it won't be messed with. The problem here is security (In some cases), but more so for me, allowing broken HTML.

I'm writing this because I had to solve this recently, and simply disabling the purification quickly broke a lot of my pages. Why? Because the content, depending on the site, is syncronized automatically from other systems. This sync does not care if the HTML is bad, and showed me, in the worst way, how easy a simple tag can break a page.

eg `<div class="class-1 class2>`

What's wrong with that? Just the missing `"`, no biggie right? Well.. Should you have any javascript that targets attributes, this will wreak havoc and throw warnings like `INVALID ATTRIBUTE "class-1"` and crash some scripts.

## What about the htmlpurifier/Default.json?

Common use cases, for things like embedding YouTube or Vimeo content, is doable with a few small tweaks. There is a [Redactor Plugin](https://imperavi.com/redactor/plugins/) for allowing videos: [https://imperavi.com/redactor/plugins/video/](https://imperavi.com/redactor/plugins/video/) - where in combination with a small change in your `htmlpurifier` config will make embedding videos super easy... ***If they're on those platforms***

See [this stackexchange question](https://craftcms.stackexchange.com/questions/20133/html-purifier-config-file) (2 years old but still mostly relevant) - there are a ton just like this all over the web.

---

TLDR on that - Here's my <strong><em>iframes.json</em></strong> config file I use.

This file is in my project's `/config/htmlpurifier/` folder

```json
{
  "Attr.AllowedFrameTargets": [
    "_blank", "_target", "*"
  ],
  "Attr.EnableID": true,
  "HTML.Trusted": true,
  "HTML.AllowedComments": [
    "pagebreak"
  ],
  "HTML.SafeIframe": true,
  "URI.SafeIframeRegexp": "%^(https?:)?//(www.youtube.com/embed/|player.vimeo.com/video/|forms.clickup.com/|app-cdn.clickup.com/forms-embed/)%"
}

```


---


# How


Could you make your own Redactor plugin to solve this?
**Probably!** -- Here's the docs on that, which I decided not to try: [https://imperavi.com/redactor/docs/how-to/create-a-plugin/](https://imperavi.com/redactor/docs/how-to/create-a-plugin/)

Instead I chased this rabbit for the better part of a day. I read through all the different ways the purifier library interacts with Redactor, and Craft CMS, and determined a lot has changed over the years. For instance, ***the event targeted here used to be in the Redactor codebase, not Crafts!*** ([See Redactor changelog, 2020](https://github.com/craftcms/redactor/blob/850e69a8239ea5ef6936af4ef1d1c7d7f1d67f2d/CHANGELOG.md?plain=1#L268)) and then last year, [see the deprecation notice](https://github.com/craftcms/redactor/blame/main/src/events/ModifyPurifierConfigEvent.php).

The HTML5 element definitions are basically never getting added to Redactor ***via htmlpurifier itself***, [per this discussion](https://github.com/ezyang/htmlpurifier/issues/160#issuecomment-735522214). Redactor states they plan to move to a different purification library soon anyway - [see this thread for source](https://github.com/craftcms/redactor/issues/78#issuecomment-1023027396).

Ok so HOW?!

Well, as stated allll over those github discussion boards, there's already an implementation that does this, via [xemlock's htmlpurifier implementation](https://github.com/xemlock/htmlpurifier-html5).

*Alright, how do we swap in that, for the existing htmlpurifier that Redactor and Craft use?*

Out of an abundance of caution, and with no real experience messing with Crafts dependency injection, I found the method in the TLDR code to be most transparent and harmless. There's probably other ways, but once I reviewed what was happening in Xemlock's library (See each of the types added to the definition here: [https://github.com/xemlock/htmlpurifier-html5/tree/master/library/HTMLPurifier/ChildDef/HTML5](https://github.com/xemlock/htmlpurifier-html5/tree/master/library/HTMLPurifier/ChildDef/HTML5)), I decided tapping the event wouldn't be so bad.

As an added bonus, developers that find this change should immediately understand and infer how to add their own custom types, without learning about all of this history!

---

As always, I hope you found this interesting!
