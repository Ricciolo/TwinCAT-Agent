---
name: tc-datatypes-memory
description: Use this skill when the user asks about data types, arrays, structures (STRUCT), enumerations (ENUM), pointers (POINTER), references (REFERENCE), or dynamic memory allocation (NEW/DELETE). It also contains critical rules for memory alignment (pack_mode) to minimize padding bytes.
disallowedTools:
  - edit
  - write
  - replace_string_in_file
  - multi_replace_string_in_file
  - create_file
  - grep_search
  - file_search
  - list_dir
---

# TwinCAT Data Types & Memory (Cap. 2, 13)

## Standard Data Types

| Category | Type | Size | Range / Notes |
| :--- | :--- | :--- | :--- |
| **Boolean** | `BOOL` | 8 bit | `TRUE` (1), `FALSE` (0). |
| **Bit String** | `BYTE` | 8 bit | 0 to 255 |
| | `WORD` | 16 bit | 0 to 65535 |
| | `DWORD` | 32 bit | 0 to 4294967295 |
| | `LWORD` | 64 bit | 0 to 2^64-1 |
| **Signed Integer** | `SINT` | 8 bit | -128 to 127 |
| | `INT` | 16 bit | -32768 to 32767 |
| | `DINT` | 32 bit | -2147483648 to 2147483647 |
| | `LINT` | 64 bit | 64-bit signed integer |
| **Unsigned Integer** | `USINT` | 8 bit | 0 to 255 |
| | `UINT` | 16 bit | 0 to 65535 |
| | `UDINT` | 32 bit | 0 to 4294967295 |
| | `ULINT` | 64 bit | 64-bit unsigned integer |
| **Floating Point** | `REAL` | 32 bit | IEEE 754 single precision |
| | `LREAL` | 64 bit | IEEE 754 double precision |
| **String** | `STRING(n)` | n+1 bytes | Default 80 chars, max 255. |
| | `WSTRING` | 2*(n+1) bytes | Unicode string. |
| **Time** | `TIME` | 32 bit | `T#0ms` to `T#49d17h2m47s295ms` |
| | `LTIME` | 64 bit | High-resolution time. |
| **Date** | `DATE` | 32 bit | `D#1970-01-01` format. |
| | `TIME_OF_DAY` (TOD) | 32 bit | `TOD#00:00:00` format. |
| | `DATE_AND_TIME` (DT) | 32 bit | `DT#1970-01-01-00:00:00` format. |

## User-Defined Data Types

| Type | Syntax | Notes |
| :--- | :--- | :--- |
| **ARRAY** | `ARRAY [lo..hi] OF <type>` | 1D, 2D, 3D supported. Max 9 dimensions via nesting. |
| **STRUCT** | `TYPE ST_Name : STRUCT ... END_STRUCT END_TYPE` | **8-byte alignment** (TwinCAT 3). Order by descending size to minimize padding. Unpacked structs inside packed structs are **not allowed** (error 3850). |
| **ENUM** | `TYPE E_Name : ( val1, val2, ... );` | Always use `{attribute 'strict'}` + `{attribute 'qualified_only'}`. Access: `E_Name.eValue`. |
| **ALIAS** | `TYPE T_Name : <base_type>; END_TYPE` | Supports default init and subrange combination. |
| **SUBRANGE** | `TYPE T_Name : INT (lo..hi); END_TYPE` | Only on integer types. |
| **POINTER** | `POINTER TO <type>` | Check `<> 0` before dereferencing with `^`. Use `ADR()` to get address. |
| **REFERENCE** | `REFERENCE TO <type>` | Assign with `REF=`. Check with `__ISVALIDREF()`. |
| **INTERFACE** | `I_Name` | Check `<> 0` before use. Convert with `__QUERYINTERFACE()`. |

## Type Literals / Constants

```st
nHex    := 16#FF;         (* Hexadecimal *)
nOct    := 8#77;          (* Octal *)
nBin    := 2#1010_1010;   (* Binary, underscores for readability *)
nVal    := SINT#127;      (* Explicit type prefix *)
fVal    := REAL#3.14;
tDelay  := T#100ms;
tLong   := T#2h30m;
dToday  := D#2026-02-20;
todNow  := TOD#18:30:00;
dtStamp := DT#2026-02-20-18:00:00;
sText   := 'Hello World';
sEscape := '$N';          (* Newline escape *)
```

## Reference Assignment Operators

| Operator | Effect |
| :--- | :--- |
| `refA REF= value` | `refA` now points to `value` (address assignment) |
| `refA := value` | Writes `value` to location referenced by `refA` (value copy) |

```st
refInt REF= nMyInt;
refInt := 42;
bValid := __ISVALIDREF(refInt);

ipSample := fbSample;
IF ipSample <> 0 THEN
    nResult := ipSample.Add(3, 6);
END_IF;
bOk := __QUERYINTERFACE(iBase, iDerived);
```

## Enumeration with Strict & Qualified Access

```st
{attribute 'qualified_only'}
{attribute 'strict'}
TYPE E_Color :
(
    eRed,
    eYellow,
    eGreen := 10,
    eBlue
) UINT := E_Color.eRed;
END_TYPE

eMyColor := E_Color.eBlue;   (* Always qualified *)
```

## Structure Alignment (Cap. 13)

TwinCAT 3 uses **8-byte alignment** (vs. 1-byte in TC2 x86).

- Variable aligned to multiple of `MIN(data_type_size, pack_mode)`.
- Structure total size is multiple of `MIN(largest_member_size, pack_mode)`.
- **Implicit padding bytes** are inserted (NOT initialized — relevant for `MEMCMP`).
- Each FB implicitly contains a pointer to virtual method table (8 bytes on 64-bit).

```st
TYPE ST_Optimal :      (* 16 bytes total — minimal padding *)
STRUCT
    fValue : LREAL;    (* 8 bytes at offset 0 *)
    nCount : DINT;     (* 4 bytes at offset 8 *)
    bFlag  : BOOL;     (* 1 byte at offset 12 + 3 padding *)
END_STRUCT
END_TYPE

TYPE ST_Wasteful :     (* 24 bytes total — bad ordering *)
STRUCT
    bFlag  : BOOL;     (* 1 byte + 7 padding *)
    fValue : LREAL;    (* 8 bytes *)
    nCount : DINT;     (* 4 bytes + 4 padding *)
END_STRUCT
END_TYPE
(* Tip: order members by descending size to minimize padding *)
```

### Controlling Alignment with pack_mode

```st
{attribute 'pack_mode' := '1'}   (* 1-byte — no padding *)
TYPE ST_Packed :
STRUCT
    bFlag  : BOOL;
    nValue : DINT;     (* immediately follows bFlag, no padding *)
END_STRUCT
END_TYPE
```

| pack_mode value | Alignment |
| :--- | :--- |
| `'0'` | 8-byte (default TwinCAT 3) |
| `'1'` | 1-byte (no padding) |
| `'2'` | 2-byte |
| `'4'` | 4-byte |

## Dynamic Memory — __NEW / __DELETE

FBs/DUTs require `{attribute 'enable_dynamic_creation'}`.

```st
VAR
    pFB : POINTER TO FB_Dynamic;
END_VAR

pFB := __NEW(FB_Dynamic);
IF pFB <> 0 THEN
    nResult := pFB^.DoWork(nParam := 42);
END_IF;
__DELETE(pFB);   (* releases memory, sets pFB to 0, calls FB_exit *)
```

- `__NEW` allocates from router memory pool. **Must** pair with `__DELETE` to avoid leaks.
- `__DELETE` releases memory and sets pointer to 0.
- For inherited FBs, use derived type pointer to ensure correct `FB_exit` call.

## Exception Handling

```st
VAR
    exc : __SYSTEM.ExceptionCode;
END_VAR
__TRY
    nResult := nA / nB;
__CATCH(exc)
    IF exc = __SYSTEM.ExceptionCode.RTSEXCPT_DIVIDEBYZERO THEN
        bError := TRUE;
    END_IF;
__ENDTRY
```

> Warning: `F_RaiseException()` outside a `__TRY` block stops the controller.

## UNION Type

```st
TYPE U_Convert :
UNION
    nValue : DWORD;
    aBytes : ARRAY [0..3] OF BYTE;
END_UNION
END_TYPE
```

- All members share the **same memory location** (size = largest member).
- Useful for type-punning (e.g., reading individual bytes of a `DWORD`).
- Combine with `{attribute 'pack_mode' := '1'}` to remove padding when overlaying I/O data.

## Platform-Independent Types

| Type | Description |
| :--- | :--- |
| `PVOID` | Untyped pointer; size matches platform pointer width (4 bytes on 32-bit, 8 bytes on 64-bit). Recommended return type of `ADR()`. |
| `XINT` / `UXINT` | Platform-native signed/unsigned integer (32-bit on x86, 64-bit on x64). |
| `XWORD` | Platform-native bit-string. |
| `BIT` | Single bit (only valid as struct member, packed into bytes). Cannot use `ADR` — use `BITADR`. |

Use these to write code that compiles unchanged on both 32-bit and 64-bit TwinCAT runtimes.

## Variable-Length Arrays (VAR_IN_OUT only)

```st
FUNCTION F_Sum : DINT
VAR_IN_OUT
    aData : ARRAY [*] OF DINT;   (* Bounds resolved at call site *)
END_VAR
VAR
    i, nLow, nHigh : DINT;
END_VAR
nLow  := LOWER_BOUND(aData, 1);
nHigh := UPPER_BOUND(aData, 1);
FOR i := nLow TO nHigh DO
    F_Sum := F_Sum + aData[i];
END_FOR;
```

## References (Beckhoff InfoSys)

- [Data Types](https://infosys.beckhoff.com/content/1033/tc3_plc_intro/2529388939.html)
- [Alignment](https://infosys.beckhoff.com/content/1033/tc3_plc_intro/3539428491.html)
- [Global Data Types](https://infosys.beckhoff.com/content/1033/tc3_plc_intro/10394151819.html)
