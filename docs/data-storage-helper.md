# Data Storage Helper

Use these helpers when you need to persist server-side data without sprinkling
boilerplate across blocks, items, or mixins. They wrap Minecraft's
`PersistentState` system, letting you store arbitrary NBT in world, server, or
player scopes with a tiny API.

## World and Server Storage

`WorldStateHelper` already ships with FoxAPI. `DataStorageHelper` exposes
friendly entry points that return a handle for the current world or the
server-wide overworld bucket.

```java
// World-scoped toggle (separate per dimension)
WorldStateHelper state = DataStorageHelper.world(serverWorld, "dark:moon_gates");
boolean open = state.getBoolean("open", false);
state.setBoolean("open", !open);

// Server-scoped counter shared across all worlds
WorldStateHelper global = DataStorageHelper.server(server, "dark:rituals");
global.edit(nbt -> {
    int completions = NbtIO.getInt(nbt, "counts", "finished", 0);
    NbtIO.putInt(nbt, "counts", "finished", completions + 1);
});
```

Notes:

* The helper marks the state dirty whenever you mutate it, so data flushes with
  the vanilla save lifecycle.
* The overworld is used for server-wide data to avoid a custom save folder.

## Player Storage

`PlayerStateHelper` persists a map of UUID → `NbtCompound`, ideal for tracking
abilities, quest progress, cooldowns, or tutorial flags.

```java
// Increment a per-player counter
PlayerStateHelper claims = DataStorageHelper.players(server, "dark:claims");
claims.compute(player.getUuid(), nbt -> {
    int total = nbt.getInt("count") + 1;
    nbt.putInt("count", total);
    return total;
});

// Gate a feature behind a one-time unlock
if (!claims.getBoolean(player.getUuid(), "unlocked", false)) {
    claims.setBoolean(player.getUuid(), "unlocked", true);
    // perform unlock logic
}

// Delete records when a player leaves a faction
claims.remove(player);
```

Helpers mirror the world API: `put`, `compute`, and `updateIf` all mark the
state dirty for you.

## Combining With NbtIO

Pair the storage helpers with `NbtIO` to avoid hand-rolled path traversal,
type checks, or empty-compound cleanup.

```java
// Store a nested object with defensive defaults
DataStorageHelper.withPlayer(server, "dark:waystones", player, nbt -> {
    NbtIO.putUuid(nbt, "waystone", "target", targetId);
    NbtIO.putInt(nbt, "waystone", "charges", 3);
});

// Read with defaults and skip guard clauses
NbtCompound snapshot = DataStorageHelper.players(server, "dark:waystones").view(player);
UUID target = NbtIO.getUuid(snapshot, "waystone", "target", null);
int charges = NbtIO.getInt(snapshot, "waystone", "charges", 0);
```

## Server-Side Best Practices

* Keep mutations on the server thread; the helpers assume single-threaded
  access like vanilla `PersistentState`.
* Use namespaced keys (e.g., `modid:feature`) to avoid collisions with other
  mods' saves.
* Sync only the minimal subset of data to clients. Prefer packets that ship
  the needed slice rather than mirroring the full storage blob.
* When exposing API points to other mods, return defensive copies via `view`
  so callers cannot mutate without going through your guard rails.
