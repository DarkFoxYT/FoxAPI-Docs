# Content Registration Examples

These snippets expand on the builder reference in
[`docs/content-creation-helper.md`](../content-creation-helper.md) by showing how
full classes can look when centralising registration logic.

## Item & Block Registration Hub

```java
public final class ExampleContent {
    public static final Identifier ARCANE_INGOT_ID = Foxapi.id("arcane_ingot");
    public static final Identifier RUNE_BLOCK_ID = Foxapi.id("rune_block");

    public static Item ARCANE_INGOT;
    public static Block RUNE_BLOCK;
    public static BlockEntityType<RuneBlockEntity> RUNE_BLOCK_ENTITY;

    private ExampleContent() {}

    public static void bootstrap() {
        ContentCreationHelper.item(ARCANE_INGOT_ID)
            .settings(settings -> settings.maxCount(16).fireproof())
            .factory(settings -> ARCANE_INGOT = new Item(settings))
            .addToGroupAfter(ItemGroups.INGREDIENTS, Items.BLAZE_ROD)
            .onRegister(item -> LOGGER.info("Registered {}", Registries.ITEM.getId(item)))
            .register();

        ContentCreationHelper.block(RUNE_BLOCK_ID, RuneBlock::new)
            .withBlockItem(settings -> settings.maxCount(1))
            .addBlockItemToGroup(ItemGroups.FUNCTIONAL)
            .onRegister(block -> RUNE_BLOCK = block)
            .onRegisterBlockItem(item -> LOGGER.info("Rune block item ready"))
            .register();
    }
}
```

## Block Entity & Renderer Wiring

```java
public final class ExampleBlockEntities {
    public static BlockEntityType<RuneBlockEntity> RUNE_BLOCK_ENTITY;

    private ExampleBlockEntities() {}

    public static void bootstrap() {
        ContentCreationHelper.blockEntity(Foxapi.id("rune_block_entity"), RuneBlockEntity::new)
            .blocks(() -> ExampleContent.RUNE_BLOCK)
            .configure(builder -> builder.addInstantiator(state -> new RuneBlockEntity()))
            .onRegister(type -> {
                RUNE_BLOCK_ENTITY = type;
                BlockEntityRendererFactories.register(type, RuneBlockEntityRenderer::new);
            })
            .register();
    }
}
```

## Mob Registration with Spawn Rules

```java
public final class ExampleMobs {
    public static EntityType<RuneGolemEntity> RUNE_GOLEM;

    private ExampleMobs() {}

    public static void bootstrap() {
        ContentCreationHelper.mob(
            Foxapi.id("rune_golem"),
            FabricEntityTypeBuilder.create(SpawnGroup.MONSTER, RuneGolemEntity::new)
        )
        .spawnGroup(SpawnGroup.MONSTER)
        .dimensions(1.4F, 2.7F)
        .trackRangeBlocks(16)
        .trackedUpdateRate(3)
        .fireImmune()
        .configure(builder -> builder.defaultAttributes(RuneGolemEntity::createAttributes))
        .onRegister(type -> {
            RUNE_GOLEM = type;
            SpawnRestriction.register(type, SpawnRestriction.Location.ON_GROUND,
                Heightmap.Type.MOTION_BLOCKING_NO_LEAVES, RuneGolemEntity::canSpawn);
        })
        .register();
    }
}
```

The builders keep the registration flow consistent across content types: collect
static holders, call the helper during your mod initialiser and wire secondary
systems (renderers, attributes, loot tables) inside the provided callbacks.
