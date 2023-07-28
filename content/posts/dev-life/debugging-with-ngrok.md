---
title:  Debugging with Ngrok
date:   2023-07-28 06:00:00
image:  '/images/blog/dev-life/debugging-with-ngrok/preview.jpg'
tags:   [agile, scrum, tools]
---

I've always had a soft spot for Ngrok, a tool that effortlessly exposes local web services to the internet.

Recently, I found myself needing to debug a webhook. A vendor should have been calling our webhook with a payload of event data, but something was off. I couldn't place if it was the content of the payload or the structure of the data. Reviewing production logs was slow and I wasn't exactly sure what I was looking for. I wanted a way of reviewing the payload data just to make sure everything was as expected. 

Ngrok came to mind as useful tool that could help. Stand ngrok up locally, configure the webhook with the ngrok address, and review the logged payload to work out what was going on.

This worked reasonably well, I could just `var_dump` the output in bobby basic php script and hey presto. Which was fine, until I wanted to test how the payload was being consumed by the microservice. Perhaps the problem was there.

Over the last year, I've shifted away from developing using Virtual Box and VMs, I've embraced using Docker whenever possible. The projects I work on are all containerised, and it wasn't immediately obvious how I was going to get Ngrok to play nicely forwarding the ports to the right place.

## Old habits die hard

As I tinkered with a new Dockerfile, installing Ngrok via Apt with custom repositories, etc - it hit me that I was applying old techniques to a modern paradigm.  I was ending up with kludge... which would be fine for a quick debug... but I was curious what the better way was. I needed to step back. A quick google later and I found good examples of how to use Ngrok with Docker Compose, straightforward and elegant.

```yml
version: '3'
services:
  ngrok:
    image: ngrok/ngrok:latest
    command:
      - "start"
      - "--all"
      - "--config"
      - "/etc/ngrok.yml"
    volumes:
      - ./.docker/ngrok/ngrok.yml:/etc/ngrok.yml
    ports:
      - "4040:4040"
```

I've integrated this in to my [boilerplate project](https://github.com/asherawelan/docker-compose-boiler-plate), and it works really well.

## Good outcome, poor practice

Long story short, we found the vendor was in error. Not all events were calling the webhook, as they shoul have been. I would chalk that up as a win for this method of debugging, but I can't really. I was using Ngrok to inspect webhook payloads to gain confidence our code was right. At that point, after three days, I turned my attention to fixing the issue with the vendor.

Had there have been better tests, and good logging - I'd have had certainty from the start that the issue wasn't our code. In that position, with that confidence I could have applied the Sherlock Holmes' principle a lot earlier and engaged the vendor to fix the issue much sooner.

> When you have eliminated all which is impossible then whatever
> remains, however improbable, must be the truth.
> <cite>Sherlock Holmes</cite>


