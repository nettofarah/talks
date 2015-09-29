layout: true
.logo[![IFTTT](../ifttt.svg)]

---
class: middle, main

# If This Then .blue[Deploy]
## a look into IFTTT's infrastructure

.right[@nettofarah]

---
class: fake-middle

# nettofarah

## github.com/nettofarah
## @nettofarah

---
class: fake-middle
# Disclaimer
I'm not a DevOps guy!

---
class: fake-middle

# Senior Product Engineer @ IFTTT

---
# Background

- Millions of Background Jobs every day
- Over a Billion Events every day
- 215 (and growing) partner integrations

---
class: fake-middle
# a Rails Monolith

---
class: fake-middle
# How did it all start?

---
# Rails Monolith App

- Great for validating ideas
- Solid tools for dev and ops

---
class: fake-middle
# Cron Jobs

---
class: fake-middle
# Deploys in Rails

---
class: fake-middle
# Capistrano

---
# Capistrano is AWESOME!

- works for web apps
- works really well for background jobs too

---
class: fake-middle
# It works amazing

until you have to deploy it to a couple of hundred machines

---
# Consequences

- Time outs
- Long Deploy Cycle
- Complex deploy code
- Simultaneous Restarts can kill you!

---
# Provisioning with Chef

- Really good tool
- But can get complex over time

---
class: fake-middle
# Personal opinion on Chef

I have a really hard time figuring out where files should live.

---
class: fake-middle
# Maybe there's a design problem here

---
class: fake-middle
# That's probably true

But we're a startup. We have limited resources and need to move fast.

---
class: fake-middle
# A Solution for Deploying to hundreds of servers
With Capistrano

---
class: fake-middle
# Deploynamo

Async Deploys

---
# A Case for Micro Services

- Teams are growing fast
- The product and userbase are growing even faster
- Old code lurking around from years ago

---
class: fake-middle
# We can take out some important parts of the system

---
class: fake-middle
# And turn them into Micro Services

Many different languages and datastores.
Most of them deployed to Heroku.

---
class: fake-middle
We love Heroku!

---
class: fake-middle
# Docker comes into the picture

---
class: fake-middle
# What if we could...

---
class: fake-middle

# deploy our services + app
to a more homogenous infrastructure?

---
class: fake-middle
# Docker + Mesos + Marathon + Chronos

(Not live in production yet)

---
class: fake-middle
# Multiple containers with different roles working together

In the same infrastructure

---
class: fake-middle
# We can now tweak our services and have more control over our deployment process

---
# Of course it comes at a cost

- paradigm shift
- complexity

---
# The Future

- Everything is a Container
- Images are built automatically
- New Services are just a new Docker image dropped onto Mesos

---
class: fake-middle
# Questions
