---
comments: true
date: 2012-05-26 09:27:03
layout: post
slug: custom-rake-tasks-for-dbdrop-dbcreate-dbmigrate-dbseed
title: Custom Rake Tasks for db:drop db:create db:migrate db:seed
wordpress_id: 341
---

Resetting the database in Ruby seems to be quite a painful task if you (or your team) make changes to the seed data / schema. `rake db:drop db:create db:migrate db:seed` is too verbose. Provided below are two custom rake tasks based on Justin French's blog post [here](http://justinfrench.com/notebook/a-custom-rake-task-to-reset-and-seed-your-database):

    
    rake db:rdb (reset database)
    rake db:trdb (reset database in test environment)


The first is equivalent to `rake db:drop db:create db:migrate db:seed`. The second is for test databases and will not perform a seed. It is equivalent to `RAILS_ENV=test rake db:drop db:create db:migrate`. Note that the second tasks actually ignores the current environment and will always use the 'test' environment.

Create a file under `RAILS_ROOT/lib/tasks/db.rake` and paste the code below. The new tasks should both show up with a `rake -T.`

    
    namespace :db do
      desc "Drop, create, migrate then seed the database"
      task :rdb => :environment do
        Rake::Task['db:drop'].invoke
        Rake::Task['db:create'].invoke
        Rake::Task['db:migrate'].invoke
        Rake::Task['db:seed'].invoke
      end
    
      desc "Drop, create, migrate the database with RAILS_ENV=test"
      task :trdb => :environment do
        Rails.env = 'test'
        Rake::Task['db:drop'].invoke
        Rake::Task['db:create'].invoke
        Rake::Task['db:migrate'].invoke
      end
    end
