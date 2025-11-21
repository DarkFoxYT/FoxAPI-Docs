# NBT Helper

`org.dark.foxapi.systems.helpers.nbt.NbtIO` bundles small, composable utilities
for reading and mutating NBT structures across items, entities and helper data.
This guide highlights the most common patterns and shows how to keep your code
focused on gameplay logic instead of boilerplate.

For persistent, server-side storage buckets (world, server, or per-player),
pair these helpers with `DataStorageHelper`/`PlayerStateHelper` covered in the
[data storage guide](./data-storage-helper.md).

## ItemStack Custom Data

Every method operating on `ItemStack` data uses Fabric's data component system.
`NbtIO.root(stack)` returns a defensive copy of the custom payload, while
`NbtIO.apply(stack, edit)` wraps the copy/edit/write dance for you.

```java
ItemStack stack = new ItemStack(Items.PAPER);
NbtIO.apply(stack, nbt -> {
    NbtIO.putInt(nbt, "spell", "level", 3);
    NbtIO.putString(nbt, "spell", "element", "arcane");
});
```

* Edits are persisted when the consumer returns.
* If the resulting compound is empty, the helper removes the custom data
  component – avoiding empty tags in NBT dumps.

Shortcuts exist for primitive reads/writes directly on the stack:

```java
int level = NbtIO.get(stack, "spell", "level", 0);
NbtIO.put(stack, "spell", "level", level + 1);
```

## Entities

Entities still expose the classic `writeNbt`/`readNbt` hooks. Use
`NbtIO.apply(entity, edit)` to capture, mutate and rehydrate an entity in one
step.

```java
NbtIO.apply(player, nbt -> {
    UUID previous = NbtIO.getUuid(nbt, "waystone", "target", null);
    if (previous == null) {
        NbtIO.putUuid(nbt, "waystone", "target", destinationId);
    }
});
```

The helper always allocates a fresh `NbtCompound`, so concurrent edits stay
isolated.

## Block Entities

Block entities restrict external write access. When you need to expose a public
API, keep the mutation inside the block entity class:

```java
public class ExampleBlockEntity extends BlockEntity {
    private int ticksUntilPulse;

    public void syncToClient(ServerWorld world) {
        markDirty();
        world.updateListeners(pos, getCachedState(), getCachedState(), Block.NOTIFY_ALL);
    }

    @Override
    protected void writeNbt(NbtCompound nbt, RegistryWrapper.WrapperLookup lookup) {
        super.writeNbt(nbt, lookup);
        NbtIO.putInt(nbt, "pulse", "delay", ticksUntilPulse);
    }

    @Override
    public void readNbt(NbtCompound nbt, RegistryWrapper.WrapperLookup lookup) {
        super.readNbt(nbt, lookup);
        ticksUntilPulse = NbtIO.getInt(nbt, "pulse", "delay", 0);
    }
}
```

Outside of the class, prefer the read-only
`NbtIO.readBlockEntity(blockEntity, lookup)` snapshot when you only need to
inspect stored data.

## Navigating Nested Paths

`getOrCreatePath` and `hasPath` accept dotted strings that walk nested
compounds. This keeps your call sites compact when working with hierarchical
structures.

```java
NbtCompound root = new NbtCompound();
NbtCompound cooldowns = NbtIO.getOrCreatePath(root, "spells.fire");
cooldowns.putInt("ticks", 40);
```

Lists follow a similar pattern. Use `getOrCreateList` when you need an `NbtList`
backed by a specific element type, or `listStrings` to materialise a list of
Java strings.

## Spatial Helpers

Several helpers serialise common Minecraft data types:

* `putBlockPos` / `getBlockPos`
* `putVec3d` / `getVec3d`
* `putUuid` / `getUuid`
* `putId` / `getId`

These methods validate the stored data type before returning values, so you can
safely supply defaults without defensive `contains` checks everywhere.

## Further Examples

See [`docs/examples/nbt.md`](examples/nbt.md) for walkthroughs that combine the
helpers above into item upgrades, entity synchronisation hooks and data-driven
cooldowns.
