---
layout: post
title:  "What Could Go Wrong Setting Up PostgreSQL?"
date:   2022-03-08 09:40:54 -0400
categories: data-ish databases postgres /#WCGW
---

So a rare data-ish post not in [Deepnote](https://deepnote.com/@sia-seko/){:target="_blank"}. STORY TIME!!

The TLDR version of this story is we're writing this down so we don't forget what to do like we did this time. I've decided to write a what could go wrong edition (#WCGW) everytime I get stuck doing something software-y so here goes number one.

Yesterday, I wanted to write a bash script to backup and restore a PostgreSQL database instance. Part of my process included wanting to test this out on a local server (because we don't want to make a mess). What should have been an hour at most, took me about three because though I'm well versed in SQL databases, we only ever have to set them up once. There is not a lot of practice in setup when the infrastructure is stable, and it has been a good year since I've had to do it.

**The goal here** is to install and set up a local PostgreSQL instance and use pgAdmin to manage resources.

## A series of mishaps

First I'll show you the errors along the way, and then the whole process from scratch to success! Buckle up.

### First, pgAdmin

To run the backup and restore test, all I really needed was postgres installation. However in my haste, we installed [pgAdmin](https://www.pgadmin.org/download/){:target="_blank"} without installing [PostgreSQL](https://www.postgresql.org/download/windows/){:target="_blank"}. In other words, I was trying to start a car without its engine. I connected to the management tools, but there was no postgres server to add or use.

![what pgAdmin looks like without a postgres instance for a server](/assets/img/no_postgres_server.PNG)

**The fix** here was to download postgres. So step 1 - download the installation packages for both postgres AND pgAdmin, duh!! Oh so I thought...

### Then the pgAdmin versions

You know when you are running an executable file (using a Windows PC, I should have mentioned) and your computer is slow and old so you keep clicking, maybe download a different version of the file, and now both versions are being installed? No? Just me? Well then. You see, `pgAdmin 4` has several stable versions - two of which are `v4` and `v6`. EYE downloaded and installed both while being impatient. This distinction is important to note because though both are compatible with the `postgres` installation, some features may not work. And guess what is part of the some features collective - backup and restore.

**The lesson here** is to install the latest stable version of pgAdmin to ensure you have all features that you'll need to use.

### ...and permissions

Now at this point, I didn't get to the part where it failed yet. I'm just installing and running and executing and loading my dummy data from a file to test. Then we try this:

```sh
psql pg_dump -U postgres -h localhost -p 5432 db_name
```

PostgreSQL wasn't having it.

```sh
Permission denied. Press any key to continue...
```

This error message appeared. Permission means access restrictions, so to fix that, we gave the `postgres` user that was set up while installing `PostgreSQL`. The schema my database was in is `public`, so we did the following to make sure this user is allowed to make changes to the databases within.

```sql
-- be in pgAdmin, on a SQL script file
-- enter and run this
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO postgres;
```

### And back to versions clashing

Now when the access was set, (the Royal) we tried this again:

```sh
psql pg_dump -U postgres -h localhost -p 5432 db_name
```

This time, we got further! Verified inputs :check:, entered password :check:, aaaaand, got this error.

```sh
pg_dump: server version: 14.2; pg_dump version: 12.4
pg_dump: aborting because of server version mismatch
```

Also sidebar - what luck how similar those versions are :smile:. Anywho with this error, there are two options off the top:

- Change the pg_dump version (to do with `postgres` version installed)
- Change the server version (to do with `pgAdmin` version installed)

This is where my brain went:

- I just installed the most recent postgres version so if anything is not compatible, it has to do with the server configuration. Therefore, look at pgAdmin
- I check the version for pgAdmin on my PC, and guess what - there are two versions, both installed, both functioning.
- Checking the running instance tells me I'm on `v4` which is less updated than `v6`. So (the royal) we shut that down, and start up `v6` pgAdmin, add postgres server and user, and try to `pg_dump` again. THIS TIME IT WORKS!!

Nothing beyond here is interesting, so at this point, we test our commands, add them to the script and test that and we're all done. So for the fun part, if you came here to understand how to set up your local postgres instance to do some database actions, here is the A to zed. Or like Americans say, zee :smile:.

... COMING SOON after edits

![A spongebob meme saying one month later](/assets/img/one_month_later_ss.jpg)

March was long, so here we are... But we made it back :tada:

### Once you have walked through the thorns... now the setup

With all the above breadcrumb moments in mind, let's proceed!

*Note that this is a high level overview that doesn't go into every detail of all the ways you can install PostgreSQL and use it with pgAdmin. The goal is to point out pitfalls, but I'm happy to answer any questions that were not covered in this scenario.*

1. pgAdmin can be installed as a standalone application, or as part of the postgreSQL installation. In this walkthrough, we'll do both. Since we're making an assumption here (well I am) that you haven't installed any of these applications on your machine, let's skip to step 2 and install them together!

2. To be able to add a server on the pgAdmin screen we're on, we need to actually have a postgres instance to attach! Now let's download [a postgreSQL `exe` file](https://www.postgresql.org/download/windows/){:target="_blank"}. Run that to install. As you follow the prompts, you'll notice something here.
    ![you can install pgAdmin along with postgreSQL in this screenshot where you can select what to install](/assets/img/postgresSQL_install.PNG)

    You have the option to install pgAdmin along with other components, so we will keep that. If for some reason, you have pgAdmin already installed, uncheck that box and move on. This may take a while so get some tea or water, stretch, alladat.

3. Open pgAdmin after installing it. It may take a few minutes to load, but looks like this once done. You will notice that there are no servers or databases connected if this is your first time. We want to attach our postgreSQL instance to a postgres server.

    - **An error you might encounter here** is `Failed to Configure`. What has worked for me in the past is unchecking the `Fixed Port Number` or changing it to a free port.
    - Once you have it open, you will see a screen like the one displayed earlier, but with a prompt to create (for first time users) a password. Create a password and remember it. You will need it later.
    ![pgAdmin when there are no servers attached and password creation](/assets/img/pg_admin_pwsetup.PNG)

4. On this screen, you may find that `PostgreSQL 14` is already present as a user. When you click and try to open this user instance, you will be prompted to enter a password. This is ***different*** from the password you just created for pgAdmin. Create (and remember) this password too. This is the password for the DB user of a postgres instance, not the pgAdmin server management system as a whole.

    ![GIF hopefully showing how you are prompted to enter yet another password for postgres user](/assets/img/postgres_user_login.gif)

    - You may be in a situation where you're being prompted to enter a password that presumably exists. It could be that you had some older version of postgres user on your computer or something of the like. [This user on YouTube](https://www.youtube.com/watch?v=5dLN98y-n6c&t=91s){:target="_blank"} has created a great video that walks you through resetting your password and re-entering it to use!

5. At this point, you will actually have two options for how to use your server instance to create databases, schemas, and tables.

- Command line interface with psql (you can search `psql` on your start menu or spotlight on a Mac and a terminal that is designed for your postgreSQL usage will appear)
- On pgAdmin with the guidance of controls and a user interface. This was the goal of sharing! Instead of using your CLI, you can do all the things you want on the great pgAdmin interface and not remember all the CLI commands.

I hope you struggle a little less from knowing some of the issues that you could encounter here. This is written as a back and forth to illustrate how the troubleshooting went. Curious to hear if your brain got what was going on with this order!

Antyway, Dawn Staley just won a National Championship so time to go celebrate :tada:

![a GIF of Dawn Staley dancing as she always does in her drip, I really hope you see this one](/assets/img/dawn_staley_dancing.gif)
