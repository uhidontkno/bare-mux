# Bare-Mux
A system for managing http transports in a project such as [Ultraviolet](https://github.com/Titaniumnetwork-dev/Ultraviolet).

Written to make the job of creating new standards for transporting http data seamless.

Implements the [TompHTTP Bare](https://github.com/tomphttp/specifications/) client interface in a modular way.

Specifically, this is what allows proxies such as [Nebula](https://github.com/NebulaServices/Nebula) to switch HTTP transports seamlessly.

A transport is a module that implements the `BareTransport` interface.
```js
export interface BareTransport {
  init: () => Promise<void>;
  ready: boolean;
  connect: (
    url: URL,
    origin: string,
    protocols: string[],
    requestHeaders: BareHeaders,
    onopen: (protocol: string) => void,
    onmessage: (data: Blob | ArrayBuffer | string) => void,
    onclose: (code: number, reason: string) => void,
    onerror: (error: string) => void,
  ) => [( (data: Blob | ArrayBuffer | string) => void, (code: number, reason: string) => void )] => void;

  request: (
    remote: URL,
    method: string,
    body: BodyInit | null,
    headers: BareHeaders,
    signal: AbortSignal | undefined
  ) => Promise<TransferrableResponse>;

  meta: () => BareMeta
}
```

Examples of transports include [EpoxyTransport](https://github.com/MercuryWorkshop/EpoxyTransport),  [CurlTransport](https://github.com/MercuryWorkshop/CurlTransport), and [Bare-Client](https://github.com/MercuryWorkshop/Bare-as-module3).

Here is an example of using bare-mux:
```js
/// As an end-user
import { BareMuxConnection } from "@mercuryworkshop/bare-mux";
const conn = new BareMuxConnection("/bare-mux/worker.js");
// Set Bare-Client transport
await conn.setTransport(`
    const exports = await import("/bare-mux/index.js");
    return new exports.BareClient("https://tomp.app");
`);

/// As a proxy developer
import { BareClient } from "@mercuryworkshop/bare-mux";
const client = new BareClient();
// Fetch
const resp = await client.fetch("https://example.com");
// Create websocket
const ws = client.createWebSocket("wss://echo.websocket.events");
```