---
title: Disabling 2FA in local Magento 2 development
date: 2024-05-14 19:30
categories: [Magento, Dev]
author: bwilliamson
tags: [magento, ddev, docker, bash, zsh]
---
There seems to be some confusion around how to disable 2FA in a local Magento 2 environment, or, more specifically, how difficult it is.

# It's just one command
I'm not sure why modules out there exist for this. So I thought it would be fun to post this as a quick guide.


> **Protip**: If you're using DDEV, consider making an alias in your home additions folder for this.

> **Pro Pro Tip**: If you're like me, you won't do the easy thing and instead you'll just do CTRL+R and type 'Two' after having done this command once, and it will be the first thing that pops up. Usually.

```bash
bin/magento module:disable Magento_AdminAdobeImsTwoFactorAuth Magento_TwoFactorAuth
&&
bin/magento setup:upgrade
&&
bin/magento setup:di:compile
```
*Ok so maybe that's 3 commands. But you can totally make it as a single thing.*

That's it! You're done.

In my local, I'll use my `dance` alias instead of the setup:upgrade, etc, but you get the idea.

# Conclusion
I'm not sure why modules exist for this.
