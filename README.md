# TurboStorage

A high-performance blob storage abstraction for .NET built on [Akka.Streams](https://getakka.net/).

## Features

- **Akka.Streams-first API** — `Source<ReadOnlyMemory<byte>>` / `Sink<ReadOnlyMemory<byte>>` with full backpressure
- **Immutable data model** — sealed records for all types
- **Multiple providers** — Local filesystem, in-memory, virtual (mount-based routing)
- **Composable** — plug Akka.Streams operators (throttle, buffer, map) directly into storage pipelines

## Quick Start

```csharp
using Akka.Actor;
using Akka.Streams;
using Akka.Streams.Dsl;
using TurboStorage.Local;

using var system = ActorSystem.Create("my-app");
var materializer = system.Materializer();

var store = new LocalBlobStore("/path/to/storage");

// Write
var data = "hello world"u8.ToArray();
var writeResult = await Source.Single(new ReadOnlyMemory<byte>(data))
    .RunWith(store.Write("greeting.txt"), materializer);

// Read
var (readResult, content) = await store.Read("greeting.txt")
    .ToMaterialized(Sink.Seq<ReadOnlyMemory<byte>>(), Keep.Both)
    .Run(materializer);

var metadata = await readResult;
Console.WriteLine($"Read {metadata.Size} bytes from {metadata.Path}");
```

## Providers

| Package | Provider |
|---|---|
| `TurboStorage` | Local filesystem, In-memory, Virtual (mount routing) |

## License

[MIT](LICENSE)
