# Content Creation Helper

The `ContentCreationHelper` class under
`org.dark.foxapi.systems.helpers.content` exposes a fluent API for
registering new game content. It is organised around four builders that
mirror the most common content types (items, blocks, block entities and
mobs). Each builder makes the underlying Fabric and vanilla APIs easier
to compose while still exposing hooks for advanced configuration. For a
full class-level walkthrough, see the examples in
[`docs/examples/content-registration.md`](examples/content-registration.md).

## Items

```java
ContentCreationHelper.item(id)
    .settings(settings -> settings.maxCount(1))
    .factory(CustomItem::new)
    .addToGroup(ItemGroups.INGREDIENTS)
    .addToGroupAfter(ItemGroups.COMBAT, Items.DIAMOND_SWORD)
    .onRegister(item -> LOGGER.info("Registered {}", Registries.ITEM.getId(item)))
    .register();
```

* **`settings`** – mutate the lazily created `Item.Settings` before the
  item is constructed.
* **`factory`** – replace the default factory (`Item::new`) with any
  function that accepts the configured `Item.Settings`.
* **`addToGroup` / `addToGroupBefore` / `addToGroupAfter`** – enqueue the
  item for creative tab population without manually wiring
  `ItemGroupEvents` listeners.
* **`onRegister`** – observe or cache the finished `Item` instance once
  it is present in the registry.

### Creative Tab Slotting

When you want a consistent slot order inside the creative inventory, chain the
tab helpers to position entries relative to existing items:

```java
ContentCreationHelper.item(id)
    .configureSettings(settings -> settings.maxCount(1))
    .addToGroupBefore(ItemGroups.TOOLS, Items.SHIELD)
    .addToGroupAfter(ItemGroups.COMBAT, Items.IRON_SWORD)
    .register();
```

`addToGroupBefore`/`addToGroupAfter` mirror Fabric's `ItemGroupEntries#addBefore`
and `addAfter` calls so you do not have to wire additional listeners to keep
slot positions stable across reloads. The same helpers are available on
`BlockBuilder` via `addBlockItemToGroupBefore` and `addBlockItemToGroupAfter` to
keep block items adjacent to their vanilla peers.

## Blocks & Block Items

```java
ContentCreationHelper.block(id, MyBlock::new)
    .withBlockItem(settings -> settings.maxCount(16))
    .addBlockItemToGroup(ItemGroups.BUILDING_BLOCKS)
    .onRegister(block -> BLOCKS.add(block))
    .onRegisterBlockItem(item -> ITEMS.add(item))
    .register();
```

* `withBlockItem()` creates a vanilla `BlockItem` with default settings.
* `withBlockItem(settings -> ...)` lets you configure those settings.
* `withBlockItem(block -> new CustomBlockItem(block, settings))` accepts
  a full factory override.
* `noBlockItem()` skips block item creation entirely.
* `addBlockItemToGroup`/`addBlockItemToGroupBefore`/`addBlockItemToGroupAfter`
  mirror the item helpers for creative tab placement and will
  automatically ensure a block item is registered.
* `onRegister` / `onRegisterBlockItem` provide lifecycle callbacks for
  wiring the results into data structures or logging.

## Block Entity Types

Block entities can now be associated with multiple blocks and configured
before registration:

```java
ContentCreationHelper.blockEntity(id, MyBlockEntity::new)
    .blocks(MyBlocks.PRIMARY, MyBlocks.SECONDARY)
    .configure(builder -> builder.addInstantiator(AdditionalState::new))
    .onRegister(type -> BlockEntityRendererFactories.register(type, MyRenderer::new))
    .register();
```

* `block` accepts either eager `Block` references or `Supplier<Block>`
  instances for lazily-resolved registries.
* `blocks(Block... blocks)` and `blocks(Supplier<Block>... suppliers)`
  allow registering the same block entity type for several blocks.
* `configure` exposes the backing `FabricBlockEntityTypeBuilder` so that
  custom tickers or instantiators can be added before the type is built.
* `onRegister` fires after the type is present in the registry, which is
  ideal for renderer or ticker registration.

## Mob Entity Types

`MobBuilder` wraps a `FabricEntityTypeBuilder` and layers additional
convenience:

```java
ContentCreationHelper.mob(id, FabricEntityTypeBuilder.create(SpawnGroup.CREATURE, MyMob::new))
    .dimensions(0.6F, 1.95F)
    .trackRangeBlocks(8)
    .trackedUpdateRate(2)
    .spawnableFarFromPlayer()
    .fireImmune()
    .configure(builder -> builder.spawnRestriction(SpawnRestriction.Location.ON_GROUND, Heightmap.Type.MOTION_BLOCKING_NO_LEAVES, MyMob::canSpawn))
    .onRegister(type -> SpawnRestriction.register(type, SpawnRestriction.Location.ON_GROUND, Heightmap.Type.MOTION_BLOCKING_NO_LEAVES, MyMob::canSpawn))
    .register();
```

* `spawnGroup` overrides the group used when the entity type is built.
* `dimensions` accepts either a `float` pair or a custom
  `EntityDimensions` instance.
* `trackRangeBlocks`, `trackedUpdateRate`, `spawnableFarFromPlayer`, and
  `fireImmune` delegate directly to the wrapped builder.
* `configure` exposes the underlying builder for bespoke adjustments.
* `onRegister` is useful for hooking entity type side effects such as
  renderer binding or spawn placement registration.

## Error Handling & Safety

All builders validate their required inputs and throw an informative
`IllegalStateException` if something important is missing (for example,
forgetting to attach at least one block to a block entity builder). This
keeps registration failures local and easier to diagnose.

## Suggested Usage Pattern

1. Create static holder fields for your content (for example in a
   `ModContent` class).
2. During initialisation, call the helper builder methods to construct
   and register each content type.
3. Use the `onRegister` callbacks to push references into lookup tables
   or kick off auxiliary registration such as renderers, data components
   or networking hooks.

By centralising content registration through these helpers you obtain a
consistent, testable entry point for future additions while retaining the
flexibility to drop down to Fabric or vanilla APIs whenever necessary.
