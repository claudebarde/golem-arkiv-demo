# "Where is Claude?" - an Arkiv demo

This app demonstrates how Arkiv can store and stream structured location data on the Kaolin chain.

It reads the latest Arkiv entity created by the admin address and renders the decoded payload on an OpenStreetMap map.

In practice, the app uses Arkiv as a lightweight decentralized data layer where location updates are written as expiring JSON payloads and discovered through Arkiv queries and live events.

## Developing

After installing the dependencies with `bun install`, start a development server:

```sh
bun run dev

# or start the server and open the app in a new browser tab
bun run dev -- --open
```

## Building

To create a production version of your app:

```sh
bun run build
```

You can preview the production build with `bun run preview`.

> To deploy your app, you may need to install an [adapter](https://svelte.dev/docs/kit/adapters) for your target environment.

## General Experience with Arkiv

The dev experience with Arkiv is generally good.

The TypeScript SDK handles many of the low-level details, for example, it will forge the transaction to create a new entity and handle all the different aspects of it. The SDK is self-understandable, and there seem to be very few quirks to learn that are inherent in learning a new tooling suite.

A lot of the code you write is similar to what you may have worked with before: Arkiv uses MetaMask as a wallet to create transactions, the data querying is done with keywords that are used by web2 databases (`where`, `eq`, `gte`, etc), so you can get up and running very fast.

The only issues I had were not due to the SDK, but to other elements in the ecosystem. First, I was unable to obtain test tokens from the faucet and left a message in the Discord channel. The tokens were transferred to my account, although the faucet displayed an error message.  
During development, I also got an issue with the RPC endpoint: after creating a new entity, it would lag for 30 seconds or so before displaying an error message in the browser console. However, the data had been correctly created.

I believe more example apps would be beneficial to help developers start building. This app is rather simple, but more complex examples could demonstrate the full potential of Arkiv. For example, I would have liked to see more examples of query buildings.
