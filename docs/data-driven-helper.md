# Data-Driven Helper

`DataDrivenHelper` under `org.dark.foxapi.systems.helpers.datadriven` bundles
small builders for emitting JSON assets without hand-writing payloads or
recreating pretty-print boilerplate. It focuses on three common authoring
flows—models, blockstates, and connected textures—plus a batch builder for
writing many files at once.

## Quick Model & Blockstate Generation

Produce the vanilla JSON you need for items and blocks with one-liners:

```java
// Minimal blockstate that points every variant at a shared model
JsonObject blockState = DataDrivenHelper.createBlockState(Foxapi.id("crystal_block_model"));

// Single-layer handheld item model
JsonObject wandModel = DataDrivenHelper.createSimpleItemModel(
    Identifier.ofVanilla("item/handheld"),
    Foxapi.id("item/crystal_wand")
);
```

Pair the JSON with the built-in writer to keep indentation consistent and
folders created automatically:

```java
Path statePath = Paths.get("data/my_mod/blockstates/crystal_block.json");
Path modelPath = Paths.get("assets/my_mod/models/item/crystal_wand.json");

DataDrivenHelper.write(statePath, blockState);
DataDrivenHelper.write(modelPath, wandModel);
```

## Connected Texture Payloads

Custom connected texture loaders often need carefully structured JSON. The
helper exposes a fluent `ConnectedTextureBuilder` so you can add your sprite,
strategy, and optional custom loader identifier without manually crafting JSON
objects:

```java
JsonObject ct = DataDrivenHelper.connectedTexture()
    .sprite(Foxapi.id("block/crystal_block"))
    .strategy(ConnectedTextureStrategy.VERTICAL)
    .loader(Identifier.of("mymod", "ctm"))
    .property("render_cutout", true)
    .build();
```

## Batch Bundling with `DataDrivenHelper.Builder`

When emitting multiple assets—such as a recipe, its companion item model, and a
blockstate—you can stage them inside the builder and write them in one pass:

```java
DataDrivenHelper.builder()
    .add(Paths.get("data/my_mod/recipes/crystal_wand.json"),
        CraftingGridHelper.shapeless(Foxapi.id("crystal_wand"))
            .ingredient(Items.AMETHYST_SHARD.getRegistryEntry().registryKey().getValue())
            .ingredient(Items.STICK.getRegistryEntry().registryKey().getValue())
            .build())
    .addItemModel(Paths.get("assets/my_mod/models/item/crystal_wand.json"),
        Identifier.ofVanilla("item/generated"),
        Foxapi.id("item/crystal_wand"))
    .addBlockState(Paths.get("data/my_mod/blockstates/crystal_block.json"),
        Foxapi.id("block/crystal_block"))
    .writeAll();
```

Using the batch builder keeps related assets in sync and avoids forgetting to
create parent directories before writing files.

## Tips for Custom Pipelines

* Build complex models with `DataDrivenHelper.model(parent)` to add textures and
  arbitrary properties before calling `build()`.
* `ConnectedTextureBuilder` can be seeded with a sprite via
  `DataDrivenHelper.connectedTexture(sprite)` if you just need to toggle the
  loader or strategy in a lambda.
* The helper reuses a shared pretty-printing `Gson` instance, so outputs remain
  stable across runs—useful for comparing generated assets in version control.
