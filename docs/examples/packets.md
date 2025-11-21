# Packet Example – Echo Request/Response

This walkthrough wires a complete request/response packet pair using
`FoxapiPacketHandler`. It demonstrates how to structure packet classes, register
flows and write assertions around the helper queues.

It deliberately mirrors common modding patterns:

* **One class that owns IDs and registrations.** Everything lives under
  `ExamplePackets` so you can scan a single file to understand the mod's
  networking surface.
* **Request/response pairing.** Many features (forms, GUIs, sync signals) follow
  a similar pattern, so this sample doubles as a template for future packets.
* **Asynchronous handlers.** Both sides hop onto the correct executor just like
  Fabric callbacks, keeping your logic thread-safe even in the sandbox.

## Packet Classes

```java
public record EchoRequest(UUID requestId, String payload) implements FoxapiPacket {
    public EchoRequest(PacketByteBuf buf) {
        this(buf.readUuid(), buf.readString());
    }

    @Override
    public void write(PacketByteBuf buf) {
        buf.writeUuid(requestId);
        buf.writeString(payload);
    }
}

public record EchoResponse(UUID requestId, String payload) implements FoxapiPacket {
    public EchoResponse(PacketByteBuf buf) {
        this(buf.readUuid(), buf.readString());
    }

    @Override
    public void write(PacketByteBuf buf) {
        buf.writeUuid(requestId);
        buf.writeString(payload);
    }
}
```

Other patterns that drop straight into `FoxapiPacketHandler`:

* **One-to-many broadcasts.** Swap `sendToPlayer` for `broadcast` to push the
  response to every connected player, or `sendToMatching` to target dimensions
  and permissions.
* **Versioned payloads.** Add an initial `int` version field and switch on it in
  the decoding constructor to maintain backwards compatibility while evolving
  formats.

## Registration

```java
public final class ExamplePackets {
    public static final Identifier ECHO_REQUEST = Foxapi.id("echo_request");
    public static final Identifier ECHO_RESPONSE = Foxapi.id("echo_response");

    private ExamplePackets() {}

    public static void init() {
        FoxapiPacketHandler.registerServerbound(
            ECHO_REQUEST,
            EchoRequest::new,
            ExamplePackets::handleEchoRequest
        );

        FoxapiPacketHandler.registerClientbound(
            ECHO_RESPONSE,
            EchoResponse::new,
            ExamplePackets::handleEchoResponse
        );
    }

    private static void handleEchoRequest(ServerPlayerEntity sender, EchoRequest packet) {
        sender.getServer().execute(() -> {
            FoxapiPacketHandler.sendToPlayer(
                sender,
                ECHO_RESPONSE,
                new EchoResponse(packet.requestId(), packet.payload().toUpperCase())
            );
        });
    }

    private static void handleEchoResponse(MinecraftClient client, EchoResponse packet) {
        client.execute(() -> LOGGER.info("Echo {} -> {}", packet.requestId(), packet.payload()));
    }
}
```

## Exercising the Flow in Tests

```java
@Test
void echoRoundTrip() {
    UUID id = UUID.randomUUID();
    MinecraftServer server = TestFixtures.server();
    ServerPlayerEntity player = TestFixtures.player(server);

    // register once for the test lifecycle
    ExamplePackets.init();

    // simulate the client submitting a request
    FoxapiPacketHandler.sendToServer(ExamplePackets.ECHO_REQUEST, new EchoRequest(id, "hello"));
    FoxapiPacketHandler.flushServer(player);

    // the server handler enqueues a response for the originating player
    FoxapiPacketHandler.flushClient(TestFixtures.client());

    // assert client side state changes
    assertThat(TestLogEvents.messages()).contains("Echo " + id + " -> HELLO");
}
```

Key takeaways:

* **Flush each queue after enqueueing packets.** The helper mirrors the
  client/server separation, so tests should advance both logical sides.
* **Treat handlers as asynchronous.** Wrap side effects in
  `MinecraftServer#execute` or `MinecraftClient#execute` to mimic vanilla
  threading expectations.
* **Keep identifiers centralised.** Static fields on the registration class make
  reuse in tests trivial.
