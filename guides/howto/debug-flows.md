# How to Debug Flows

Debugging ProActions workflows is essential for identifying and fixing issues. This guide covers debugging techniques, tools, and best practices.

## DEBUG Step

The `DEBUG` step logs flow context and variables to the console.

### Basic Debug

```yaml
- title: 'Debug Example'
  flow:
    - step: SET
      myVariable: "Hello World"
      count: 42

    - step: DEBUG

    - step: HUB_COMPLETION
      instruction: "Process: {textContent}"
```

**Console Output:**

```
[DEBUG] Current flow context:
{
  myVariable: "Hello World",
  count: 42
}
```

### Debug at Strategic Points

Place DEBUG steps at key points in your flow:

```yaml
- title: 'Complex Workflow'
  flow:
    - step: GET_TEXT_CONTENT
      at: CURSOR

    - step: DEBUG

    - step: HUB_COMPLETION
      instruction: "Process: {selectedText}"

    - step: DEBUG

    - step: REPLACE_TEXT
      at: CURSOR
```

## Common Debugging Scenarios

### 1. Variable Not Set

**Problem:** Variable shows as undefined

```yaml
- step: SHOW_NOTIFICATION
  message: "Value: {{ flowContext.myVar }}"
  # Shows: "Value: undefined"
```

**Debug:**

```yaml
- step: DEBUG
  message: "Check if myVar exists"

- step: SHOW_NOTIFICATION
  message: "Value: {{ flowContext.myVar }}"
```

**Solution:** Ensure variable is set before use:

```yaml
- step: SET
  myVar: "Hello"

- step: SHOW_NOTIFICATION
  message: "Value: {{ flowContext.myVar }}"
```

### 2. Service Not Responding

**Problem:** LLM step hangs or fails

```yaml
- step: HUB_COMPLETION
  instruction: "Process this"
  # Times out or errors
```

**Debug:**

```yaml
- step: TRY
  try:
    - step: DEBUG
      message: "Before LLM call"

    - step: HUB_COMPLETION
      instruction: "Test"
      max_tokens: 10

    - step: DEBUG
      message: "After LLM call - response received"
  catch:
    - step: DEBUG
      message: "Error occurred"

    - step: SHOW_NOTIFICATION
      message: "Service error: {{ flowContext.error.message }}"
```

**Check:**

- ProActions Hub is running
- API keys are configured
- Network connectivity
- Browser network tab for failed requests

### 3. SCRIPTING Step Errors

**Problem:** Script execution fails with cryptic error

```yaml
- step: SCRIPTING
  script: |
    flowContext.result = params.data.toUpperCase();
    return flowContext;
  # Error: Cannot read properties of undefined
```

**Debug:**

Check the console for detailed SCRIPTING error log showing:
- What variables are available
- Where in the script the error occurred
- The full error message and stack trace

**Solution:** Verify variables exist before using:

```yaml
- step: SCRIPTING
  script: |
    if (!params?.data) {
      throw new Error("params.data is required");
    }
    flowContext.result = params.data.toUpperCase();
    return flowContext;
```

Or use optional chaining and defaults:

```yaml
- step: SCRIPTING
  script: |
    flowContext.result = (params?.data ?? '').toUpperCase();
    return flowContext;
```

## Debugging Techniques

### 1. Simplify the Flow

Start with a minimal flow and add complexity:

```yaml
# Start simple
- title: 'Test Basic'
  flow:
    - step: SHOW_NOTIFICATION
      message: "Flow started"

# Add one step at a time
- title: 'Test With LLM'
  flow:
    - step: SHOW_NOTIFICATION
      message: "Flow started"

    - step: HUB_COMPLETION
      instruction: "Say hello"
      options:
        max_tokens: 5

    - step: SHOW_NOTIFICATION
      message: "LLM response: {{ flowContext.text }}"
```

### 2. Test Each Step Independently

Create test actions for individual steps:

```yaml
- title: 'Test GET_TEXT_CONTENT'
  flow:
    - step: GET_TEXT_CONTENT
      at: CURSOR

    - step: SHOW_NOTIFICATION
      message: "Selected: {selectedText}"

- title: 'Test HUB_COMPLETION'
  flow:
    - step: HUB_COMPLETION
      instruction: "Say hello"
      options:
        max_tokens: 5

    - step: SHOW_NOTIFICATION
      message: "Response: {{ flowContext.text }}"
```

### 3. Use SHOW_NOTIFICATION for Quick Checks

```yaml
- title: 'Quick Check'
  flow:
    - step: SET
      name: "John"
      age: 30

    - step: SHOW_NOTIFICATION
      message: "Name: {{ flowContext.name }}, Age: {{ flowContext.age }}"
```

### 4. Log to Console Manually

```yaml
- step: SCRIPTING
  script: |
    console.log('[Custom Debug] Variable check:', {
      textContent: flowContext.textContent?.substring(0, 100),
      userName: flowContext.userName,
      customVar: flowContext.customVar
    });
    return flowContext;
```

### 5. SCRIPTING Step Error Logging

The SCRIPTING step provides detailed error logs when script execution fails:

```yaml
- step: SCRIPTING
  script: |
    // This will fail with a helpful error message
    const result = flowContext.undefinedVar.someMethod();
    return flowContext;
```

**Console Output:**

```
======================================================================
🐛 SCRIPTING ERROR in action: "My Action Name"
======================================================================

Type: script
Mode: sync

Script Preview:
    1 | // This will fail with a helpful error message
    2 | const result = flowContext.undefinedVar.someMethod();
    3 | return flowContext;

Available Context:
  - Context keys: step, client, cfg, flowContext, ctx
  - FlowContext keys: text, selectedText, myVariable

Error Details:
  Message: Cannot read properties of undefined (reading 'someMethod')

Stack Trace:
TypeError: Cannot read properties of undefined (reading 'someMethod')
    at eval (eval at evalInScope ...)
    ...
======================================================================
```

**What the error log shows:**

- **Action name**: Which action the error occurred in
- **Type and Mode**: Whether it's script/template and sync/async
- **Script preview**: First 20 lines of the failing script with line numbers
- **Available context**: What variables are accessible in the script
- **Error details**: The actual error message and stack trace

**Debugging script errors:**

1. Check the "FlowContext keys" to verify variables exist
2. Look at the script preview line numbers to locate the error
3. Use the error message to understand what went wrong
4. Add `console.log()` statements before the failing line:

```yaml
- step: SCRIPTING
  script: |
    console.log('Checking variable:', flowContext.undefinedVar);
    const result = flowContext.undefinedVar?.someMethod() ?? 'default';
    return flowContext;
```

## Debugging Configuration Issues

### YAML Syntax Errors

**Symptoms:**

- Actions don't appear
- Console shows parsing errors

**Debug:**

1. Check browser console for error messages
2. Validate YAML syntax with online validator
3. Check indentation (use spaces, not tabs)
4. Ensure quotes are properly closed

**Example Error:**

```
YAML parse error at line 15: unexpected token
```

**Solution:**

- Go to line 15 in YAML file
- Check for missing colon, dash, or quote
- Verify indentation is correct

### Include Path Errors

**Symptoms:**

- Sections not loading
- Console shows errors

**Debug:**

```yaml
# Check if file exists
AI_KIT:
  SECTIONS:
    - !include /SysConfig/ProActions/sections/my-actions.ai-kit.section.yaml
```

**Solution:**

- Verify the include path format is valid:
  - absolute (`/SysConfig/...`), or
  - relative with `./` / `../` (resolved from the current file)
- Check file exists and is readable
- Ensure proper file permissions

### Service Configuration Issues

**Symptoms:**

- Service steps fail
- "Service not configured" errors

**Debug:**

```yaml
- title: 'Test Service Config'
  flow:
    - step: TRY
      try:
        - step: HUB_COMPLETION
          instruction: "Test"

        - step: SHOW_NOTIFICATION
          notificationType: success
          message: "Service working"
      catch:
        - step: SHOW_NOTIFICATION
          notificationType: error
          message: "Service error: Check configuration"
```

**Check:**

- Service is defined in SERVICES section
- Endpoint URLs are correct
- ProActions Hub is running (if using Hub)

## Browser Network Tab

Use the network tab to debug API calls:

### 1. Open Network Tab

1. Open browser DevTools (F12)
2. Click "Network" tab
3. Execute your ProActions workflow

### 2. Filter API Requests

- Filter by "XHR" or "Fetch"
- Look for requests to your services
- Check for 401, 403, 404, 500 errors

### 3. Inspect Request/Response

Click on a request to see:

- **Headers**: Check authentication headers
- **Payload**: Request body sent to API
- **Response**: API response or error message
- **Timing**: How long the request took

## Common Error Messages

### "Step not found: STEP_NAME"

**Cause:** Step type doesn't exist or is misspelled

**Solution:** Check step name spelling, verify step is registered

### "Variable undefined"

**Cause:** Variable not set before use

**Solution:** Add DEBUG step to check flow context, ensure variable is set

### "Service not configured"

**Cause:** Service not defined in SERVICES section

**Solution:** Check service configuration in services.ai-kit.services.yaml

### "Permission denied"

**Cause:** User doesn't have access to action

**Solution:** Check forUsers, forGroups, forTeams in action configuration

### "XPath not found"

**Cause:** XPath doesn't match document structure

**Solution:** Verify XPath with browser inspector, simplify XPath

## Debugging Best Practices

### 1. Start Simple, Add Complexity

```yaml
# Start with this
- title: 'Test'
  flow:
    - step: SHOW_NOTIFICATION
      message: "It works!"

# Then add
- title: 'Test with Variables'
  flow:
    - step: SET
      myVar: "Hello"

    - step: SHOW_NOTIFICATION
      message: "{{ flowContext.myVar }}"

# Then add more
- title: 'Test Full Flow'
  flow:
    - step: SET
      myVar: "Hello"

    - step: HUB_COMPLETION
      instruction: "Process: {{ flowContext.myVar }}"

    - step: SHOW_NOTIFICATION
      message: "{{ flowContext.text }}"
```

### 2. Use Descriptive Debug Messages

```yaml
# Good
- step: DEBUG
  message: "After fetching text content - checking selectedText"

# Avoid
- step: DEBUG
  message: "Debug 1"
```

### 3. Remove Debug Steps in Production

Comment out or remove DEBUG steps before deploying:

```yaml
- title: 'Production Flow'
  flow:
    # - step: DEBUG
    #   message: "Debug: Starting flow"

    - step: HUB_COMPLETION
      instruction: "Process: {textContent}"
```

### 4. Use TRY/Catch for Error Handling

```yaml
- step: TRY
  try:
    - step: HUB_COMPLETION
      instruction: "Process: {textContent}"
  catch:
    - step: DEBUG
      message: "Error details"

    - step: SHOW_NOTIFICATION
      notificationType: error
      message: "An error occurred"
```

### 5. Test with Sample Data

Create test actions with hardcoded data:

```yaml
- title: 'Test with Sample Data'
  flow:
    - step: SET
      testText: "This is sample text for testing"
      testCount: 5

    - step: HUB_COMPLETION
      instruction: "Process: {{ flowContext.testText }}"

    - step: SHOW_NOTIFICATION
      message: "Result: {{ flowContext.text }}"
```

## Debugging Checklist

When a flow doesn't work:

- [ ] Check browser console for errors
- [ ] Verify YAML syntax is valid
- [ ] Add DEBUG steps to inspect variables
- [ ] Test with simplified flow
- [ ] Check service configuration
- [ ] Verify permissions (forUsers, forGroups, forTeams)
- [ ] Test individual steps separately
- [ ] Check network tab for API errors
- [ ] Verify XPaths match document structure
- [ ] Ensure development mode is enabled
- [ ] Clear configuration cache
- [ ] Refresh browser

## Next Steps

- **[Use Templates](./use-nunjucks-templates.md)** - Template syntax and functions
- **[Configuration Basics](../configuration/basics.md)** - Configuration fundamentals
