# Connected Texture Generation Helper

FoxAPI's data-driven helper now exposes a dedicated builder for connected
textures. The builder focuses on three goals:

1. **Mod compatibility** – choose the loader namespace (`fabric`,
   `continuity`, `ctm`, or any custom identifier) without duplicating JSON
   boilerplate.
2. **Strategy expressiveness** – pick from built-in helpers like
   `tiled(width, height)`, or define custom strategies with
   `ConnectedTextureStrategy.custom()`.
3. **Developer ergonomics** – compose definitions fluently, attach
   conditions, and write everything out through the `DataDrivenHelper`
   bundler alongside models and blockstates.

## Quickstart: tiled textures for multiple loaders

```java
Identifier sprite = Identifier.of("example", "block/marble");
Path output = Path.of("src/generated/resources/assets/example/connected_textures/marble.json");

DataDrivenHelper.builder()
    .addConnectedTexture(output, sprite, builder -> builder
            .strategy(ConnectedTextureStrategy.tiled(3, 3))
            .config("tiles", 16)
            .when("block", Identifier.of("example", "marble_block"))
            .continuityLoader())
    .build(payload -> payload.forEach((path, json) -> {
        try {
            DataDrivenHelper.write(path, json);
        } catch (IOException exception) {
            LOGGER.error("Failed to write {}", path, exception); // Replace LOGGER with your mod logger.
        }
    }));
```

This snippet:

* Targets the Continuity loader (`continuity:connected_textures`).
* Uses the built-in tiled strategy and tweaks its configuration.
* Restricts the rule to a specific block identifier via `when(...)`.
* Uses the standard `DataDrivenHelper` bundler so connected textures can be
  written alongside models, blockstates, and item definitions.

## Building bespoke strategies

Some loader implementations expose strategy keys that FoxAPI does not ship by
default. Use the new `ConnectedTextureStrategy.custom(key, configConsumer)`
factory to bridge that gap:

```java
ConnectedTextureStrategy glowStrategy = ConnectedTextureStrategy.custom(
        "custom_glow",
        json -> json.addProperty("fade", 4)
);

JsonObject definition = DataDrivenHelper.connectedTexture()
        .sprite(Identifier.of("example", "block/glowstone"))
        .loader(Identifier.of("ctm", "connected_textures"))
        .strategy(glowStrategy)
        .config("emit_light", true)
        .whenIn("facing", "north", "south", "east", "west")
        .build();
```

The resulting JSON contains the custom strategy key, merged configuration
values, and a condition limiting it to horizontal faces.

## Direct root and condition access

Advanced integrations sometimes need root-level flags or complex condition
objects. The builder offers helpers for those scenarios:

```java
JsonObject conditions = new JsonObject();
conditions.addProperty("powered", true);

JsonObject json = DataDrivenHelper.connectedTexture(sprite)
        .strategy(ConnectedTextureStrategy.vertical())
        .fabricLoader()
        .configureConditions(existing -> existing.add("state", conditions))
        .property("priority", 5)
        .property("extra_loader", Identifier.of("anothermod", "ct"))
        .modifyRoot(root -> root.addProperty("debug", true))
        .build();
```

Key takeaways:

* `configureConditions` exposes the live conditions object so you can nest
  arbitrary JSON.
* `property`/`modifyRoot` let you tack on loader-specific flags without
  sacrificing readability.
* Builder validation ensures both a sprite and a strategy are provided before
  emitting JSON, preventing broken files from being generated.

With these additions, connected texture definitions live alongside the rest of
your generated assets and stay mod-agnostic with minimal ceremony.
