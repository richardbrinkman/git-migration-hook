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
