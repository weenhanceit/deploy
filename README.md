# Deploy

Notes about deploying Rails apps and static web sites to a local physical server using Kamal 2.

## Overview

At the time of writing, Kamal 2 is quite new. I ran into quite a few small but frustrating problems upgrading an existing app to deploy to Kamal, and I believe some of them might be traceable to having generated a number of files in Rails or Kamal versions that weren't current.

So here's the process that I think we want to take to get our three Rails apps, one static site, and on PHP site, to the Docker world.

Well, to start with, if that's the target, how many more GBs am I going to need? All the above seems to running in about 0.5 GB en a single-VM world. The database is running elsewhere, so that's part of the equation.

It sort of looks like we need one user to be the deploy user. It could be good to make a user that can't `sudo` and see if they can run Docker. 

### Draft Plan

1. Build a new Kamal server (Server + Docker) Make it minimal?
2. Build container for static site.
3. `.env` just needs Docker Hub key.
4. Deploy static site and test with `curl`.
5. Change existing Nginx for static site to do a pass-through to Kamal. Does this work?
6. Get cert for statis site. Does this work?
7. Deploy `acedashboards.com`to new server.
8. Change existing Nginx for `acedashboards.com` to pass-through to Kamal. Does this work?
9. Get cert for `acedashboards.com`. Do I need a cert for each site?
10. Upgrade `plazachapina.ca` to Rails 8.
11. Add `/up` if it doesn't already have it.
12. Checkout `production.rb` that it's configured correctly.
13. Work on `deploy.yml`.
14. `.env` needs Docker Hub key and reference to `master.key`.
15. Deploy `plazachapina.ca` to kamal
16. Build a containter for LimeSurvey -- A PHP server and install LimeSurvery. I bet there are volumes that have to be kept across upgrades. Ugh. This will be ugly. There is no official container, but LimeSurvey devs at one time pointed people at a container built by a parner: https://github.com/adamzammit/limesurvey-docker. LimeSurvey people seem to like Apache, so there are a few tweaks needed in the `Dockerfile`. 

## Target Architecture

### Option A

* Kamal app.
* Database accessory.
* Repeat for each app.
* One staiic Nginx app.

Database migration is for each app. Is this good or bad?

### Option B

* Kamal app referencing shared database server.
* Repeat for each app.
* One static Nginx app.

Database migration is for all apps at once. 

Pro: I can get here mor eesily, because I just have to deploy apps and they're pointed at their existing database. 

## Database Migration

One approach would be to add another accessory for the new Postgres version, and add an accessory that upgrades databases. The accessory is to help set up the mirroring of the databases, so the new one has what the old one has. Do some validation, too.

As long as the apps are reasonably up-to-date, I wouldn't expect client version mismatch with server version to be a problem.

(Look into whether Kamal allows aliases for accessories, so that you can just flip the name when you're ready to use the new one.
