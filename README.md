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

### Session Management

#### Starting a Session

The `startPlaybackSession` function is used to initiate a playback session, and this sends a heartbeat to increase concurrent viewers:

```javascript
import { startPlaybackSession } from "@blendvision/link";

const TOKEN = "YOUR BV PLAYBACK TOKEN";

startPlaybackSession({
  token: TOKEN,
}).then((session) => {
  // Use the session object to configure the player and load content
  console.log("Session started:", session);
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
      console.error("Error ending session:", error);
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
player = await BlendVision.createPlayer("my-player", config);
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
            license: LICENSE_KEY,
            modulesConfig: {
              ["analytics.resource_id"]: session.content.id,
              ["analytics.resource_type"]: session.content.type,
            },
            source: session.sources,
          });

          console.info("Playback", player);
        })
        .then(() => console.info("Playback session success!"))
        .catch((error) =>
          console.error("Playback session failed error", error)
        );
    </script>
  </body>
</html>
```

### License

This project uses the [BlendVision SDK](https://blendvision.com), and you should have proper licenses and tokens to use their services.
