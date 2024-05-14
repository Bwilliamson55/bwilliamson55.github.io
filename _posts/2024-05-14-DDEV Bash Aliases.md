---
title: DDEV Bash Aliases
date: 2024-05-14 08:00
categories: [Magento, Dev]
author: bwilliamson
tags: [magento, ddev, docker, bash, zsh]
---
Do you have your aliases set up in your DDEV project? You should! Here's a quick guide to setting up some useful aliases for your DDEV projects.

# Create the file
In your DDEV project, in the .ddev directory, create this file:
`.ddev/homeadditions/.bash_aliases` or `.ddev/homeadditions/.zsh_aliases` if you're using zsh.

# Add your aliases
Here are some of my favorites:

```bash
alias dance="bin/magento setup:upgrade && bin/magento setup:di:compile && bin/magento setup:static-content:deploy -f"
alias dance-fast="bin/magento setup:upgrade --keep-generated && bin/magento setup:static-content:deploy -f"
alias dance-front="rm -rf pub/static/frontend/* var/view_preprocessed/* && bin/magento c:f && bin/magento setup:upgrade --keep-generated && bin/magento setup:static-content:deploy -f"
alias nuke="rm -rf var/cache/* var/page_cache/* var/view_preprocessed/* pub/static/* generated/*"
alias disable-caches="bin/magento cache:disable block_html full_page layout"
alias enable-caches="bin/magento cache:enable block_html full_page layout"
```

Full documentation here: [https://ddev.readthedocs.io/en/latest/users/extend/in-container-configuration/](https://ddev.readthedocs.io/en/latest/users/extend/in-container-configuration/)

## Restart your project
After you've added your aliases, you'll need to restart your project to see them take effect.

```bash
ddev restart
```


# That is all
That's it!

Once your project is restarted, you can use these aliases inside a `ddev ssh` session.

If you're a Magento dev, you'll likely find these useful. If you're not, you can still use the concept to create your own aliases for your projects.
