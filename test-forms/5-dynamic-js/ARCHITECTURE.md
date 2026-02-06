# Dynamic JS Rendering Architecture

## Overview
A form where all fields are rendered dynamically via JavaScript after page load. The HTML contains only an empty container - no form fields exist until JavaScript executes.

## Purpose
Test the ghostwriter agent's ability to:
- Wait for JavaScript to execute and render content
- Detect dynamically created form elements
- Handle forms that don't exist in the initial HTML
- Work with modern SPA-style interfaces

## Structure

```
5-dynamic-js/
├── index.html        # Contains JS that renders the form
├── confirmation.html # Success page
└── ARCHITECTURE.md   # This file
```

## How It Works

### Initial HTML State
```html
<div id="loading">
  <div class="spinner"></div>
  <p>Loading application form...</p>
</div>

<div id="form-container"></div>
<!-- No form fields in HTML! -->
```

### After JavaScript Executes (500ms)
```html
<div id="loading" style="display: none;"></div>

<div id="form-container">
  <form id="application-form">
    <div class="form-section">
      <h2>Personal Information</h2>
      <div class="form-group">
        <label for="full_name" class="required">Full Name</label>
        <input type="text" id="full_name" name="full_name" required>
      </div>
      <!-- ... more fields ... -->
    </div>
  </form>
</div>
```

## Form Configuration

The form is defined as a JavaScript object:

```javascript
const formConfig = {
  sections: [
    {
      title: 'Personal Information',
      fields: [
        { name: 'full_name', label: 'Full Name', type: 'text', required: true },
        // ...
      ]
    },
    // ...
  ]
};
```

## Field Types Supported

| Type | Renders As | Notes |
|------|------------|-------|
| text | `<input type="text">` | Standard text input |
| email | `<input type="email">` | Email with validation |
| tel | `<input type="tel">` | Phone number |
| url | `<input type="url">` | URL with validation |
| file | `<input type="file">` | File upload |
| select | `<select>` | Single-select dropdown |
| multiselect | `<select multiple>` | Multi-select dropdown |
| checkbox_group | Multiple `<input type="checkbox">` | Group of checkboxes |
| radio | Multiple `<input type="radio">` | Radio button group |
| textarea | `<textarea>` | Multi-line text |

## Field Inventory

### Section 1: Personal Information (4 fields)
| Field | Type | Required |
|-------|------|----------|
| Full Name | text | Yes |
| Email Address | email | Yes |
| Phone Number | tel | Yes |
| LinkedIn Profile | url | No |

### Section 2: Technical Skills (5 fields)
| Field | Type | Required |
|-------|------|----------|
| Primary Language | select | Yes |
| Frontend Frameworks | multiselect | No |
| Backend Frameworks | multiselect | No |
| Database Experience | checkbox_group | No |
| Years Experience | select | Yes |

### Section 3: Additional Information (5 fields)
| Field | Type | Required |
|-------|------|----------|
| Portfolio/GitHub URL | url | No |
| Resume | file | Yes |
| Cover Letter | textarea | No |
| Start Availability | select | Yes |
| Work Preference | radio | Yes |

**Total Fields: 14**

## Technical Implementation

### Render Timing
```javascript
// Simulate API delay
setTimeout(renderForm, 500);
```
- 500ms delay before form appears
- Shows loading spinner during delay
- Agent must wait for content

### Dynamic Element Creation
```javascript
const input = document.createElement('input');
input.type = field.type;
input.id = field.name;
input.name = field.name;
if (field.required) input.required = true;
formGroup.appendChild(input);
```

### Loading State
- Initial: Shows spinner and "Loading application form..."
- After 500ms: Hides loading, shows populated form
- Loading element uses CSS `display: none` when hidden

## Agent Challenges

### 1. Timing
The agent cannot interact with the form immediately:
- Page load completes but form doesn't exist
- 500ms later, form is rendered
- Agent must detect when form becomes available

### 2. DOM Observation
The agent should either:
- Poll for form element existence
- Use MutationObserver to detect DOM changes
- Wait a fixed delay before scanning

### 3. No Static Selectors
```html
<!-- This is ALL that exists initially -->
<div id="form-container"></div>
```
- Cannot query selectors that don't exist
- Must re-scan after JavaScript executes

## Testing Scenarios

### Scenario 1: Wait for Form
1. Agent loads page
2. Agent detects loading spinner OR empty form container
3. Agent waits/polls until form appears
4. Agent fills form

### Scenario 2: Immediate Attempt (Should Fail)
1. Agent loads page
2. Agent immediately tries to find form fields
3. No fields found → Agent should retry after delay

### Scenario 3: Checkbox/Radio Handling
1. Agent must handle checkbox group (multiple checkboxes same name)
2. Agent must select one radio option (exclusive)
3. Both render differently than standard inputs

## Comparison with Other Forms

| Aspect | Static Forms | Dynamic JS Form |
|--------|--------------|-----------------|
| Initial HTML | Complete form | Empty container |
| Field Detection | Immediate | After JS executes |
| Load Time | Instant | 500ms+ delay |
| Complexity | Low | High |
| Real World | Legacy sites | Modern SPAs |

## Real-World Parallels

This pattern is common in:
- React/Vue/Angular applications
- Single Page Applications (SPAs)
- Sites using Workday, Greenhouse, Lever
- Any form loaded via API/AJAX

## Limitations

- Fixed 500ms delay (real apps vary)
- No actual API call (simulated with setTimeout)
- No loading errors (always succeeds)
- No progressive rendering (all-at-once)
- Form config is static (not fetched from server)

## Extending This Test

To make it more challenging:
1. Increase delay to 2-3 seconds
2. Add random delay variation
3. Render sections progressively
4. Add error states (form fails to load)
5. Use actual API fetch with mock server
