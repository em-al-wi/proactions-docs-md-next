# Control Steps


Control flow steps manage the execution flow of your workflows, including conditional logic, loops, error handling, and parallel execution.


## Available Steps


### [BREAK](./steps/BREAK.md)

Breaks out of the current loop (e.g., WHILE or FOR) based on the given condition.




**Version:** 1.1.0



**Example:**
```yaml

 - step: WHILE
   condition: "{{ flowContext.counter < 10 }}"
   do:
     - step: SET
       counter: "{{ flowContext.counter + 1 }}"
       convertTo: "number"
     - step: BREAK
       condition: "{{ flowContext.counter == 5 }}"
 
```


---


### [CALL_TEMPLATE](./steps/CALL_TEMPLATE.md)

Calls a reusable flow by legacy template name (`name`) or scoped reference (`ref`).


**Aliases:** CALL_FLOW



**Version:** 1.0.6



**Example:**
```yaml

 - step: CALL_TEMPLATE
   name: "myTemplate"
 
```


---


### [DEBUG](./steps/DEBUG.md)

Logs debug information about the current flow context and step configuration. Optionally triggers a debugger breakpoint.




**Version:** 1.0.6



**Example:**
```yaml

 - step: DEBUG
   id: "myDebugPoint"
   omitDebugger: false
 
```


---


### [EMIT_EVENT](./steps/EMIT_EVENT.md)

Emits a custom internal event that can trigger other actions subscribed via action events.


**Aliases:** DISPATCH_EVENT



**Version:** 1.2.0



**Example:**
```yaml

- step: EMIT_EVENT
  name: "custom.contentProcessed"
  payload:
    summary: "{{ flowContext.text }}"

```


---


### [END](./steps/END.md)

Ends the execution of a flow based on the given condition, with optional notification.


**Aliases:** END_IF



**Version:** 1.0.12



**Example:**
```yaml

 - step: END_IF
   condition: "{{errorOccurred}}"
 
```


---


### [FOR](./steps/FOR.md)

Executes a set of steps for a fixed number of iterations (numeric) or iterates over an array (items).




**Version:** 1.1.0



**Example:**
```yaml

 - step: FOR
   var: "i"
   start: 1
   end: 5
   do:
     - step: SET
       iteration: "{{ flowContext.i }}"
 
```


---


### [IF](./steps/IF.md)

Execute a conditional branch based on a condition




**Version:** 1.0.3



**Example:**
```yaml

 - step: IF
   condition: "{{ userInput === 'yes' }}"
   then:
     - step: SHOW_NOTIFICATION
       message: "Proceeding..."
 
```


---


### [PARALLEL](./steps/PARALLEL.md)

Runs multiple step lists in parallel and merges their flow contexts upon completion.




**Version:** 1.0.0



**Example:**
```yaml

 - step: PARALLEL
   steps:
     - # Branch 1
       - step: SET
         branch1Var: "value1"
     - # Branch 2
       - step: SET
         branch2Var: "value2"
 
```


---


### [PLATFORM](./steps/PLATFORM.md)

Routes flow execution to platform-specific branches (`swing`, `prime`, `standalone`) with optional `default` fallback.




**Version:** 1.2.0



**Example:**
```yaml

 - step: PLATFORM
   swing:
     - step: SHOW_NOTIFICATION
       message: "Running Swing path"
   prime:
     - step: SHOW_NOTIFICATION
       message: "Running Prime path"
   default:
     - step: DEBUG
       id: "platform-fallback"
       omitDebugger: true
 
```


---


### [SET](./steps/SET.md)

Sets variables in the flow context. Can execute steps, evaluate expressions, or set key-value pairs.




**Version:** 1.0.3



**Example:**
```yaml

 - step: SET
   userId: "{{ flowContext.currentUser.id }}"
   isActive: true
   count: 5
   convertTo: "number"
 
```


---


### [SLEEP](./steps/SLEEP.md)

Pause execution for a given delay (ms). Useful for pacing flows or waiting for external state changes.




**Version:** 1.0.8



**Example:**
```yaml

- step: SLEEP
  delay: 2000

```


---


### [SWITCH](./steps/SWITCH.md)

Evaluates a switch value and executes the steps for the matching case. Supports both array and object syntax for cases.




**Version:** 1.0.3



**Example:**
```yaml

 - step: SWITCH
   condition: "{{ flowContext.userRole }}"
   cases:
     "admin":
       - step: SHOW_NOTIFICATION
         message: "Welcome Admin"
     "user":
       - step: SHOW_NOTIFICATION
         message: "Welcome User"
     "default":
       - step: SHOW_NOTIFICATION
         message: "Welcome Guest"
 
```


---


### [TRY](./steps/TRY.md)

Executes a set of steps in a try block, and if an error occurs, executes catch steps.




**Version:** 1.1.0



**Example:**
```yaml

 - step: TRY
   try:
     - step: SET
       value: "{{ undefinedVar }}"
   catch:
     - step: SET
       errorHandled: true
 
```


---


### [WHILE](./steps/WHILE.md)

Repeatedly executes a set of steps while a condition evaluates to true.




**Version:** 1.0.3



**Example:**
```yaml

 - step: WHILE
   condition: "{{ flowContext.counter < 10 }} "
   do:
     - step: SET
       counter: "{{ flowContext.counter + 1 }}"
       convertTo: "number"
     - step: SHOW_NOTIFICATION
       message: "Iteration {{flowContext.counter}}"
 
```


---



## Related Documentation

- [Step Library Overview](./overview.md) - Browse all available steps
- [Getting Started](../getting-started/your-first-flow.md) - Learn how to create workflows
