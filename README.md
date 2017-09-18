# git-migration-hook

We all love git and the easy ways it allows us to create, switch and merge
branches. However, when you have a project, say a website, which uses a
database it's hard to migrate the database schema with your project code (i.e.
PHP code).

Consider the following scenario. You have two branches:
* master - which contains the website that's currently online
* feature - which contains an unfinished new feature

For the new feature to work properly, it needs to change the database schema.
The workflow is as follows:

1. When, in you feature branch, you change your code which needs a change of the
   database schema, you also commit two new SQL files in the migrations directory,
   say:
   * ```20151120130200-extra-field-up.sql``` (containing all the SQL queries to migrate
     upwards)
   * ```20151120130200-extra-field-down.sql``` (containg all the SQL queries to migrate
     downwards)
2. When you now perform a checkout to master, the post-receive git hook will:
   1. find all *-down.sql scripts in the commits from ```<new HEAD>..<old HEAD>```
   2. execute those scripts with the local database
   3. find all *-up.sql scripts in the commits from ```<old HEAD>..<new HEAD>```
   4. execute those scripts with the local database
3. When you merge your feature branch into master, the post-merge hook will:
   1. find all *-up.sql scripts in the commits from ```master..feature```
   2. execute those scripts with the local database

## Install
Simply copy the post-checkout and/or post-merge file to the .git/hooks
directory of your own git repository. You can edit the config section of those
files. See the files themselves for an explanation.

## Usage
The naming of the migration SQL files is crucial. They should end with
```up.sql``` or ```down.sql```. The rest of the name is completely up to you.
However if you have a single commit with multiple up-migrations and/or multiple
down-migrations the order in which they are performed depends on the
lexicographical order. Migration files that are within different commits, will
always be called in the same (reverse) order as the commits.

It's not a requirement that you have both an up-migration and a down-upgrade,
nor is it a requirement that up- and down-migrations are named similar.

## Warning
Although most data will remain intact when migrating from one branch to
another, some data might get lost. If your feature branch has an extra table
or field, then the complete table or field (including the data you have
inserted in it) will get lost if you checkout a branch which does not have that
table/field. No backup will be made.

A ```git reset``` will change the ```HEAD``` without invoking any hook.
Unfortunately there does not exist any git hook that can be used, so this
project cannot migrate the database when you perform a ```git reset```.

```pre-rebase``` is not implemented yet, so the database will not migrate when
you perform a ```git rebase```.
