# Getting started

::: danger
this lavalink wrapper libary will dont have backwards compatible with lavalink v3
:::

## Installation

Let's start installing dependencies!

```sh
npm install sakulink
```

```sh
yarn add sakulink
```

```sh
pnpm add sakulink
```

```sh
bun add sakulink
```

## Create your first client

```ts
// import both libraries
import { Client } from "discord.js";
import { Manager } from "sakulink";

// Initiate both main classes
const client = new Client({ intents: 33409 });

// Define some options for the node
const nodes = [
  {
    identifier: "JIRAYU",
    host: "lavalink.jirayu.net",
    password: "youshallnotpass",
    port: 2353,
    secure: false
    retryAmount: Infinity,
    retryDelay: 50,
    search: true,
    playback: true,
  }
];

// Assign Manager to the client variable
client.manager = new Manager({
  shards: client.shard?.count ?? 1,
  autoPlay: true,
  defaultSearchPlatform: "youtube music",

  // The nodes to connect to, optional if using default lavalink options
  nodes: nodes,
  // Method to send voice data to Discord
  send: (id, payload) => {
    const guild = client.guilds.cache.get(id);
    if (guild) guild.shard.send(payload);
  }
});

// Emitted whenever a node connects
client.manager.on("nodeConnect", node => {
  console.log(`Node "${node.options.identifier}" connected.`)
})

// Emitted whenever a node encountered an error
client.manager.on("nodeError", (node, error) => {
  console.log(`Node "${node.options.identifier}" encountered an error: ${error.message}.`)
})

// Listen for when the client becomes ready
client.once("ready", () => {
  // Initiates the manager and connects to all the nodes
  client.manager.init(client.user.id);
  console.log(`Logged in as ${client.user.tag}`);
});

// THIS IS REQUIRED. Send raw events to Sakulink
client.on("raw", d => client.manager.updateVoiceState(d));

// Finally login at the END of your code
client.login("your bot token here");
```

## Create your first command

```ts
// Add the previous code block to this

client.on("messageCreate", async (message) => {
  // Some checks to see if it's a valid message
  if (!message.content.startsWith("!") || !message.guild || message.author.bot)
    return;

  // Get the command name and arguments
  const [command, ...args] = message.content.slice(1).split(/\s+/g);

  // Check if it's the play command
  if (command === "play") {
    if (!message.member.voice.channel)
      return message.reply("you need to join a voice channel.");
    if (!args.length)
      return message.reply("you need to give me a URL or a search term.");

    const search = args.join(" ");

    let res;

    try {
      // Search for tracks using a query or url, using a query searches youtube automatically and the track requester object
      res = await client.manager.search(search, message.author);
      // Check the load type as this command is not that advanced for basics
      if (res.loadType === "empty") throw res.exception;
      if (res.loadType === "playlist") {
        throw { message: "Playlists are not supported with this command." };
      }
    } catch (err) {
      return message.reply(
        `there was an error while searching: ${err.message}`
      );
    }

    if (res.loadType === "error") {
      return message.reply("there was no tracks found with that query.");
    }

    // Create the player
    const player = client.manager.create({
      guild: message.guild.id,
      voiceChannel: message.member.voice.channel.id,
      textChannel: message.channel.id,
      volume: 80,
      selfMute: false,
      selfDeafen: true,
    });

    // Connect to the voice channel and add the track to the queue
    if (playrer.state !== "CONNECTED") player.connect();
    player.queue.add(res.tracks[0]);

    // Checks if the client should play the track if it's the first one added
    if (!player.playing && !player.paused) player.play();

    return message.reply(`enqueuing ${res.tracks[0].title}.`);
  }
});
```

## Emit your first event

```ts
client.manager = new Manager(/* options above */)
  // Chain it off of the Manager instance
  // Emitted when a node connects
  .on("nodeConnect", (node) =>
    console.log(`Node "${node.options.identifier}" connected.`)
  );

// Emitted when a track starts
client.manager.on("trackStart", (player, track) => {
  const channel = client.channels.cache.get(player.textChannel);
  channel.send(
    `Now playing: \`${track.title}\`, requested by \`${track.requester.username}\`.`
  );
});

// Emitted the player queue ends
client.manager.on("queueEnd", (player) => {
  const channel = client.channels.cache.get(player.textChannel);
  channel.send("Queue has ended.");
  player.destroy();
});
```

## Want more example?

Check out the [example](https://github.com/JirayuSrisawat-Github/sakulink-example)

## Used by

| Client  | Creator     |
| ------- | ----------- |
| Matsuri | JIRAYU      |
| SaLee   | SomboyTiger |
| Yuzu    | Slamy       |
