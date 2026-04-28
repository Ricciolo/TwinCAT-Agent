---
name: tc-syntax-control-flow
description: Use this skill to correctly format ST (Structured Text) logic, operators, and control flow statements. It includes precedence rules, IF/CASE/FOR/WHILE loops syntax, mathematical/bitwise operator rules (MOD, SHL, ADR, BITADR), and the list of reserved IEC keywords.
---

# TwinCAT Syntax & Control Flow (Cap. 3, 4, 15)

## Operator Precedence (strongest to weakest)

| Precedence | Operation | Symbol |
| :--- | :--- | :--- |
| 1 | Parentheses | `(expression)` |
| 2 | Function call | `FunctionName(params)` |
| 3 | Exponentiation | `EXPT` |
| 4 | Negate / Complement | `-`, `NOT` |
| 5 | Multiply, Divide, Modulo | `*`, `/`, `MOD` |
| 6 | Add, Subtract | `+`, `-` |
| 7 | Comparison | `<`, `>`, `<=`, `>=` |
| 8 | Equality | `=`, `<>` |
| 9 | Boolean AND | `AND` |
| 10 | Boolean XOR | `XOR` |
| 11 | Boolean OR | `OR` |

Equal-precedence operators are processed **left to right**.

## All Operators

| Operation | Operators | Example |
| :--- | :--- | :--- |
| **Assignment** | `:=` | `bVar := TRUE;` |
| **Arithmetic** | `+`, `-`, `*`, `/`, `MOD`, `EXPT` | `nVal := (nA * nB) MOD 10;` |
| **Bitwise** | `AND`, `OR`, `XOR`, `NOT` | `nFlags := nInput AND 16#FF;` |
| **Bit Shift** | `SHL`, `SHR`, `ROL`, `ROR` | `nResult := SHL(nVal, 2);` |
| **Comparison** | `=`, `<>`, `>`, `<`, `>=`, `<=` | `IF nCnt >= 10 THEN ...` |
| **Selection** | `SEL`, `MAX`, `MIN`, `LIMIT`, `MUX` | `nVal := SEL(bSwitch, 0, 100);` |
| **Address** | `ADR`, `SIZEOF`, `XSIZEOF`, `BITADR` | `pData := ADR(stConfig);` |
| **Dynamic Memory** | `__NEW`, `__DELETE` | `pFB := __NEW(FB_Dyn); __DELETE(pFB);` |
| **Type Query** | `__QUERYINTERFACE`, `__QUERYPOINTER` | Runtime interface/pointer conversion. Returns `BOOL`. |
| **Validation** | `__ISVALIDREF` | `bOk := __ISVALIDREF(refVar);` |
| **Introspection** | `__VARINFO`, `__POUNAME`, `__POSITION` | Runtime variable info, POU name, source position. |

### Operator Notes

- `MOD` is **not defined for `REAL`** types (error 4017).
- `ADR` is **not allowed on bits** — use `BITADR` instead (error 4031).
- `ADR` must not be applied to expressions or constants (error 4030).
- `XSIZEOF` returns `ULINT` on 64-bit, `UDINT` on 32-bit; use `__UXINT` for portability.
- `__NEW` allocates from router memory pool. FBs/DUTs require `{attribute 'enable_dynamic_creation'}`.

## Control Flow

### IF...THEN...ELSIF...ELSE...END_IF

```st
IF fTemperature < 0.0 THEN
    eState := E_State.FROZEN;
ELSIF fTemperature = 0.0 THEN
    eState := E_State.FREEZING;
ELSIF fTemperature > 100.0 THEN
    eState := E_State.BOILING;
ELSE
    eState := E_State.NORMAL;
END_IF;
```
Condition must be `BOOL` expression (error 4253).

### CASE...OF...ELSE...END_CASE

```st
CASE nCommandID OF
    1:      bMotorOn := TRUE;
    2:      bMotorOn := FALSE;
    3, 4:   bAlarm := TRUE;
    10..20: nSpeed := nCommandID * 10;
ELSE
    bFault := TRUE;
END_CASE;
```
- Selector must be an **integer type** (error 4264).
- Each CASE constant may only be used **once** (error 4270).
- CASE ranges must not overlap (error 4273).
- Only **one** ELSE branch allowed (error 4274).

### FOR...TO...BY...DO...END_FOR

```st
FOR nIndex := 1 TO 100 BY 2 DO
    IF aBuffer[nIndex] = 70 THEN
        nFound := nIndex;
        EXIT;
    END_IF;
END_FOR;
```
- Counter must be an **integer type** (error 4257) with **write access** (error 4258).
- `BY` is optional (defaults to 1).

### WHILE...DO...END_WHILE

```st
nIdx := 0;
WHILE nIdx <= 100 AND aData[nIdx] <> 70 DO
    nIdx := nIdx + 2;
END_WHILE;
```
Condition must be `BOOL` (error 4254).

### REPEAT...UNTIL...END_REPEAT

```st
REPEAT
    nRetryCount := nRetryCount + 1;
    bSuccess := F_TryConnect();
UNTIL bSuccess OR nRetryCount >= 5
END_REPEAT;
```
Executes **at least once**. Condition must be `BOOL` (error 4255).

### EXIT / RETURN

- `EXIT` — exits innermost loop. Error 4262 if outside a loop.
- `RETURN` — exits current POU immediately.

## Variable Declaration Scopes

| Scope | Description | Notes |
| :--- | :--- | :--- |
| `VAR_INPUT` | Input parameters | Read-only inside POU. |
| `VAR_OUTPUT` | Output parameters | **Not allowed in functions** (error 3820). |
| `VAR_IN_OUT` | Pass-by-reference | **Cannot have initial values** (error 3761). Must always be provided (error 4061). |
| `VAR` | Local variables | Static lifetime in FB, temporary in Function. |
| `VAR_TEMP` | Temporary variables | Stack memory, reset every cycle. |
| `VAR RETAIN` | Retained variables | Persisted across power cycles. **No effect in functions** (warning 1410). |
| `VAR PERSISTENT` | Persistent variables | Persisted across power cycles. |
| `VAR CONSTANT` | Constants | Read-only. Cannot be on direct addresses (error 3726). |
| `VAR_EXTERNAL` | External global ref | Must match a globally declared variable (error 3840). |
| `VAR_STAT` | Static variables | Only exist once even with multiple FB instances. |
| `VAR_INST` | Instance variables in methods | Retained between method calls. Only in **methods of FBs**. |
| `VAR_GENERIC CONSTANT` | Generic constant | Integer constant defined at instantiation. Requires **Build 4026+**. |

### VAR_INST Example

```st
METHOD MethLast : INT
VAR_INPUT
    nVar  : INT;
END_VAR
VAR_INST
    nLast : INT := 0;
END_VAR
MethLast := nLast;
nLast    := nVar;
```

### VAR_GENERIC CONSTANT Example

```st
FUNCTION_BLOCK FB_Buffer
VAR_GENERIC CONSTANT
    nMaxLen : UDINT := 1;
END_VAR
VAR
    aData : ARRAY[0..nMaxLen-1] OF BYTE;
END_VAR

VAR
    fbBuf100 : FB_Buffer<100>;
END_VAR
```

## SUPER and THIS

```st
SUPER^();                  (* Call FB body of base class *)
SUPER^.METH_DoIt();        (* Call method of base class *)
THIS^.nVarB := 222;        (* Access FB variable shadowed by local *)
F_FunA(fbMyFb := THIS^);   (* Pass own instance to function *)
```
Only usable in **methods and function block implementations**.

## Direct Addresses

```st
VAR
    bInput1 AT %IX0.0 : BOOL;   (* Digital input, byte 0, bit 0 *)
    nAnalog AT %IW2   : INT;    (* Analog input, word 2 *)
    bOutput AT %QX0.0 : BOOL;   (* Digital output *)
    nMemory AT %MW10  : INT;    (* Memory word 10 *)
END_VAR
```

| Prefix | Area |
| :--- | :--- |
| `%I` | Input |
| `%Q` | Output |
| `%M` | Memory/Marker |
| `X` | Bit | `B` | Byte | `W` | Word | `D` | Double Word |

- Only `BOOL` variables allowed on bit addresses (error 3722).
- No array declaration allowed on direct addresses (error 3727).

## Reserved Keywords (Cap. 15)

Keywords must be written in **CAPITALS** and cannot be used as variable names.

**Control Flow:** `IF`, `THEN`, `ELSIF`, `ELSE`, `END_IF`, `CASE`, `OF`, `END_CASE`, `FOR`, `TO`, `BY`, `DO`, `END_FOR`, `WHILE`, `END_WHILE`, `REPEAT`, `UNTIL`, `END_REPEAT`, `EXIT`, `RETURN`, `CONTINUE`

**Declarations:** `VAR`, `VAR_INPUT`, `VAR_OUTPUT`, `VAR_IN_OUT`, `VAR_GLOBAL`, `VAR_TEMP`, `VAR_STAT`, `VAR_INST`, `VAR_EXTERNAL`, `VAR_CONFIG`, `VAR_GENERIC`, `END_VAR`, `CONSTANT`, `RETAIN`, `PERSISTENT`, `AT`

**POU Types:** `PROGRAM`, `END_PROGRAM`, `FUNCTION`, `END_FUNCTION`, `FUNCTION_BLOCK`, `END_FUNCTION_BLOCK`, `METHOD`, `END_METHOD`, `PROPERTY`, `END_PROPERTY`, `ACTION`, `END_ACTION`, `INTERFACE`, `END_INTERFACE`

**Special (ExST):** `__NEW`, `__DELETE`, `__ISVALIDREF`, `__QUERYINTERFACE`, `__QUERYPOINTER`, `__VARINFO`, `__POUNAME`, `__POSITION`, `__TRY`, `__CATCH`, `__FINALLY`, `__ENDTRY`, `AND_THEN`, `OR_ELSE`

## System Functions

- **Timers**: `TON` (On-Delay), `TOF` (Off-Delay), `TP` (Pulse).
- **Triggers**: `R_TRIG` (Rising Edge), `F_TRIG` (Falling Edge).
- **Counters**: `CTU` (Count Up), `CTD` (Count Down), `CTUD` (Count Up/Down).
- **String**: `CONCAT`, `LEN`, `LEFT`, `RIGHT`, `MID`, `INSERT`, `DELETE`, `REPLACE`, `FIND`.
- **Memory**: `MEMCPY`, `MEMSET`, `MEMMOVE` (in `Tc2_System`).
- **Math**: `ABS`, `SQRT`, `SIN`, `COS`, `TAN`, `ASIN`, `ACOS`, `ATAN`, `EXP`, `LN`, `LOG`.

```st
VAR
    fbTonDelay : TON;
    fbTrigRise : R_TRIG;
END_VAR

fbTonDelay(IN := bStart, PT := T#5S);
IF fbTonDelay.Q THEN
    bDelayedStart := TRUE;
END_IF;

fbTrigRise(CLK := bButton);
IF fbTrigRise.Q THEN
    nPressCount := nPressCount + 1;
END_IF;
```

## CONTINUE Statement

Skips the rest of the current iteration and proceeds with the next loop pass. Valid only inside `FOR`, `WHILE`, `REPEAT`.

```st
FOR i := 1 TO 100 DO
    IF aData[i] = 0 THEN
        CONTINUE;   (* skip zero entries *)
    END_IF;
    nSum := nSum + aData[i];
END_FOR;
```

## Short-Circuit Boolean Operators (ExST)

| Operator | Behavior |
| :--- | :--- |
| `AND_THEN` | Right operand evaluated only if left operand is `TRUE` |
| `OR_ELSE` | Right operand evaluated only if left operand is `FALSE` |

```st
(* Safe pointer dereference using short-circuit *)
IF (pConfig <> 0) AND_THEN (pConfig^.bEnabled) THEN
    fbProcess();
END_IF;
```

Standard `AND` / `OR` always evaluate both sides — risky with side effects (warning 1504/1505).

## CASE with Enumerations

```st
{attribute 'qualified_only'}
{attribute 'strict'}
TYPE E_State : (eIdle, eRun, eStop) END_TYPE

CASE eMyState OF
    E_State.eIdle: ...
    E_State.eRun:  ...
    E_State.eStop: ...
END_CASE;
```

With `{attribute 'strict'}`, the compiler verifies all enum values are handled (or `ELSE` is provided).

## Operands — AT Declarations

```st
VAR_GLOBAL
    bEStop  AT %IX0.0  : BOOL;     (* Bit-aligned input *)
    nCount  AT %MW20   : INT;      (* Memory word *)
    aBuf    AT %MB100  : ARRAY [0..9] OF BYTE;  (* INVALID — error 3727 *)
END_VAR
```

- `AT %I*` and `AT %Q*` declarations are usually replaced in TwinCAT 3 by **link mappings** in the System Manager / `.tsproj` rather than fixed addresses.
- `%M*` (marker area) is preserved across cycles but not retained across power cycles unless declared `RETAIN`.

## References (Beckhoff InfoSys)

- [Operators](https://infosys.beckhoff.com/content/1033/tc3_plc_intro/2528853899.html)
- [Operands](https://infosys.beckhoff.com/content/1033/tc3_plc_intro/2529284235.html)
- [Keywords](https://infosys.beckhoff.com/content/1033/tc3_plc_intro/2529971595.html)
- [System Functions](https://infosys.beckhoff.com/content/1033/tcplccontrol/925653899.html)
