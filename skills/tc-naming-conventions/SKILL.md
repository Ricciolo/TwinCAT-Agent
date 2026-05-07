---
name: tc-naming-conventions
description: Use this skill ALWAYS before generating or modifying the names of any variables, Function Blocks (FB), Data Types (DUT), Functions, Methods, Properties, or Interfaces. It contains the mandatory Beckhoff Hungarian notation rules, prefixes (b, n, f, st, fb, etc.), and shading/name resolution rules to avoid conflicts.
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

# TwinCAT Naming Conventions (Cap. 1, 14)

Strictly follow Hungarian notation prefixes and PascalCase for identifiers to maintain code standard compliance.

## Prefixes for Object Types

| Object Type | Prefix | Example Definition | Example Instance | Call Example |
| :--- | :--- | :--- | :--- | :--- |
| **Function Block** | `FB_` | `FUNCTION_BLOCK FB_AxisControl` | `fbAxisControl` | `fbAxisControl();` |
| **Structure** | `ST_` | `TYPE ST_MachineStatus : STRUCT` | `stMachineStatus` | `stMachineStatus.nCounter := 5;` |
| **Enum** | `E_` | `TYPE E_OperationMode : (` | `eOpMode` | `eOpMode := E_STOP;` |
| **Program** | `P_` | `PROGRAM P_Main` | - | - |
| **Function** | `F_` | `FUNCTION F_Calculate : REAL` | - | - |
| **Interface** | `I_` | `INTERFACE I_Observable` | `iObservable` | - |
| **Global Variable List** | `GVL_` | `VAR_GLOBAL GVL_IO` | - | - |
| **Alias Type** | `T_` | `TYPE T_Nibble : ...` | `Nibble` | `Nibble := 1;` |

## Prefixes for Data Types

| Type | Prefix | Description | Example |
| :--- | :--- | :--- | :--- |
| **BOOL** | `b` | bit | `bStart`, `bReady`, `bSwitch` |
| **INT, UINT, WORD, SINT, USINT, DINT, UDINT, BYTE, DWORD, LWORD** | `n` or `i` | numeric / integer | `nCount`, `nIndex`, `iErrorID` |
| **REAL, LREAL** | `f` | float | `fTemperature`, `fPosition`, `fValue` |
| **STRING, WSTRING** | `s` | string | `sMessage`, `sSerialNumber`, `sName` |
| **TIME** | `t` | time | `tDelayTime`, `tDelay` |
| **DATE** | `d` | date | `dMonday` |
| **DATE_AND_TIME (DT)** | `dt` | date and time | `dtTimestamp`, `dtNewYear` |
| **TIME_OF_DAY (TOD)** | `tod` | time of day | `todAlarm` |
| **ARRAY** | `a` or `arr` | arrays | `aDataBuffer`, `arrMessages` |
| **POINTER** | `p` | pointer | `pConfig`, `pData` |
| **REFERENCE** | `ref` | reference | `refAxis` |
| **ENUM** | `e` | enum instance | `eState` |
| **Count of bytes** | `cb` | byte count | `cbLength` |
| **Count of words** | `cw` | word count | `cwRead` |

## Naming Rules

- Object name prefixes are always **UPPER CASE** with underscore `_` as separator (e.g., `FB_GetData`).
- Variable and instance prefixes are always **lower case**. The first letter of the identifier is always upper case.
- If an identifier consists of several words, the first letter of each word is upper case (**PascalCase**). No separators between words (e.g., `fbGetData`, `stBufferEntry`).
- Identifiers should only contain: `0...9`, `A...Z`, `a...z`, `_`.
- All identifiers should be specified in **English**.
- Multiple underlines in identifiers are **not allowed** (compiler error).
- **Properties** and **Methods** use plain **PascalCase** without Hungarian type prefixes (e.g., `Position`, `IsReady`, `Execute`, `Reset`).
- **Backing fields** for properties follow standard Hungarian notation. Do **NOT** use a leading underscore `_` prefix. Example: for a property `Name : STRING`, the backing field is `sName`, not `_sName`.

## Library Version Convention

Each library must contain a version function readable at runtime:

```st
FUNCTION F_GetVersionMyLib : UINT
VAR_INPUT
    nVersionElement : INT;
END_VAR

CASE nVersionElement OF
    1: (* major *) F_GetVersionMyLib := 1;
    2: (* minor *) F_GetVersionMyLib := 0;
    3: (* revision *) F_GetVersionMyLib := 0;
ELSE
    F_GetVersionMyLib := 16#FFFF;
END_CASE
```

## Shading Rules (Name Resolution — Cap. 14)

When the same identifier is used for different elements, the compiler searches in this order:

1. **Local variables:** (a) method locals → (b) FB/program/function locals + base FBs → (c) local methods of FB
2. **Global variables:** (a) project GVLs (if not `qualified_only`) → (b) library GVLs
3. **Type names:** (a) project types → (b) library types
4. **Libraries:** namespaces

### Avoiding Name Conflicts

- Follow naming conventions (prefixes for variables vs. types).
- Use `{attribute 'qualified_only'}` on GVLs and enumerations.
- Use qualified library access: `Tc2_System.ADSLOGSTR(...)`.
- Use static analysis rule **SA0027** to detect duplicate names.
- Use `THIS^.varName` to access a shadowed FB variable from a method.

## Identifier Rules

- Allowed characters: `0-9`, `A-Z`, `a-z`, `_`. No spaces, no special characters, no umlauts.
- **Identifiers are case-insensitive** for the compiler (`bMyVar` and `bmyvar` collide), but always write them in PascalCase to keep readability.
- Cannot begin with a digit.
- Multiple consecutive underscores are **not allowed** (compiler error).
- Identifiers starting with `__` (double underscore) are **reserved** for internal compiler use.
- A variable name must not equal a TwinCAT reserved keyword (see `tc-syntax-control-flow`).
- Action and step names follow the same rules as identifiers.

## Library Naming

- Beckhoff-supplied libraries start with the prefix `Tc` (e.g., `Tc2_System`, `Tc3_Module`).
- Custom libraries should use a **company namespace** prefix (e.g., `MyCorp_Motion`).
- Always declare a version function `F_GetVersion<LibName>` returning `UINT` so the runtime version is queryable.

## SUPER^ vs THIS^ — Quick Reference

| Construct | Where Allowed | Use |
| :--- | :--- | :--- |
| `THIS^` | FB body, methods, properties | Access own FB instance, disambiguate shadowed locals |
| `SUPER^` | Derived FB body, methods, properties | Call base FB body or base method |
| `SUPER^.FB_init` | **Forbidden** in explicit `FB_init` | Base init runs implicitly |
| `SUPER^.FB_reinit` | Required in derived `FB_reinit` | Must be called explicitly |

## References (Beckhoff InfoSys)

- [Programming Conventions (TC2 PLC Control)](https://infosys.beckhoff.com/content/1033/tcplccontrol/12703362827.html)
- [Identifier (TwinCAT 3)](https://infosys.beckhoff.com/content/1033/tc3_plc_intro/2529913227.html)
- [Shading Rules / Name Resolution](https://infosys.beckhoff.com/content/1033/tc3_plc_intro/11951162891.html)
