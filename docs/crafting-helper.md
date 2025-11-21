# Crafting Grid Helper

`CraftingGridHelper` under `org.dark.foxapi.systems.helpers.datadriven` streamlines
both the JSON you need for custom crafting grids and the Java scaffolding that
backfills new serializers.

## Shaped Recipes with Custom Dimensions

Use the shaped builder to generate JSON for any grid size. Override the
serializer type when targeting a custom crafting table and set explicit
dimensions to match its layout. When `grid` is omitted, the helper falls back
to vanilla's 3×3 expectations:

```java
JsonObject fourByFour = CraftingGridHelper.shaped(Foxapi.id("giant_pickaxe"), 1)
    .type(Identifier.of("my_mod", "forge_hammer"))
    .grid(4, 4)
    .pattern("IIII", " S S", " S S", " S S")
    .key('I', Items.IRON_BLOCK.getRegistryEntry().registryKey().getValue())
    .key('S', Items.STICK.getRegistryEntry().registryKey().getValue())
    .build();

Path recipePath = Paths.get("data/my_mod/recipes/giant_pickaxe.json");
DataDrivenHelper.write(recipePath, fourByFour);
```

Vanilla-style shaped recipes work the same way without calling `grid`, while the
`group` helper stays available for recipe book grouping.

## Shapeless Payloads

Shapeless recipes mirror the shaped API but focus on the ingredient list and the
serializer identifier:

```java
JsonObject bowlOfLight = CraftingGridHelper.shapeless(Foxapi.id("bowl_of_light"), 2)
    .type(Identifier.of("my_mod", "celestial_cauldron"))
    .ingredient(Items.BOWL.getRegistryEntry().registryKey().getValue())
    .tag(Identifier.of("c", "glowstone_dusts"))
    .ingredient(Items.AMETHYST_SHARD.getRegistryEntry().registryKey().getValue())
    .build();
```

`ingredients(Consumer<JsonArray>)` is available when you want to push multiple
entries at once from a loop—handy for counting duplicates or fanning out tag
variants:

```java
JsonObject potionBundle = CraftingGridHelper.shapeless(Foxapi.id("potion_bundle"), 3)
    .ingredients(array -> {
        JsonObject bottle = new JsonObject();
        bottle.addProperty("item", Items.GLASS_BOTTLE.getRegistryEntry().registryKey().getValue().toString());
        IntStream.range(0, 3).forEach(i -> array.add(bottle));
    })
    .ingredient(Items.STRING.getRegistryEntry().registryKey().getValue())
    .build();
```

## Serializer Skeletons

When you need a new serializer to accompany a custom grid, the helper can emit a
ready-to-edit Java skeleton:

```java
String serializer = CraftingGridHelper.serializerSkeleton(
    "com.example.mymod.crafting",
    "CelestialCauldronRecipeSerializer",
    Identifier.of("my_mod", "celestial_cauldron"),
    "CelestialCauldronRecipe"
);

Files.writeString(Paths.get("build/generated/CelestialCauldronRecipeSerializer.java"), serializer);
```

The template includes imports, a typed identifier, and method stubs for JSON and
network decoding so you can focus on parsing your larger-than-3x3 grids.

## Batch Recipes Alongside Models

`CraftingGridHelper` pairs cleanly with `DataDrivenHelper.Builder` when you want
to emit recipes, companion item models, and supporting blockstates in one
commit-friendly sweep:

```java
DataDrivenHelper.builder()
    .add(Paths.get("data/my_mod/recipes/potion_bundle.json"), potionBundle)
    .addItemModel(Paths.get("assets/my_mod/models/item/potion_bundle.json"),
        Identifier.ofVanilla("item/generated"),
        Foxapi.id("item/potion_bundle"))
    .addBlockState(Paths.get("data/my_mod/blockstates/potion_bundle.json"),
        Foxapi.id("block/potion_bundle"))
    .writeAll();
```

Bundling the writes ensures directories exist, keeps JSON pretty-printed, and
makes it harder to forget a matching model when you add a new recipe.
