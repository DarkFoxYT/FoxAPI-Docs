# NBT Examples – Practical Patterns

The snippets below demonstrate how to lean on `NbtIO` for common gameplay
scenarios.

## Upgrading an Item In-World

```java
public class UpgradeScrollItem extends Item {
    public UpgradeScrollItem(Settings settings) {
        super(settings);
    }

    @Override
    public TypedActionResult<ItemStack> use(World world, PlayerEntity user, Hand hand) {
        ItemStack stack = user.getStackInHand(hand);
        NbtIO.apply(stack, nbt -> {
            int level = NbtIO.getInt(nbt, "upgrade", "level", 0);
            NbtIO.putInt(nbt, "upgrade", "level", Math.min(level + 1, 5));
            NbtIO.putBoolean(nbt, "upgrade", "glowing", true);
        });
        return TypedActionResult.success(stack, world.isClient());
    }
}
```

Highlights:

* `apply` removes the component when it becomes empty – there is no extra code
  to clean up unused tags.
* Primitive helpers keep the editing logic tight and readable.

## Persisting Entity Waypoints

```java
public class WaystoneComponent implements Component {
    private final Entity entity;
    private BlockPos target;

    public WaystoneComponent(Entity entity) {
        this.entity = entity;
    }

    public void setTarget(BlockPos pos) {
        this.target = pos;
        NbtIO.apply(entity, nbt -> NbtIO.putBlockPos(nbt, "waystone", "target", pos));
    }

    public Optional<BlockPos> getTarget() {
        NbtCompound nbt = NbtIO.root(entity);
        return Optional.ofNullable(NbtIO.getBlockPos(nbt, "waystone", "target", null));
    }
}
```

Highlights:

* Entity edits go through `apply` to guarantee `readNbt` is invoked after
  mutation.
* Reads can use the lightweight `root` snapshot when you just need to inspect
  state.

## Block Entity Sync Payload

```java
public class RitualTableBlockEntity extends BlockEntity implements TickedBlockEntity {
    private final List<UUID> participants = new ArrayList<>();

    public void addParticipant(UUID player) {
        participants.add(player);
        markDirty();
    }

    @Override
    protected void writeNbt(NbtCompound nbt, RegistryWrapper.WrapperLookup lookup) {
        super.writeNbt(nbt, lookup);
        NbtList list = NbtIO.getOrCreateList(nbt, "ritual", "participants", NbtElement.INT_ARRAY_TYPE);
        list.clear();
        for (UUID id : participants) {
            list.add(NbtIntArray.of(NbtHelper.fromUuid(id)));
        }
    }

    @Override
    public void readNbt(NbtCompound nbt, RegistryWrapper.WrapperLookup lookup) {
        super.readNbt(nbt, lookup);
        participants.clear();
        NbtList list = NbtIO.getOrCreatePath(nbt, "ritual").getList("participants", NbtElement.INT_ARRAY_TYPE);
        for (int i = 0; i < list.size(); i++) {
            participants.add(NbtHelper.toUuid((NbtIntArray) list.get(i)));
        }
    }
}
```

Highlights:

* `getOrCreateList` ensures the list exists before you write to it.
* Pairing `writeNbt`/`readNbt` keeps serialisation symmetrical.

## Debugging Tip

When a structure does not behave as expected, dump it to the log:

```java
LOGGER.info("Item NBT: {}", NbtIO.root(stack));
```

The helper always returns defensive copies, so logging has no side effects on
your gameplay state.
