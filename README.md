# chatcounter

This is a simple Vue application that uses the Twitch API to pick out the most frequently said words/phrases in chat (up to 3 words long). This is intended for use as an OBS Browser Source. Refreshing the page clears data and restarts tracking.

Before running, place your bot/channel information in the `config.js` file in the root directory, replacing the default info:
```js
export default {
  "BOT_USERNAME": "botusername",
  "BOT_TOKEN": "oauth:xxxxxxxxxxxxx",
  "CHANNEL_NAME": "channelname"
}
```

`BOT_USERNAME` is the username of the Twitch account that will be authenticated with. To get the `BOT_TOKEN`, make sure you are signed into the `BOT_USERNAME` account and go to [https://twitchapps.com/tmi/](). `CHANNEL_NAME` is the name of the user whose chat will be scanned.

Then, it's simply `npm run dev` to run the application.
