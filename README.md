<a href="https://github.com/sponsors/gi-os"><img src="https://raw.githubusercontent.com/gi-os/gi-os/main/images/bright-sponsors.svg" width="100%" alt="Thank you to my sponsors"></a>

# BrightNYCSubway Lite

> **This is the Lite version.** BrightNYCSubway Lite is built only with Light's official SDK, so it can go in the Light Tool Library. It does less than the full app. For the full app, get [BrightTransit](https://github.com/gi-os/BrightTransit) ([on BrightMarket](https://brightmarket.gzl.dev/app/com.thelightphone.lightnycsubway/)).

Live New York City subway arrivals for the Light Phone III, built with the official LightOS SDK for the Light Tool Library.

It reads the MTA's public GTFS-realtime feeds straight from the phone. There is no server in the middle and no API key. It opens on your saved station with Uptown and Downtown side by side, soonest train first.

## How this relates to BrightTransit

The code comes from [gi-os/BrightTransit](https://github.com/gi-os/BrightTransit), the sideloaded build ("Subway Times"). There the tool sits in `examples/LightNYCSubway` with the id `com.thelightphone.lightnycsubway`. Light's builder only builds the `tool/` module, and a Tool Library tool needs an id outside Light's namespace. So here the tool is in `tool/` with the id `com.gios.brightnycsubwaylite`. Both can be installed side by side.

## Data

The phone talks to the MTA feeds and nothing else. Your saved station stays on the phone.

## Build

```sh
./gradlew -DlightSdk.toolOnly=true :tool:assembleDebug
```

Built on [lightphone/light-sdk](https://github.com/lightphone/light-sdk) (MIT).
