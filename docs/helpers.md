# Helper Catalog

FoxAPI bundles a grab bag of small helpers that smooth over common modding chores. This page summarises each helper family and links to the implementation so you can quickly jump into the right utility.

## General Utilities

| Helper | Summary |
| --- | --- |
| [Coding helper onboarding guide](./coding-helper.md) | Starter templates, feature checklists, and copy-pasteable stubs for new modders. |

## Gameplay & Timing

| Helper | Summary |
| --- | --- |
| [`AttributeHelper`](../src/main/java/org/dark/foxapi/systems/helpers/AttributeHelper.java) | Fluent builder for adding/removing attribute modifiers with deterministic IDs, optional hooks, and timed removal via `TickScheduler`. |
| [`Cooldowns`](../src/main/java/org/dark/foxapi/systems/helpers/Cooldowns.java) | Friendly wrapper around `ItemCooldownManager` with timeline planning and ready/trigger helpers. |
| [`DamageHelper`](../src/main/java/org/dark/foxapi/systems/helpers/DamageHelper.java) | Shortcuts for applying common damage sources or building a custom `DamagePlan`. |
| [`EffectHelper`](../src/main/java/org/dark/foxapi/systems/helpers/EffectHelper.java) | Tiny helpers for applying status effects with consistent durations and amplifier handling. |
| [`EnchantHelper`](../src/main/java/org/dark/foxapi/systems/helpers/EnchantHelper.java) | Convenience methods for testing enchant compatibility, level caps, and calculating bonuses. |
| [`RaycastHelper`](../src/main/java/org/dark/foxapi/systems/helpers/RaycastHelper.java) | Replicates vanilla block/entity raycasts with a fluent query API to keep hit detection code tidy. |
| [`TextureMixingHelper`](../src/main/java/org/dark/foxapi/systems/helpers/TextureMixingHelper.java) | Client-side sprite compositor that blends texture layers, tints them, and optionally uploads/exports the result. |
| [`TickScheduler`](../src/main/java/org/dark/foxapi/systems/helpers/TickScheduler.java) | Minimal delayed-task scheduler for server/world ticks with cancellation and repeat support. |
| [`OptionsBindingHelper`](../src/main/java/org/dark/foxapi/systems/helpers/options/OptionsBindingHelper.java) | Binds `SimpleOption` values to the custom `StyledCycleButton`, including fluent sizing/positioning. |

## NBT Helpers

| Helper | Summary |
| --- | --- |
| [`NbtIO`](../src/main/java/org/dark/foxapi/systems/helpers/nbt/NbtIO.java) | Streaming read/write utilities for NBT blobs, including safe compression and type checks. See [NBT helper notes](./nbt-helper.md) for examples. |
| [`WorldStateHelper`](../src/main/java/org/dark/foxapi/systems/helpers/nbt/WorldStateHelper.java) | Persists and loads arbitrary state into world properties without hand-rolling codecs. |
| [`PlayerStateHelper`](../src/main/java/org/dark/foxapi/systems/helpers/nbt/PlayerStateHelper.java) | Persistent state table keyed by player UUID for server-side flags, counters, and other NBT blobs. |
| [`DataStorageHelper`](../src/main/java/org/dark/foxapi/systems/helpers/nbt/DataStorageHelper.java) | Facade that exposes world/server storage and player tables in one place. Covered in the [data storage guide](./data-storage-helper.md). |

## Networking

| Helper | Summary |
| --- | --- |
| [`FoxapiPacket`](../src/main/java/org/dark/foxapi/systems/helpers/network/FoxapiPacket.java) & [`FoxapiPacketHandler`](../src/main/java/org/dark/foxapi/systems/helpers/network/FoxapiPacketHandler.java) | In-memory packet sandbox that mirrors Fabric's networking APIs for fast iteration. Walkthroughs live in the [networking helper guide](./networking-helper.md). |

## Content & Data-Driven Helpers

| Helper | Summary |
| --- | --- |
| [`ContentCreationHelper`](../src/main/java/org/dark/foxapi/systems/helpers/content/ContentCreationHelper.java) | Fluent registration utilities for blocks, items, creative-tab slotting, and related assets. See the [content creation guide](./content-creation-helper.md). |
| [`DataDrivenHelper`](../src/main/java/org/dark/foxapi/systems/helpers/datadriven/DataDrivenHelper.java) | Builders for emitting blockstates, models, connected textures, and other JSON assets. Covered in the [data-driven helper walkthrough](./data-driven-helper.md). |
| [`CraftingGridHelper`](../src/main/java/org/dark/foxapi/systems/helpers/datadriven/CraftingGridHelper.java) | Utilities for authoring custom-shaped crafting grids and serialising them to JSON. Covered in the [crafting helper guide](./crafting-helper.md). |
| [`ConnectedTextureStrategy`](../src/main/java/org/dark/foxapi/systems/helpers/datadriven/ConnectedTextureStrategy.java) | Strategy interface for generating connected texture variants from data-driven definitions. |

## Library Module System

| Helper | Summary |
| --- | --- |
| [`LibraryHelper`](../src/main/java/org/dark/foxapi/systems/helpers/library/LibraryHelper.java) | Entry point for registering library modules and orchestrating their lifecycle. |
| [`SharedServiceRegistry`](../src/main/java/org/dark/foxapi/systems/helpers/library/SharedServiceRegistry.java) | Lightweight service locator used by modules to share singletons without global state. |
| [`LibraryModule`](../src/main/java/org/dark/foxapi/systems/helpers/library/LibraryModule.java) & [`LibraryModuleContext`](../src/main/java/org/dark/foxapi/systems/helpers/library/LibraryModuleContext.java) | Module contract and context passed to implementors during registration and runtime. |

## Entity & AI

| Helper | Summary |
| --- | --- |
| [`EntityAIHelper`](../src/main/java/org/dark/foxapi/systems/helpers/entity/EntityAIHelper.java) | Helpers for attaching and pruning goal selectors on living entities without duplicating boilerplate. |
| [`AttributeHelper`](../src/main/java/org/dark/foxapi/systems/helpers/AttributeHelper.java) | See General Utilities above; often paired with AI adjustments for temporary buffs or debuffs. |

## Multiblock Framework

| Helper | Summary |
| --- | --- |
| [`MBPattern`](../src/main/java/org/dark/foxapi/systems/helpers/multiblock/MBPattern.java) | Describes a multiblock layout using block predicates and anchor metadata. |
| [`MBMatcher`](../src/main/java/org/dark/foxapi/systems/helpers/multiblock/MBMatcher.java) & [`MBMatch`](../src/main/java/org/dark/foxapi/systems/helpers/multiblock/MBMatch.java) | Pattern matching utilities that locate valid structures and expose matched block positions. |
| [`MBRegistry`](../src/main/java/org/dark/foxapi/systems/helpers/multiblock/MBRegistry.java) | Registry for multiblock patterns and controller types. |
| [`MBInstance`](../src/main/java/org/dark/foxapi/systems/helpers/multiblock/MBInstance.java) | Runtime handle for an assembled multiblock, including orientation and bounding boxes. |
| [`MBOrientation`](../src/main/java/org/dark/foxapi/systems/helpers/multiblock/MBOrientation.java) | Utility for rotating multiblock layouts to match player placement. |
| [`MBBlockPredicate`](../src/main/java/org/dark/foxapi/systems/helpers/multiblock/MBBlockPredicate.java) | Predicate builder for matching blocks within a pattern. |
| [`ControllerBlock`](../src/main/java/org/dark/foxapi/systems/helpers/multiblock/ControllerBlock.java) & [`ControllerBlockEntity`](../src/main/java/org/dark/foxapi/systems/helpers/multiblock/ControllerBlockEntity.java) | Base controller block and block entity that coordinate structure activation. |
| [`AccurateCollisionHelper`](../src/main/java/org/dark/foxapi/systems/helpers/multiblock/AccurateCollisionHelper.java) | Generates voxel shapes for multiblock components to keep collision boxes aligned with the pattern. |
| [`FoxapiMultiblockInit`](../src/main/java/org/dark/foxapi/systems/helpers/multiblock/FoxapiMultiblockInit.java) | Bootstrap entry point for registering the multiblock framework pieces. |

## Other Systems

| Helper | Summary |
| --- | --- |
| [`Spell`/`Veil`/`Bookshelf` integration helpers](../src/main/java/org/dark/foxapi/systems/integration) | Bridges for third-party mods (Lodestone, Veil, Bookshelf, Cardinal Components) used by the editor suite. |
| [`ContentCreationHelper` examples](./examples/content-registration.md) | Copy-pasteable snippets showing helper usage in context. |

