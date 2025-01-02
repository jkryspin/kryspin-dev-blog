---
title: Get notified on Discord when a folders content changes using Nodejs
date: "2019-08-16T18:13:13.650Z"
---

##Background
If you ever find yourself needing to know when a file gets updated or removed from a directory when you're not at home this is the solution for you!

I have a Raspberry PI which I use to download files while I'm not at home, but sometimes I wonder if the PI has crashed or if the files have completed downloading. With this solution I can use my discord on my phone to check what is still downloading vs what is done. For my setup once the file is completed downloading it gets put into a /completed directory automatically. So for this project we are going to watch the /completed directory for changes.

##Prerequisites
- You must have `npm` and `nodejs` installed.
- You must be familiar with running `nodejs` processes. 
- You must be familiar with Discord.


###Installing Dependencies
Install the npm packages `node-watch` and `discord.js`

```terminal
npm install node-watch 
npm install discord.js

```

###Setup
Bring in the modules you just installed and set `DISCORD_READY` to 0. I will explain what `DISCORD_READY` does later on.

You will need a Discord key and you will have to have added the discord bot to your discord server. I will not cover that here, but this is a good guide to follow: https://discordpy.readthedocs.io/en/latest/discord.html.
Once you have your Discord bot setup and authorized to your server come back here for the next steps. 
You should also copy the channel ID of the channel in which you want to post these messages to. You can find that by right clicking the channel in discord while in developer mode

```javascript
const Discord = require('discord.js');
const client = new Discord.Client();
const watch = require('node-watch');
const DISCORD_READY = 0;
const DISCORD_KEY = PASTE_YOUR_KEY_HERE
const DISCORD_CHANNEL = PASTE_YOUR_CHANNEL_ID_HERE
```
##Creating discord functions
Create a function called `connectToDiscord` which will return a promise and handle connecting and logging in to discord. 

This works by first logging in with the `DISCORD_KEY`. After you login you set up an event listener which attaches to the `ready` event. Once discord is ready your tag is printed and the promise reolves.
```javascript
function connectToDiscord() {
    console.log("Connecting to discord")
    return new Promise((resolve, reject) => {
        client.login(DISCORD_KEY);
        client.on('ready', () => {
            console.log(`Logged in as ${client.user.tag}!`);
            resolve()
        });
    })
}
```

The next function is as followed. This fuction returns a boolean which lets us know if we are already connected or not. When client.status is 0 that means discord is ready for you to use the api. That is why we set `DISCORD_READY` to 0. This is a [magic number](https://en.wikipedia.org/wiki/Magic_number_(programming)).
```javascript
function isConnected() {
    return client.status === DISCORD_READY
}
```

##Creating filesystem watcher
Now we are connected to discord, but we need a way to watch for file changes, we have already imported the module above so lets dive into the code.

We set up a watcher on whatever file path you wish. As a second parameter we can pass in an object with recursive true. This will find any changes made no matter how deep the files change in the folder. 

The third parameter is a callback function which executes when something is updated or crated or removed. When this happens we log what changed and run `sendFileChangeMessage(name)`, but first make sure the we are connected to discord.

```javascript
watch(YOUR_FOLDER_PATH_TO_WATCH, { recursive: true }, async function(evt, name) {
    if (!isConnected()) 
        await connectToDiscord();
    console.log('%s changed.', name);
    sendFileChangeMessage(name)
});
```
##Sending message to discord channel
You have everything set up except sending the message to discord!
This part is simple. First we get the channel based on the channel id you provided, and then execute `sendMessage` on that. If all is well, once a file is changed in your directory you will see a message pop up on your discord with the full path of the file changed.
```javascript
async function sendFileChangeMessage(content) {
    try {
        await client.channels.get(String(DISCORD_CHANNEL)).sendMessage(content)
        console.log("Message Sent!")
    }
    catch (err){
        console.log(err)
    }
}
```

##More ideas
- Deploying this onto a rpi
- Embedding more information into the discord message
