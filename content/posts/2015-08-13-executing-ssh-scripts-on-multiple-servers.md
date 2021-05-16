+++
author = "Alex Bouma"
date = 2015-08-13T12:58:00+01:00
draft = false
cover = "/img/ghost/unsplash-highway.jpg"
slug = "executing-ssh-scripts-on-multiple-servers"
tags = ["app", "saas"]
title = "Executing SSH scripts on multiple servers"
aliases = ["/executing-ssh-scripts-on-multiple-servers"]
+++

> Since this article I have bought a special domain for it and rebranded it to: [Server Chief](https://server.chief.app)

I manage a bunch of server, each week I have to SSH in each and every one of them to update packages, check their disk usage and do a bunch of maintenance stuff. This is a lot of work 🙁

So I thought, hey! This can be much easier if I had a tool that would SSH in for me and execute the commands for me and report the output back so I only have to login to servers that have errors and I can easily scan through the output (and look back at earlier executions to see if something has changed).

As it turns out there are a few tools out there (the best I found [Commando.io](https://commando.io/)) but they are all to bulky, too expensive or just not how I wanted it to work.

The one thing led to another and I installed me a fresh [Laravel](http://laravel.com/) instance and started working on said tool.

#### Servers

I started with server management, I needed to add all my servers and the app should be able to logon to the server. I wanted it to be transparant to the user so I decided to generate a public/private key pair for each server to use as credentials so that I did not need to store passwords and users can easilly revoke access without removing the server from the dashboard.

![Server Chief Dashboard](/img/ghost/Screen-Shot-2015-08-13-at-15-24-17-1024x230.jpg)

This was all very straight forward, I pulled in the [phpseclib](https://packagist.org/packages/phpseclib/phpseclib) package to generate a RSA public/private key pair and saved them with a server name, host, user and location to the database (the private key is stored encrypted).

#### Scripts

Then on to the next section, I needed to be able to define scripts, so I could easily start them without typing them each time… makes sense, right 🙂

![Server Chief script editor](/img/ghost/Screen-Shot-2015-08-13-at-15-30-40-1024x516.jpg)

For the script editor I used the [CodeMirror](http://codemirror.net/) editor with the shell syntax highlighting.

#### Execute

Now comes the really interesting (and cool) part of the app. The script execution!

![Server Chief queuing execution](/img/ghost/Screen-Shot-2015-08-13-at-15-34-26-1024x320.jpg)

When you queue an execution you can select the script and on which servers the script should be executed, there is no limit on how many servers at once you can execute, if the app is too busy with other requests the execution simply waits on the queue as the interface will indicate.

![Server Chief job waiting](/img/ghost/Screen-Shot-2015-08-13-at-15-46-11-1024x218.jpg)

When the app starts running the script the interface updates in realtime, showing the server output and the start time, as wel as the execution time when finished and if the script finished successfully or with errors).

![Server Chief successfull execution](/img/ghost/Screen-Shot-2015-08-13-at-15-49-30-1024x793.jpg)

Each execution has it’s own job on the queue so if one server fails the rest continues on just fine untill they are all done. The interface is updates realtime using [Slanger](https://github.com/stevegraham/slanger) a open source server implementation of [Pusher](https://pusher.com/).

#### Cool, now what?

Well I’m not done yet, I would like to <del datetime="2015-08-13T21:41:49+00:00">create server groups</del> (implemented as tags) so I don’t have to manually select which of the many servers the script needs to run. I also would like to have some sort of subscription/<del datetime="2015-08-13T21:41:49+00:00">registration</del> (this is implemented now) so others can use this app too without me making accounts manually.

<del datetime="2015-08-13T21:41:49+00:00">So, no, there is no registration, yet</del> go [here](https://server.chief.io/register). If you are very eager to try (on your own risk ;)) you can register now, please do let me know how it works for you!
