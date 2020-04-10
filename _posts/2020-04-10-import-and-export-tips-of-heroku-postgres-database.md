---
layout: post
title: "Import and export tips of Heroku Postgres database"
tags:
- heroku
- postgres
- rails
- TIL
---

I always keep forgetting Heroku's database import and export commands flow. So
here I am writing down that flow for me. I hope it might be useful for other 
Heroku users too.

### 1. Import database from Heroku on the local machine
1 - First, take database backup on the Heroku:

```
heroku pg:backups:capture -a your-app-name
```

2 - Download the database from Heroku:

```
heroku pg:backups:download -a your-app-name
```

The database will get downloaded as the **latest.dump** file on your machine.

3 - Import database in your machine:

```
pg_restore --verbose --clean --no-acl --no-owner -h localhost -U your-username -d your-database-name latest.dump
```

You might not need to pass all those options and try to adjust this command
based on your database settings.


### 2. Export local machine's database on Heroku
There are two steps we can use for exporting the database to the Heroku server.

1. Creating a dump file from the local machine and uploading it to the Heroku server
2. Direct push the local database to Heroku


#### Step 1:
Create a local dump file from using `pg_dump` command:

```
pg_dump -Fc —no-acl —no-owner -h localhost -U your-username your-database-name > your-db.dump
```

Then upload the local dump file to the Heroku server (*I haven't tried this command yet*): 

```
heroku pg:psql -a your-app-name < your-db.dump 
```

#### Step 2:

Push your machine's database directly to the Heroku server:
```
heroku pg:push your-database-name DATABASE_URL -a your-app-name
```

### 3. Extra

To check Heroku's configuration variables:
```
heroku config -a your-app-name
```

To reset the whole database on Heroku if you have the database on Heroku
but you don't care about it:

```
heroku pg:reset -a your-app-name
```

### 4. Reset primary key sequence
After exporting database on Heroku server, you might need to reset 
sequence(auto-increment) on primary key column (or similar column), to solve 
it, you can follow this [blog post](http://blog.joncairns.com/2013/01/reset-postgresql-auto-increment-value-in-rails/).
