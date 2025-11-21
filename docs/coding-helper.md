# Coding Helper & Onboarding Guide

This guide collects low-friction patterns and copy-pasteable scaffolds so
new modders can iterate quickly with FoxAPI. Everything here leans on the
existing helper families and keeps the amount of custom boilerplate as
small as possible.

## Quick-start Templates

### Minimal Item Registration

Use `ContentCreationHelper` to wire a custom item into the registry and a
creative tab without manually touching event listeners:

```java
public final class ModItems {
    public static final Identifier GLOVETRAP_ID = id("glovetrap");
    public static Item GLOVETRAP;

    public static void init() {
        GLOVETRAP = ContentCreationHelper.item(GLOVETRAP_ID)
            .settings(settings -> settings.maxCount(16))
            .factory(settings -> new Item(settings))
            .addToGroupAfter(ItemGroups.COMBAT, Items.IRON_SWORD)
            .register();
    }

    private static Identifier id(String path) {
        return new Identifier("example", path);
    }
}
```

This pattern keeps the fields you need for later lookups while hiding the
creative-tab wiring and registration boilerplate.

### Block + Block Entity Pair

Pair the content helper with a simple block entity class to get ticking
state and rendering hooks without duplicating registration code:

```java
public class ReactorBlock extends Block {
    public ReactorBlock(Settings settings) { super(settings); }
}

public class ReactorBlockEntity extends BlockEntity {
    public ReactorBlockEntity(BlockPos pos, BlockState state) {
        super(ModBlockEntities.REACTOR, pos, state);
    }

    public static void tick(World world, BlockPos pos, BlockState state, ReactorBlockEntity be) {
        // tick logic here
    }
}

public final class ModBlocks {
    public static final Identifier REACTOR_ID = id("reactor");
    public static Block REACTOR;
    public static BlockEntityType<ReactorBlockEntity> REACTOR_BE;

    public static void init() {
        REACTOR = ContentCreationHelper.block(REACTOR_ID, settings -> new ReactorBlock(settings.strength(4.0F)))
            .withBlockItem(item -> new BlockItem(item, new Item.Settings().maxCount(1)))
            .addBlockItemToGroup(ItemGroups.REDSTONE)
            .register();

        REACTOR_BE = ContentCreationHelper.blockEntity(REACTOR_ID, ReactorBlockEntity::new)
            .blocks(() -> REACTOR)
            .onRegister(type -> BlockEntityRendererFactories.register(type, ReactorRenderer::new))
            .configure(builder -> builder.ticker((world, pos, state, be) -> ReactorBlockEntity.tick(world, pos, state, be)))
            .register();
    }

    private static Identifier id(String path) {
        return new Identifier("example", path);
    }
}
```

The helper ensures both registrations occur with matching IDs while
exposing hooks for renderers and custom tickers.

### Data-Driven Model + Crafting Bundle

Combine `DataDrivenHelper` and `CraftingGridHelper` to emit the JSON
alongside the code change:

```java
public final class ModAssets {
    public static void generate(FabricDataOutput output, Item registeredItem) {
        DataDrivenHelper.create(output)
            .itemModel(registeredItem, model -> model.parent("item/generated").texture("layer0", "example:item/glovetrap"))
            .shapedRecipe(registeredItem, recipe -> recipe
                .patternLine("SSS")
                .patternLine("S S")
                .patternLine("SSS")
                .key('S', Items.STRING))
            .write();
    }
}
```

Calling `write()` queues both the model and recipe without hand-authoring
files in `resources/`.

## Reusable Building Blocks

* **Action gating:** pair `Cooldowns` with `TickScheduler` to debounce
  right-click abilities or timed charges without scattered counters.
* **Damage math:** run hit calculations through `DamageHelper` and
  `EnchantHelper` so knockback, crits and enchant caps stay consistent
  with vanilla expectations.
* **Status + attributes:** `EffectHelper` and `AttributeHelper` let you
  attach temporary buffs/debuffs with deterministic IDs, which makes them
  easy to remove or refresh.
* **Targeting:** `RaycastHelper` mirrors vanilla block/entity raycasts
  with fewer arguments, which keeps projectile or spell targeting code
  readable.
* **Client options:** `OptionsBindingHelper` binds `SimpleOption`
  instances to the custom `StyledCycleButton` so you can add toggles
  without duplicating GUI glue code.

## Inventory & UI Comforts

* Keep creative tabs tidy with `addToGroupBefore/After` and
  `addBlockItemToGroupBefore/After` to avoid manual `ItemGroupEvents`
  listeners.
* For custom screens, define slot indices as constants and reuse them
  when creating `Slot` instances; it keeps handler/renderer alignment
  obvious even for beginners.
* When syncing handler state, lean on `FoxapiPacket` to round-trip the
  payload locally before you add a real server listener.

## Debugging & Safety Nets

* Prefer helper lifecycle callbacks (`onRegister`, `configure`) to stash
  references or log IDs—the errors surface during init instead of mid
  gameplay.
* Use `WorldStateHelper` to persist testing toggles in the save without
  wiring codecs manually.
* Texture experiments can be quick: `TextureMixingHelper` can tint and
  blend sprites, then export the result for reuse.

## Feature Checklists

* **New item:** define a static field, register via the item builder,
  place it in a creative tab, and add a shaped recipe and generated
  model through `DataDrivenHelper`.
* **New block + BE:** register the block and block item, wire a block
  entity type via `ContentCreationHelper`, add a ticker, and generate the
  blockstate/model variants together.
* **Networked tool:** gate the action with `Cooldowns`, marshal the
  payload through `FoxapiPacket`, and use `DamageHelper` + `EffectHelper`
  for server-side effects.

## Copy-Paste Stubs

* **Blank item class**

```java
public class SimpleItem extends Item {
    public SimpleItem(Settings settings) { super(settings); }

    @Override
    public TypedActionResult<ItemStack> use(World world, PlayerEntity user, Hand hand) {
        return TypedActionResult.success(user.getStackInHand(hand));
    }
}
```

* **Blank block entity**

```java
public class SimpleBlockEntity extends BlockEntity {
    public SimpleBlockEntity(BlockPos pos, BlockState state) {
        super(ModBlockEntities.SIMPLE, pos, state);
    }

    public static void tick(World world, BlockPos pos, BlockState state, SimpleBlockEntity be) {
        // tick logic here
    }
}
```

Drop these stubs into your project and focus on gameplay logic instead of
wiring.
