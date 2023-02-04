---
title: Magento 2 Time Savers
date: 2021-08-20 07:44
categories: [Magento]
author: bwilliamson
tags: [bash, magento, mysql, elasticsearch]
---

# We all get tired of WET coding (Writing everything twice).

Here are some excerpts from my ***`~/.zshrc`*** which save me a lot of time.
These are specific to Magento development via CLI, mostly.

## General

Extract a tar to a specific directory-
```bash
#e.g. restore-tar ~/tarname.tar.gz ~/myExtractedFolder
function restore-tar() {
    filename=${1:?"The file path must be specified."}
    directory=${2:?"The destination folder must be specified."}
    tar -xzvf $filename -C $directory
}
```

## ElasticSearch and Mysql

Replace ${PROJECT_NAME} with your database name, and filename as desired.
```bash
#ES server status
alias check-elastic='curl -XGET "localhost:9200/_cluster/health?pretty"'
#Log in to mysql as root - using valet you need to specify 127.0.0.1
alias mysql-root='mysql -h 127.0.0.1 -u root'
#Make a quick database backup. Use restore-db to roll-back.
alias mysqldump-magento="mysqldump ${PROJECT_NAME} -u root -p -h 127.0.0.1 | gzip > ~/dbdump/${PROJECT_NAME}_$(date "+%Y-%m-%d_%H%M").sql.gz"

#zcat a gzip sql file into specified db
#e.g. restore-db ~/dbdump/myfile.sql.gz magentoDBname
#for OSX use gzcat, it uses the same arguments.
function restore-db() {
    dbfile=${1:?"The path must be specified."}
    dbname=${2:?"The destination db name must be specified."}
    zcat $dbfile | mysql -h 127.0.0.1 -u root -p $dbname
}
```

## Removing Magento things
The ***`||:`*** is to allow the CLI to continue even if the result is an error. Directory not found, etc. I chain these aliases with other aliases via ***`&&`*** frequently, so this is a must for my workflow.
```bash
#Flush static files, including styles.
alias flush-static='rm -rf pub/static/* ||: && rm -rf var/view_preprocessed/* ||: && rm -rf var/cache/* ||: && rm -rf var/page_cache/* ||:'
#Dump most files, including frontend, followed by a setup and compile command. This is what I call 'the rain dance'
alias nuke-compile='rm -rf generated/* ||: && rm -rf pub/static/* ||: && rm -rf var/view_preprocessed/* ||: && rm -rf var/cache/* ||: && rm -rf var/page_cache/* ||: && bin/magento set:up && php -dmemory_limit=4G bin/magento setup:di:compile && bin/magento c:c'
#Dump files including generated classes, interceptors, and frontend files.
alias nuke-mage='rm -rf generated/* ||: && rm -rf pub/static/* ||: && rm -rf var/view_preprocessed/* ||: && rm -rf var/page_cache/* ||: && rm -rf var/cache/* ||:'
```

## Magento Cache things
I know these are silly, but I'm very tired of typing ***`bin/magento`*** all day.
```bash
alias c:c='bin/magento c:c'
alias c:f='bin/magento c:f'
```

## Magento Compile and Deploy things
These are assuming the mode is 'developer'.
```bash
#Part of my rain dance, I usually precede a compile with a setup command. Not always necessary though.
alias compile-mage='bin/magento set:up && php -dmemory_limit=4G bin/magento setup:di:compile && bin/magento c:c'
#Keep generated is useful in development when you just want to load in your new module.
alias compile-mage-kg='bin/magento set:up --keep-generated && php -dmemory_limit=4G bin/magento setup:di:compile && bin/magento set:static-content:deploy -f && bin/magento c:c'
#Just the frontend- in development this is silly to use, but sometimes you want to see if there's a LESS compilation issue.
alias deploy-static='bin/magento set:static-content:deploy -f'
```

## Aliases for Aliases
These are more silly shorthand for myself.
```bash
alias set:up='bin/magento set:up'
alias di:compile='php -d memory_limit=4G bin/magento setup:di:compile'
```

## Git things
```bash
alias fpg='git fetch && git pull'
```

## Conclusion

I like to flirt with the line between being lazy and efficient. [Does it need to be automated?](https://xkcd.com/1205/) Sure. Why not?
