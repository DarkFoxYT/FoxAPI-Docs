# Networking Helper

FoxAPI ships with `org.dark.foxapi.systems.helpers.network.FoxapiPacketHandler`, a
self-contained sandbox for registering and exercising custom packets while you
prototype features in the editor suite. The helper mirrors Fabric's
`ClientPlayNetworking`/`ServerPlayNetworking` APIs but runs entirely in-memory,
letting you verify serialisation logic without a running game instance.

If you just want to get moving, drop this snippet into a test mod and tweak the
payload type:

```java
public final class QuickstartPackets {
    public static final Identifier ECHO = Foxapi.id("echo");

    public static void init() {
        FoxapiPacketHandler.registerBidirectional(
            ECHO,
            EchoPacket::new, QuickstartPackets::handleClient,
            EchoPacket::new, QuickstartPackets::handleServer
        );
    }

    private static void handleClient(MinecraftClient client, EchoPacket packet) {
        client.execute(() -> LOGGER.info("client received {}", packet.message()));
    }

    private static void handleServer(ServerPlayerEntity player, EchoPacket packet) {
        player.getServer().execute(() -> LOGGER.info("server received {}", packet.message()));
    }
}
```

Run `FoxapiPacketHandler.sendToServer(ECHO, new EchoPacket("ping"))` then flush
both queues (see below) and you have a complete round trip without ever opening
Minecraft.

## Defining a Packet

Implement [`FoxapiPacket`](../src/main/java/org/dark/foxapi/systems/helpers/network/FoxapiPacket.java)
for every logical packet shape you need. The interface requires a single
`write(PacketByteBuf buf)` method that serialises the packet's payload.

**Lifecycle at a glance**

1. `write` runs on the *sending* side when you enqueue a packet.
2. The handler's factory (often the decoding constructor) runs on the *receiving*
   side when the queue is flushed or `dispatch*` is invoked.
3. The handler receives the deserialised packet and the logical executor
   (`MinecraftClient` or `ServerPlayerEntity`) so you can mirror vanilla thread
   usage.

```java
public record EchoPacket(String message) implements FoxapiPacket {
    public EchoPacket(PacketByteBuf buf) {
        this(buf.readString());
    }

    @Override
    public void write(PacketByteBuf buf) {
        buf.writeString(message);
    }
}
```

Patterns that work well:

* Prefer Java records for small data holders – the canonical constructor and
  accessor generation keeps the class tiny.
* Always supply a constructor that accepts a `PacketByteBuf`; the factories you
  register later will call it when decoding a payload.
* Implement `write` symmetrically with the decoding constructor. The sandbox
  copies buffers for you, so there is no need to retain references.
* Co-locate identifiers and packet classes in a single `Packets` utility so
  modders can grep one place to understand the full networking surface.

## Registering Packet Flows

Use `FoxapiPacketHandler.registration(id)` to build up one or both logical
flows.

```java
public static final Identifier ECHO_ID = Foxapi.id("echo");

public static void registerPackets() {
    FoxapiPacketHandler.registration(ECHO_ID)
        .clientbound(EchoPacket::new, ExamplePackets::handleClient)
        .serverbound(EchoPacket::new, ExamplePackets::handleServer)
        .register();
}
```

Each flow requires two callbacks:

* **Factory** – a function that accepts a `PacketByteBuf` and returns your packet
  type (usually a constructor reference).
* **Handler** – code that executes on the logical side when the packet arrives.
  Client handlers receive a `MinecraftClient`; server handlers receive the
  `ServerPlayerEntity` that sent the packet.

Shortcuts exist when you only need one flow:

```java
FoxapiPacketHandler.registerClientbound(ECHO_ID, EchoPacket::new, ExamplePackets::handleClient);
FoxapiPacketHandler.registerServerbound(ECHO_ID, EchoPacket::new, ExamplePackets::handleServer);
```

You can register multiple builders at once with `registerAll(builderA, builderB)`
if you want to share factories across packet IDs.

### Registration Checklist

* **Unique identifier** – keep IDs stable and namespaced via `Foxapi.id`.
* **Factory/handler pairs** – both must be non-null per flow or registration will
  fail fast.
* **One-time setup** – call your `register*` methods from a static `init`
  invoked by your test harness or mod initialiser. Re-registration of the same
  flow throws to protect you from silently swapping handlers.
* **Optional bidirectional asymmetry** – `registerBidirectional` allows
  different factories/handlers per direction when payloads diverge between
  client and server.

## Sending and Dispatching Packets

Once registered, use the helper to exercise your logic:

```java
FoxapiPacketHandler.sendToServer(ECHO_ID, new EchoPacket("ping"));
FoxapiPacketHandler.sendToPlayer(player, ECHO_ID, new EchoPacket("hello"));
FoxapiPacketHandler.broadcast(server, ECHO_ID, new EchoPacket("global"));
FoxapiPacketHandler.sendToMatching(server, ECHO_ID, new EchoPacket("dimension"),
    p -> p.getWorld().getRegistryKey().equals(World.END));
```

Queued packets are processed when you explicitly flush the relevant side:

```java
FoxapiPacketHandler.flushServer(player);
FoxapiPacketHandler.flushServer(server);
FoxapiPacketHandler.flushClient(MinecraftClient.getInstance());
```

For immediate execution – for example inside a test – call
`dispatchServer`/`dispatchClient` with a pre-filled `PacketByteBuf`.

### Tips for Modders

* **Mirror vanilla threading.** Use `execute` on the provided client/server
  objects if your handler touches game state. That keeps your test logic aligned
  with how Fabric/vanilla invoke networking callbacks.
* **Leverage factories for compatibility.** If you evolve packet formats, add a
  second factory that can parse the new payload and branch inside the handler
  until you can drop the older variant.
* **Cross-link with editor features.** Because the helper is sandboxed, you can
  freely log, mutate fake player state or assert against queued packets without
  worrying about protocol spam.
* **Use `registerAll` for suites.** Group packet registrations by feature and
  register them together in tests, mirroring how a real mod initialises its
  networking module.

## Example Walkthrough

The [`docs/examples/packets.md`](examples/packets.md) guide contains a complete
sample that registers an echo packet, exercises both flows and outlines how to
structure integration tests around the helper. Use it as a template for
packets that respond to players, emit server broadcasts or chain multiple
messages together.

## Troubleshooting

* **`IllegalStateException` when sending** – ensure the packet ID is registered
  for the flow you are targeting. The helper validates this before writing to
  the queue.
* **Handlers never run** – remember to call `flushServer`/`flushClient` in your
  tests. The queues decouple writing from dispatch.
* **Buffer leaks** – if you copy buffers manually, release them after use. The
  helper already copies payloads for you, so constructors should not stash
  references to the incoming `PacketByteBuf`.
