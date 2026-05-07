---
name: tc-advanced-patterns
description: Use this skill when implementing advanced TwinCAT 3 (ExST) features. It covers Properties (GET/SET), Methods, initialization lifecycles (FB_init, FB_reinit, FB_exit), State Machine patterns, pragmas (attributes, conditional compiling, regions, message output), and PlcTaskSystemInfo access.
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

# TwinCAT Advanced Patterns (Cap. 10, 11, 12)

## Pointers and References

```st
VAR
    stConfig  : ST_MachineConfig;
    pConfig   : POINTER TO ST_MachineConfig;
    refConfig : REFERENCE TO ST_MachineConfig;
    nSize     : UDINT;
END_VAR

pConfig := ADR(stConfig);
IF pConfig <> 0 THEN
    pConfig^.nSpeed := 100;
END_IF;
nSize := SIZEOF(stConfig);

refConfig REF= stConfig;
refConfig.nSpeed := 100;
```

## State Machine Pattern

```st
FUNCTION_BLOCK FB_StateMachine
VAR
    eState : E_MachineState := E_MachineState.IDLE;
END_VAR

CASE eState OF
    E_MachineState.IDLE:
        IF bStartCmd THEN
            eState := E_MachineState.INIT;
        END_IF;

    E_MachineState.INIT:
        IF bInitDone THEN
            eState := E_MachineState.RUNNING;
        ELSIF bError THEN
            eState := E_MachineState.ERROR;
        END_IF;

    E_MachineState.RUNNING:
        IF bStopCmd THEN
            eState := E_MachineState.STOPPING;
        END_IF;

    E_MachineState.STOPPING:
        IF bStopped THEN
            eState := E_MachineState.IDLE;
        END_IF;

    E_MachineState.ERROR:
        IF bReset THEN
            eState := E_MachineState.IDLE;
        END_IF;
END_CASE;
```

## Property (GET/SET) — TwinCAT 3 Extension

```st
FUNCTION_BLOCK FB_Motor
VAR
    fSpeed : REAL;   (* backing field: Hungarian notation, no leading _ *)
END_VAR

PROPERTY Speed : REAL    (* Property name: PascalCase, no Hungarian prefix *)

(* GET accessor *)
Speed GET:
    Speed := fSpeed;

(* SET accessor *)
Speed SET:
    IF Speed >= 0.0 AND Speed <= 3000.0 THEN
        fSpeed := Speed;
    END_IF;
```

## Interface Implementation — TwinCAT 3 Extension

```st
INTERFACE I_Actuator
METHOD Start : BOOL
METHOD Stop  : BOOL
END_INTERFACE

FUNCTION_BLOCK FB_Valve IMPLEMENTS I_Actuator
VAR
    bOpen : BOOL;
END_VAR

METHOD Start : BOOL
    bOpen := TRUE;
    Start := TRUE;
END_METHOD

METHOD Stop : BOOL
    bOpen := FALSE;
    Stop := TRUE;
END_METHOD
```

## Methods

```st
FUNCTION_BLOCK FB_Buffer
VAR
    aData  : ARRAY [0..99] OF INT;
    nCount : INT := 0;
END_VAR

METHOD Add : BOOL
VAR_INPUT
    nValue : INT;
END_VAR
    IF nCount < 100 THEN
        aData[nCount] := nValue;
        nCount := nCount + 1;
        Add := TRUE;
    ELSE
        Add := FALSE;
    END_IF;
END_METHOD

METHOD Clear : BOOL
    MEMSET(ADR(aData), 0, SIZEOF(aData));
    nCount := 0;
    Clear := TRUE;
END_METHOD
```

## Pragmas & Attributes

### Message Pragmas

```st
{text 'Part compiled'}
{info 'TODO: rename'}
{warning 'Deprecated usage'}
{error 'Not implemented'}      (* Stops compilation *)
```

### Key Attribute Pragmas

| Attribute | Effect |
| :--- | :--- |
| `{attribute 'pack_mode' := '1'}` | Control struct alignment (0=8byte, 1=1byte, 2=2byte, 4=4byte) |
| `{attribute 'qualified_only'}` | Force `GVL_Name.VarName` / `E_Name.eValue` syntax |
| `{attribute 'strict'}` | Strict type checking for enums |
| `{attribute 'enable_dynamic_creation'}` | Required for `__NEW` dynamic allocation |
| `{attribute 'call_after_init'}` | Method called after `FB_init` and external assignments |
| `{attribute 'call_after_online_change_slot'}` | Method called after Online Change slot processing |
| `{attribute 'call_on_type_change'}` | Method called when type changes during Online Change |
| `{attribute 'no_copy'}` | Skip variable during Online Change copy |
| `{attribute 'no-exit'}` | Suppress `FB_exit` call |
| `{attribute 'noinit'}` | Skip initialization |
| `{attribute 'obsolete'}` | Mark as deprecated (generates warning) |
| `{attribute 'instance-path'}` | Get instance path as STRING |
| `{attribute 'hide'}` | Hide from symbol information |
| `{attribute 'TcRpcEnable'}` | Enable RPC for methods |
| `{attribute 'linkalways'}` | Always link even if unused |

### Conditional Pragmas

```st
{define VARIANT1}
{define VERSION '2.0'}

{IF defined(VARIANT1)}
    sMode : STRING := 'Variant1';
{ELSIF hasvalue(VERSION, '2.0')}
    sMode : STRING := 'V2';
{ELSE}
    sMode : STRING := 'Default';
{END_IF}
```

| Operator | Usage |
| :--- | :--- |
| `defined(<id>)` | Check if define exists |
| `defined(variable: <var>)` | Check if variable declared (impl only) |
| `defined(type: <type>)` | Check if type exists (impl only) |
| `defined(pou: <pou>)` | Check if POU exists (impl only) |
| `hasvalue(<id>, '<val>')` | Check define value |
| `hasattribute(pou: <p>, '<a>')` | Check POU attribute (impl only) |
| `hastype(variable: <v>, <t>)` | Check variable type (impl only) |

### Region Pragma

```st
{region "Motor Control"}
    fbMotor(bStart := bCmd);
    IF fbMotor.bFault THEN
        eState := E_State.ERROR;
    END_IF;
{endregion}
```

### Warning Suppression

```st
{warning disable C0195}
    (* code that would normally trigger warning C0195 *)
{warning restore C0195}
```

## FB_init, FB_reinit and FB_exit (Cap. 11)

### FB_init

Implicitly available. Called by TwinCAT when FB instance is initialized. **Do not call manually.**

```st
METHOD FB_init : BOOL
VAR_INPUT
    bInitRetains : BOOL;  (* TRUE = retain vars initialized *)
    bInCopyCode  : BOOL;  (* TRUE = instance is in copy code (online change) *)
    nComNum      : INT;   (* Additional custom input *)
END_VAR
```

**Parameter values by operating case:**

| Case | `bInitRetains` | `bInCopyCode` |
| :--- | :--- | :--- |
| First/New Download | `TRUE` | `FALSE` |
| Online Change | `FALSE` | `TRUE` |
| TwinCAT Restart | `FALSE` | `FALSE` |

- **No** `SUPER^.FB_init` call allowed.
- Additional inputs set in instance declaration: `fbDev : FB_Device(nComNum := 1);`

### FB_reinit

Called automatically **after** instance copy during Online Change (when declaration changes).

```st
METHOD FB_reinit : BOOL
(* For derived FBs: must explicitly call SUPER^.FB_reinit() *)
```

### FB_exit

Called automatically **before** instance removal.

```st
METHOD FB_exit : BOOL
VAR_INPUT
    bInCopyCode : BOOL;  (* TRUE = online change; FALSE = download/shutdown *)
END_VAR
(* Use to: release __DELETE memory, close handles, etc. *)
(* For derived FBs: derived FB_exit executes first, then base. *)
```

### Operating Cases — Call Sequence

| Step | First Download | New Download | Online Change |
| :--- | :--- | :--- | :--- |
| 1 | `FB_init` | `FB_exit` | `FB_exit` |
| 2 | External init | `FB_init` | `FB_init` |
| 3 | `call_after_init` | External init | External init |
| 4 | — | `call_after_init` | `call_after_init` |
| 5 | — | — | Copy process |
| 6 | — | — | `FB_reinit` |

- **Online Change with only implementation changes**: `FB_exit`/`FB_init`/`FB_reinit` are **NOT** called.
- POINTER/REFERENCE variables may become invalid after Online Change — use `{attribute 'call_on_type_change'}`.

## Global Data Types (Cap. 12)

### PlcAppSystemInfo — `_AppInfo`

| Field | Type | Description |
| :--- | :--- | :--- |
| `AdsPort` | `UINT` | ADS port of PLC application |
| `BootDataLoaded` | `BOOL` | PERSISTENT variables loaded without error |
| `OldBootData` | `BOOL` | PERSISTENT variables invalid (backup loaded) |
| `AppTimestamp` | `DT` | Compile time of PLC application |
| `OnlineChangeCnt` | `UDINT` | Online changes since last download |
| `ProjectName` | `STRING(63)` | Project name |

```st
nPort   := _AppInfo.AdsPort;
bLoaded := _AppInfo.BootDataLoaded;
```

### PlcTaskSystemInfo — `_TaskInfo[]`

```st
nIdx := GETCURTASKINDEXEX();
IF nIdx > 0 THEN
    nCycleUs := _TaskInfo[nIdx].CycleTime / 10;  (* convert to µs *)
END_IF;
```

| Field | Type | Description |
| :--- | :--- | :--- |
| `CycleTime` | `UDINT` | Set task cycle time (× 100 ns) |
| `CycleCount` | `UDINT` | Cycle counter |
| `LastExecTime` | `UDINT` | Last cycle execution time (× 100 ns) |
| `FirstCycle` | `BOOL` | TRUE during first PLC task cycle |
| `CycleTimeExceeded` | `BOOL` | TRUE if cycle time was exceeded |

## Action (TwinCAT 3 Extension)

Actions are reusable sub-routines bound to a parent POU (FB or program). They share the parent's local variable scope — no own `VAR_INPUT`/`VAR_OUTPUT`.

```st
FUNCTION_BLOCK FB_Pump
VAR
    bRunning : BOOL;
    nFaultCode : DINT;
END_VAR

ACTION ResetFault
    nFaultCode := 0;
    bRunning   := FALSE;
END_ACTION

(* Body: *)
IF bResetCmd THEN
    ResetFault();   (* Local action call *)
END_IF;
```

- Use `ACTION` for code reuse without parameter overhead.
- Use `METHOD` when you need parameters or a return value.
- An action **cannot be called from outside** the owning POU.

## Additional Attribute Pragmas

| Attribute | Effect |
| :--- | :--- |
| `{attribute 'monitoring' := 'variable'}` | Force watch monitoring as plain variable |
| `{attribute 'monitoring' := 'call'}` | Watch via implicit getter call |
| `{attribute 'init_on_onlchange'}` | Re-initialize on online change |
| `{attribute 'analysis' := '-33'}` | Disable specific static analysis rule |
| `{attribute 'global_init_slot' := 'N'}` | Force init order across GVLs |
| `{attribute 'C++_Compatible'}` | Force C++-compatible struct layout |
| `{attribute 'reflection'}` | Enable runtime reflection metadata |
| `{attribute 'symbol' := 'read write'}` | Control ADS symbol exposure |

## call_after_init Example

```st
FUNCTION_BLOCK FB_Module
VAR
    bConfigured : BOOL;
END_VAR

{attribute 'call_after_init'}
METHOD AfterInit
    bConfigured := TRUE;   (* Runs after FB_init AND after instance assignments *)
END_METHOD
```

Useful when initialization depends on values written in the instance declaration that aren't yet available inside `FB_init` itself.

## References (Beckhoff InfoSys)

- [Pragmas](https://infosys.beckhoff.com/content/1033/tc3_plc_intro/2529556363.html)
- [FB_init, FB_reinit, FB_exit](https://infosys.beckhoff.com/content/1033/tc3_plc_intro/5044757003.html)
- [Global Data Types — System](https://infosys.beckhoff.com/content/1033/tc3_plc_intro/10394151819.html)
