---
name: tc-troubleshooting
description: Use this skill ONLY when the user reports a build error, a compilation warning, or when `CheckPlcSyntaxAsync` fails. It contains specific fixes for TwinCAT IL Errors (4200-4213) and explanations for standard Compiler Warnings (1502-1990).
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

# TwinCAT Troubleshooting — Compiler Errors & Warnings (Cap. 8, 9)

## Compiler Warnings (1xxx)

| Code | Warning Message | Cause / Solution |
| :--- | :--- | :--- |
| **1100** | `Unknown function '<name>' in library` | External library used. Check all functions defined in .hex are also in .lib. |
| **1101** | `Unresolved symbol '<Symbol>'` | Define a function/program with this name. |
| **1200** | `Task '<name>', call of '<name>' Access variables not updated` | Variables used only at FB call in task configuration won't appear in cross reference. |
| **1300** | `File not found '<name>'` | Global variable object points to nonexistent file. Check path. |
| **1302** | `New externally referenced functions inserted. Online Change not possible` | Download complete project instead of online change. |
| **1400** | `Unknown Pragma '<name>' is ignored` | Pragma not supported. Check pragma syntax. |
| **1401** | `Struct '<name>' does not contain any elements` | Empty struct still uses 1 byte. |
| **1410** | `'RETAIN' and 'PERSISTENT' have no effect in functions` | Remanent vars in functions behave as normal local variables. |
| **1412** | `Unexpected token '<name>' in pragma` | Pragma syntax error or used at wrong location. |
| **1501** | `String constant passed as VAR_IN_OUT must not be overwritten` | Constant may not be written inside POU. |
| **1502** | `Variable '<name>' has same name as a POU` | Variable is used, not the POU. Use different names. |
| **1504** | `Statement may not be executed due to logical expression evaluation` | Short-circuit: if first operand is FALSE, second is not evaluated. |
| **1507** | `Instance '<name>' has same name as a function` | The function will be called, not the instance. Use different names. |
| **1550** | `Multiple calls of POU '<name>' in one network may lead to side effects` | Check if multiple calls are necessary. |
| **1850** | `Input variable at %IB used in task '<name>' but updated in another task` | Check task assignments. |
| **1990** | `No 'VAR_CONFIG' for '<name>'` | Missing address configuration. Use 'Insert All instance paths'. |

## Code Generation Errors (3100–3200)

| Code | Error | Solution |
| :--- | :--- | :--- |
| **3101** | `Total data too large` | Reduce data usage. |
| **3117** | `Expression too complex. No more registers` | Split expression using intermediate variables. |
| **3121** | `POU too large. Max 64K` | Split the POU. |
| **3124** | `String constant too large (max 253 chars)` | Reduce string constant length. |
| **3130** | `User-Stack too small` | Reduce POU call nesting depth or increase stack in target settings. |
| **3202** | `Stack overrun with nested string/array/struct function calls` | Split `CONCAT(x, f(i))` into two expressions using intermediate variable. |
| **3210** | `Function '<name>' not found` | Define it or add library. |

## Declaration Errors (3600–3900)

| Code | Error | Solution |
| :--- | :--- | :--- |
| **3612** | `Maximum number of POUs exceeded` | Modify maximum in Target Settings / Memory Layout. |
| **3614** | `Project must contain a POU named '<name>' or a task configuration` | Create main POU of type `PROGRAM` (e.g., `PLC_PRG`). |
| **3615** | `<name> (main routine) must be of type program` | Init POU must be `PROGRAM`. |
| **3701** | `Name used in interface not identical with POU Name` | Use 'Rename object' or change declaration name to match. |
| **3703** | `Duplicate definition of identifier '<name>'` | Rename the variable or object. |
| **3704** | `Data recursion` | An FB instance uses itself directly or indirectly. |
| **3705** | `VAR_IN_OUT in Top-Level-POU not allowed without Task-Configuration` | Create task configuration or remove `VAR_IN_OUT` from `PLC_PRG`. |
| **3720** | `Address expected after 'AT'` | Add valid address (`%I*`, `%Q*`, `%M*`) after `AT`. |
| **3721** | `Only 'VAR' and 'VAR_GLOBAL' can be located to addresses` | Move to `VAR` or `VAR_GLOBAL`. |
| **3722** | `Only 'BOOL' variables allowed on bit addresses` | Modify address or change variable type. |
| **3726** | `Constants cannot be laid on direct addresses` | Remove address from constant. |
| **3727** | `No array declaration allowed on this address` | Remove address from array. |
| **3740** | `Invalid type '<name>'` | Define type or check for typos. Ensure library is referenced. |
| **3744** | `Enum constant '<name>' already defined` | Enum values must be unique within enum and across global/local enums. |
| **3745** | `Subranges are only allowed on Integers` | Use integer base type. |
| **3748** | `More than three dimensions not allowed for arrays` | Use `ARRAY OF ARRAY` (max 9 via nesting). |
| **3760** | `Error in initial value` | Initial value doesn't match type definition. |
| **3761** | `VAR_IN_OUT variables must not have initial value` | Remove `:= value` from `VAR_IN_OUT`. |
| **3782** | `Unexpected end` | Add `END_VAR` or terminating keyword (`END_IF`, etc.). |
| **3802** | `Out of retain memory` | Retain memory exhausted. |
| **3820** | `VAR_OUTPUT and VAR_IN_OUT not allowed in functions` | Functions cannot have output or in-out variables. |
| **3821** | `At least one input required for functions` | Add at least one `VAR_INPUT`. |
| **3840** | `Unknown global variable '<name>'` | `VAR_EXTERNAL` references nonexistent global. |
| **3841** | `Declaration of '<name>' doesn't match global declaration` | `VAR_EXTERNAL` type must match global declaration. |
| **3850** | `Unpacked struct inside packed struct not allowed` | Causes memory misalignment. Modify struct definition. |
| **3902** | `Keywords must be uppercase` | Use capitals. |
| **3903** | `Invalid duration constant` | Use correct format: `T#100ms`, `T#2s`. |
| **3904** | `Overflow in duration constant` | Value exceeds `TIME` max (~49 days). |

## Expression & Statement Errors (4000–4370)

| Code | Error | Solution |
| :--- | :--- | :--- |
| **4001** | `Variable '<name>' not declared` | Declare variable locally or globally. |
| **4010** | `Type mismatch: Cannot convert '<t1>' to '<t2>'` | Add explicit conversion function. |
| **4011** | `Type mismatch in parameter '<name>'` | Use type conversion or correct variable. |
| **4017** | `'MOD' is not defined for 'REAL'` | Use `MOD` only with integer/bitstring types. |
| **4021** | `No write access to variable '<name>'` | Use writable variable. |
| **4029** | `Nested calls of same function not possible` | Use intermediate variable: `nTemp := fun1(b,c); nResult := fun1(a, nTemp);` |
| **4030** | `Expressions and constants not allowed as operands of ADR` | Use a variable or direct address. |
| **4031** | `ADR not allowed on bits! Use BITADR` | Replace `ADR` with `BITADR`. |
| **4034** | `Division by 0` | Division by zero in constant expression. |
| **4050** | `POU '<name>' is not defined` | Define the POU or correct the name. |
| **4052** | `'<name>' must be a declared instance of FB '<name>'` | Declare an instance of the correct FB type. |
| **4060** | `VAR_IN_OUT parameter needs variable with write access` | Provide writable variable. |
| **4061** | `VAR_IN_OUT parameter must be used` | Always provide a variable for `VAR_IN_OUT` parameters. |
| **4070** | `POU contains too complex expression` | Simplify using intermediate variables. |
| **4110** | `'^' needs a pointer type` | Declare variable as `POINTER TO`. |
| **4111** | `'[index]' needs array variable` | Indexing only works on `ARRAY OF` variables. |
| **4114** | `Constant index not within array range` | Fix index to be within declared bounds. |
| **4120** | `'.' needs structure variable` | Left side of `.` must be `STRUCT`, `FUNCTION_BLOCK`, `FUNCTION`, or `PROGRAM`. |
| **4121** | `'<name>' is not a component of <object>` | Component not found in struct/FB definition. |
| **4250** | `Another ST statement or end of POU expected` | Line doesn't start with valid ST instruction. |
| **4251** | `Too many parameters in function '<name>'` | More parameters than declared. |
| **4252** | `Too few parameters in function '<name>'` | Fewer parameters than declared. |
| **4253** | `IF/ELSIF requires BOOL expression` | Condition must evaluate to `BOOL`. |
| **4257** | `FOR variable must be of type INT` | Counter must be integer or bitstring type. |
| **4258** | `FOR expression is no variable with write access` | Counter must be writable. |
| **4262** | `EXIT outside a loop` | `EXIT` only inside `FOR`, `WHILE`, or `REPEAT`. |
| **4264** | `CASE requires selector of integer type` | Use integer/bitstring type for `CASE` selector. |
| **4267** | `FB call requires function block instance` | Declare an instance of the FB. |
| **4270** | `CASE constant '<n>' already used` | Each selector value used only once. |
| **4271** | `Lower border greater than upper border` | Fix range bounds. |
| **4273** | `CASE range '<a>..<b>' already used in '<c>..<d>'` | Ranges must not overlap. |
| **4274** | `Multiple ELSE branch in CASE statement` | Only one `ELSE` permitted. |

## Task Configuration Errors (3550–3575)

| Code | Error | Solution |
| :--- | :--- | :--- |
| **3550** | `Duplicate definition of identifier '<name>'` | Rename one task. |
| **3551** | `Task '<name>' must contain at least one program call` | Insert a program call or delete the task. |
| **3553** | `Event variable '<name>' must be of type BOOL` | Use a `BOOL` variable for event trigger. |
| **3557** | `Maximum number of Tasks exceeded` | Reduce task count to target limit. |
| **3570** | `Tasks '<a>' and '<b>' share the same priority` | Each task must have a unique priority. |
| **3575** | `Task '<name>': cycle time must be multiple of <n> µs` | Adjust cycle time. |

## SFC Errors (4350–4369)

| Code | Error | Solution |
| :--- | :--- | :--- |
| **4353** | `Step name duplicated: '<name>'` | Rename one of the duplicate steps. |
| **4354** | `Jump to undefined Step: '<name>'` | Define target step or use existing step name. |
| **4355** | `Transition must not have side effects` | Transition must be a pure boolean expression. |
| **4358** | `Action not declared: '<name>'` | Add action below SFC POU and set name in qualifier box. |
| **4359** | `Invalid Qualifier: '<name>'` | Use valid qualifier (N, R, S, L, D, P, SD, DS, SL). |
| **4364** | `Transition must be a boolean expression` | Result of transition must be `BOOL`. |

## IL Errors (4200–4213)

| Code | Error | Solution |
| :--- | :--- | :--- |
| **4201** | `IL Operator expected` | Each IL instruction must start with operator or jump label. |
| **4202** | `Unexpected end of text in brackets` | Insert closing bracket. |
| **4203** | `<name> in brackets not allowed` | `JMP`, `RET`, `CAL`, `LDN`, `LD`, `TIME` not valid in bracket expressions. |
| **4204** | `Closing bracket with no opening bracket` | Insert opening bracket or remove closing one. |
| **4207** | `'N' modifier requires BOOL, BYTE, WORD or DWORD` | `N` modifier needs a type supporting boolean negation. |
| **4208** | `Conditional operator requires type BOOL` | Expression must yield boolean result. |
| **4210** | `CAL, CALC, CALN require FB instance as operand` | Declare an instance of the function block. |
| **4213** | `S and R require BOOL operand` | Use a boolean variable. |

## Diagnostic Workflow

1. Run **`plc_check_syntax`** first for immediate syntax feedback, then run **`session_build`**.
2. Treat **`session_build`** as the source of truth for the current project state: `plc_check_syntax` can still report stale errors until a new build refreshes the diagnostics.
3. Read the **Error List** — fix errors **top-down** (later errors are often cascades of the first).
4. For each error code: look it up in the tables above, identify the offending POU/line, apply the indicated fix.
5. Run **`session_build`** again after each fix — never batch multiple speculative fixes.
6. **Never login** to a PLC with unresolved errors (rule 0.5 in the master agent).

## Common Pitfalls

- **Cascade errors**: a single missing `END_VAR` (3782) can produce dozens of subsequent errors — fix the first one and rebuild.
- **Stale syntax diagnostics**: if `plc_check_syntax` still shows old errors after a fix, run `session_build` before trusting the result list.
- **Type mismatch in batch edits**: when refactoring data types, use `plc_grep_pou` to find ALL references before editing.
- **Online change blocked**: warning 1302 means new external references — perform a full download instead.
- **Retain memory exhausted (3802)**: an FB instance with even one `VAR RETAIN` member places the entire instance in retain area; split into separate FBs.
- **Padding-induced size mismatch**: structs sent over EtherCAT/ADS must use `{attribute 'pack_mode' := '1'}` to match external definitions.

## References (Beckhoff InfoSys)

- [Compiler Errors](https://infosys.beckhoff.com/content/1033/tcplccontrol/925418891.html)
- [System Functions](https://infosys.beckhoff.com/content/1033/tcplccontrol/925653899.html)
