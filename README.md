# React Native packager issue
## Issue
This repo showcases an issue with React Native packager when reading the `browser` field inside `package.json`.

The issue happens when you are trying to override the `main` file in your `package.json`. 
That is:
```json
"main": "index.js",
"browser": {
  "./index.js": "./stream.js"
}
```

The `index.js` file is not overriden correctly by `stream.js`.

## Setup
No special setup is needed. Just clone the repo, run `npm install` and then `react-native run-ios`

> This issue was tested on MacOS Sierra 10.12.4, using XCode Simulator App v10.0.


## Evidence
This repository uses a `websocket-stream` (v3.3.3) fork to test this situation:
https://github.com/alexleonescalera/websocket-stream

In the `package.json, you can see the `browser` override for the `index.js` file mentioned above.

When you run the app the first time, you will get a **Cannot read property 'property' of undefined** error.
The reason is that `websocket-stream` is accessing code it shouldn't since the `browser` override is not happening
(i.e. it is going to the `index.js` file instead of the `stream.js` file).

The evidence is that, if you create an extra hop to avoid using the `main` file in the `browser` field, the error is gone.

This commit shows that extra hop:
https://github.com/alexleonescalera/websocket-stream/commit/696e6ae

With it, there is no longer a conflict between `main` and `browser` fields:
```json
"main": "index.js",
"browser": {
  "./main.js": "./stream.js"
}
```

You can verify this by doing the following:
1. In `package.json`, change the version of `webstream-socket` to `#browser-hack` (line 25)
    ```json
    "websocket-stream": "git@github.com:alexleonescalera/websocket-stream.git#browser-hack"
    ```
    
2. In console, run `npm update websocket-stream`

3. Refresh the running simulator with `Command + R` or re-run the app with `react-native run-ios`

You will notice that the error is now gone.

> This is just a hack to demonstrate the issue. It is not meant to be a solution.

## Proposed solution
Allow the `main` field to be overriden `browser` settings inside `package.json`.

## Additional notes
This issue has been reported as well in the [MQTT.js](https://github.com/mqttjs/MQTT.js/issues/573#issuecomment-290805156) repository.
