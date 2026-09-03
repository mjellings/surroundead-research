# Unreal / UE5.6 Serialization Notes

These observations come from reverse engineering SurrounDead save payloads and should not be treated as a generic Unreal Engine specification.

## Property names

🟢 **Confirmed** — names are frequently serialized in a form similar to:

```text
BaseName_<index>_<32 hex characters>
```

For semantic parsing it is often useful to derive the base name while retaining the original serialized name for diagnostics and round-trip work.

## Property lists

🟢 **Confirmed** — `None` is commonly used to terminate a property list.

## StructProperty

🟡 **Observed UE5.6 behaviour** — `StructProperty` layouts can differ from assumptions made for older saves. In examined data, the usual ArrayIndex may appear absent when the following bytes already form a plausible struct-type FString. Older saves can still contain the more familiar layout.

A parser therefore needs contextual validation rather than blindly advancing fixed offsets.

A `StructProperty` is also often associated with a one-byte `hasGuid` value followed by a 16-byte GUID region. In some parsing contexts this is most conveniently treated as a footer.

## ArrayProperty

Observed arrays contain an inner-type FString and may include optional GUID metadata followed by the element count.

Struct arrays may additionally expose a struct name and a versioned prefix/GUID region.

🟡 A versioned struct prefix has been observed in the form:

```text
int32 1
FString beginning /Script/ or /Game/
```

Exact applicability across all structures remains under investigation.

## DateTime

🟡 **Observed layout** in examined saves:

```text
version flag = 1
script path FString
5 bytes padding
int32 size = 8
int64 ticks
```

This should be validated against additional save/build samples before being treated as universal.

## Parser design note

SurrounDead's changing save representation makes tolerant parsing important. Prefer structural validation — plausible FString lengths, known property types, bounds checks and terminators — over fixed offsets wherever possible.
