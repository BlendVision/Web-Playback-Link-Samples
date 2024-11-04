# BlendVision Link SDK

This project provided BlendVision Link SDK, and you can use `startPlaybackSession` from the `@blendvision/link` package.

## BlendVision Player Integration

This project demonstrates how to integrate the **BlendVision Player** with the **BlendVision Link SDK** using npm.

## Getting Started

To use BlendVision's video playback features, you'll need two things:

1. A valid **BV Playback Token**.
2. A valid **BV Player License Key**.

### Installation

To include the necessary libraries via npm, run:

```bash
npm install @blendvision/link @blendvision/player
```

## `startPlaybackSession`

The `startPlaybackSession` function is used to initiate a playback session, and this sends a heartbeat to increase concurrent viewers:

```javascript
import { startPlaybackSession } from "@blendvision/link";

const TOKEN = "YOUR BV PLAYBACK TOKEN";

startPlaybackSession({
  token: TOKEN,
})
  .then((session) => {
    // Use the session object to configure the player and load content
    console.log("Session started:", session);
  })
  .catch((error) => {
    console.error("Session start failed:", error);
  });
```

This function returns a Promise that resolves with a session object containing information about the content and playback sources.

#### Ending a Session

When you're done with the playback session, it's important to end it properly, and it stops the heartbeat to decrease concurrent viewers:

```javascript
function endPlaybackSession() {
  session
    .end()
    .then(() => {
      console.log("Session ended successfully");
    })
    .catch((error) => {
      console.error("Session ended failed:", error);
    });
}
```

Call this function when the user is finished watching the content or navigating away from the page.

### Cleanup

When you're done with the playback session, you can clean up resources as follows:

```javascript
// Clean up resources when done
function onExit() {
  player.unload();
  session.end();
}
```

### Configuration

```javascript
const config = {
  licenseKey: "player license key",
  source: [
    {
      type: "application/dash+xml",
      src: "Your_DASH_URL",
    },
    {
      type: "application/x-mpegurl",
      src: "Your_HLS_URL",
    },
  ],
  modulesConfig: {
    ["analytics.resource_id"]: session.content.id,
    ["analytics.resource_type"]: session.content.type,
  },
};
player = BlendVision.createPlayer("my-player", config);
```

- **`token`**: The playback token that authenticates the session.
- **`license`**: The player license key required for player initialization.
- **`analytics.resource_id`**: Configures analytics for tracking content.
- **`analytics.resource_type`**: Specifies the type of content (e.g., video, event).

### HTML Markup Example

Ensure that the player has a container in your HTML:

```html
<!DOCTYPE html>
<html lang="en">
  ...rest code
  <body>
    <div id="my-player"></div>
    <button id="exitButton">Exit Player</button>
    <script type="module">
      import "https://unpkg.com/@blendvision/link@version";
      import "https://unpkg.com/@blendvision/player@version";

      const TOKEN = "YOUR BV PLAYBACK TOKEN";
      const LICENSE_KEY = "YOUR BV PLAYER LICENSE KEY";

      let player;
      let playbackSession;

      BlendVisionLinkSDK.startPlaybackSession({
        token: TOKEN,
      })
        .then(async (session) => {
          console.info("Playback session started", session);

          playbackSession = session;

          player = await BlendVision.createPlayer("my-player", {
            licenseKey: LICENSE_KEY,
            modulesConfig: {
              ["analytics.resource_id"]: session.content.id,
              ["analytics.resource_type"]: session.content.type,
            },
            source: session.sources,
            onPlaylogFired: (event) => {
              // Handle playback_video_ended event, ex: live ended
              if (event === "playback_video_ended") {
                console.log("Video playback ended, ending session...");
                playbackSession.end();
              }
            },
          });

          console.info("Playback", player);
        })
        .then(() => console.info("Playback session success!"))
        .catch((error) =>
          console.error("Playback session failed error", error)
        );

      // Cleanup function
      function onExit() {
        if (player) {
          player.unload();
        }
        if (playbackSession) {
          playbackSession
            .end()
            .then(() => console.log("Session ended successfully"))
            .catch((error) => console.error("Failed to end session:", error));
        }
      }

      // Add event listener for beforeunload
      window.addEventListener("beforeunload", onExit);

      // Add event listener for UI exit button
      document.getElementById("exitButton").addEventListener("click", () => {
        onExit();
        console.log("Player exited via UI");
        // Additional UI updates or redirects can be added here
      });
    </script>
  </body>
</html>
```

## `createPlayer`

### Overview

The `createPlayer` function initializes a new instance of the BlendVision player, providing control over playback settings, event handling, and custom configurations through `BlendVisionLinkSDK`. It accepts a configuration object that allows for session authentication, license management, and event logging.

### Usage

Here’s a basic setup to create a player instance with `BlendVisionLinkSDK.createPlayer`, using essential configuration properties:

```javascript
<script type="module">
  import "https://unpkg.com/@blendvision/link@version";

  const TOKEN = 'YOUR_BV_PLAYBACK_TOKEN';
  const LICENSE_KEY = 'YOUR_BV_PLAYER_LICENSE_KEY';

  const config = {
    licenseKey: LICENSE_KEY,
    onPlaylogFired: (event, data) => {
      console.info("Playback event:", event, data);
    },
  };

  let player;
  player = await BlendVisionLinkSDK.createPlayer({
    playbackSession: {
      token: TOKEN,
    },
    playerOptions: {
      id: "my-player",
      config,
    },
  });
</script>
```

### Parameters

- **playbackSession**: Contains session-specific data like `token` for authenticating the playback session.

  - `token`: A playback token provided by BlendVision for secure access.

- **playerOptions**: Defines the options for the player instance.
  - `id`: The HTML element ID for the player container.
  - `config`: Configuration options, including `licenseKey` and event handlers.

### Usage Example

The following example demonstrates how to create a player instance using `BlendVisionLinkSDK.createPlayer`, and then shows basic playback controls like `play()` and `pause()`.

```html
<script type="module">
  ...rest code

    let player;
      player = await BlendVisionLinkSDK.createPlayer({
        playbackSession: {
          token: TOKEN,
        },
        playerOptions: {
          id: "my-player",
          config,
        },
      });


      // Example of basic playback controls
      document.getElementById("playButton").addEventListener("click", () => {
        player.play();
      });

      document.getElementById("pauseButton").addEventListener("click", () => {
        player.pause();
      });

      // Stop the current video playback and free up resources related to the player.
      document.getElementById("releaseButton").addEventListener("click", () => {
        player.unload();
      });
      // Reload the same or updated content after a reset or release.
      document.getElementById("reloadButton").addEventListener("click", () => {
        player.load({ id: "my-player", token: TOKEN });
      });

      // Add event listeners for playback events or use onPlaylogFired callback in config.
      player.addEventListener("play", () => {
        console.log("video is playing");
      });

      player.addEventListener("pause", () => {
        console.log("video is paused");
      });

      player.addEventListener("ended", () => {
        console.log("video is ended");
      });
</script>
```

### License This project uses the [BlendVision SDK](https://blendvision.com),

and you should have proper licenses and tokens to use their services.
