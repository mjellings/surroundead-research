# `PlayerInfo.sav`

`PlayerInfo.sav` is the main current research target for inventory, item instances and container relationships.

## Item records

🟢 **Confirmed** — a useful discovery strategy is to scan for struct names matching the general pattern:

```text
ItemInfo_<index>_<32 hex characters>
```

The inner data resembles `RepItemInfo` / `ItemInfo` structures.

### Observed item fields

| Field | Notes |
| --- | --- |
| `ItemID` | Asset/path identifying the item type |
| `Count` | Stack/count value |
| `Weight` | Item weight |
| `Price` | Item price/value |
| `Durability` | `Vector2D`; observed as current/max-like pair |
| `ItemVec` | `Vector2D`; exact semantics unresolved |
| `Stats` | Array of `S_ItemStat`-like structures |
| `CustomDataKey` | Present; semantics not fully mapped |
| `CustomDataValue` | Present; semantics not fully mapped |
| `Pending` | Present; semantics unresolved |
| `UniqueID` | GUID |
| `UniqueServerID` | GUID; strong candidate for item-instance identity |
| `ReplicationUID` | GUID |

### Placement / container fields

Observed fields include:

```text
ContainerMotherID
InContainerUID
SlotIndex
InContainerIndex
Rotated
PickupRef
IsContainer
SubContainers
```

`SubContainers` contains `S_SubContainerInfo`-like records with fields including `MotherUID` / `ReplicationUID` and `Index`.

## Item-owned container blocks

🟡 **Observed** — container-capable items can contain extended regions such as:

- `ContainerItems` / `ContainerPickupsInfo`, referring to contained item instances via `UniqueServerID` and sometimes `PickupRef`.
- `ContainerSlots` / `S_ContainerSlots`, with fields including `Index`, `HostingUID` / `ReplicationUID` / `InContainerUID`, `Column`, `Row` and `IsEquipTo`.

The exact relationship model is still being refined.

## `MainJigContainers`

🟡 **Current best model** — `MainJigContainers` is an `ArrayProperty` involving `S_ReplicatedContainerInfo`, but its declared count should **not** automatically be interpreted as that many independent top-level serialized structs.

In an examined 0.8 save, the data appeared as one very large top-level `S_ReplicatedContainerInfo` region containing repeated inner container sections. The declared array count closely matched the number of those repeated inner sections.

Observed container fields include:

```text
UniqueServerID / ReplicationUID
ContainerIndex
Columns / Rows
ContainerDimension
IsContainer
ContainerSlots
ContainerItems / ContainerPickupsInfo
```

Working graph:

```text
MainJigContainers
  └─ S_ReplicatedContainerInfo
       ├─ ContainerSlots[]
       └─ ContainerItems / ContainerPickupsInfo[]
            └─ references item UniqueServerID

ItemInfo_*
  ├─ identity
  ├─ ContainerMotherID / InContainerUID
  ├─ SlotIndex / InContainerIndex
  └─ optional SubContainers / owned container data
```

Confidence varies by edge. Do not assume every GUID field is populated: some examined saves contain zero GUIDs in fields that appear structurally important.

## Known related structure/type names

`RepItemInfo`, `ItemInfo`, `S_ReplicatedContainerInfo`, `S_ContainerSlots`, `ContainerPickupsInfo`, `S_ContainerPickupsInfo`, `S_SubContainerInfo`, `S_ItemStat`, `Guid`, `Vector2D`.

## Open questions

🔵 Equipment-slot mapping and `IsEquipTo` semantics, weapon attachment relationships, `ItemVec`, `CustomData*`, `Pending`, complete `S_ItemStat` mapping, and container-binding behaviour when candidate GUID fields are zero remain active research areas.
