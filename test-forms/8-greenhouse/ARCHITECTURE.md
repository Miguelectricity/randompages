# Greenhouse Style Form Architecture

## Overview

This form replicates the **challenging patterns** found in Greenhouse ATS job application forms that cause dropdown option extraction failures. It is specifically designed to test and improve the `dropdown_extractor.py` module.

## Purpose

Test the ghostwriter agent's ability to handle:
- **Custom JavaScript dropdown components** (not native `<select>`)
- **Portal-rendered options** (options in a separate DOM location from trigger)
- **Asynchronously loaded options** (150ms simulated delay)
- **No ARIA roles** (no `role="listbox"` or `role="option"`)
- **CSS-in-JS class names** (non-semantic class names)
- **`data-value` attributes** (instead of native `<option value="">`)

## Source Reference

Based on extraction failures from:
- https://job-boards.greenhouse.io/industrialelectricmanufacturing/jobs/4113160009
- Jira ticket: DROID-10752

## Structure

```
8-greenhouse/
├── index.html        # Main application page with custom dropdowns
├── confirmation.html # Success page after submission
└── ARCHITECTURE.md   # This file
```

## Why This Form Is Challenging

### The Problem

The current `dropdown_extractor.py` only handles these patterns:

```javascript
// Pattern 1: Native <select>
if (element.tagName === 'SELECT') {
    return Array.from(element.options).map(...)
}

// Pattern 2: ARIA listbox with [role="option"]
const listbox = findListbox();
if (listbox) {
    return Array.from(listbox.querySelectorAll('[role="option"]')).map(...)
}
```

Greenhouse uses **neither** of these patterns.

### What Greenhouse Actually Uses

```html
<!-- Trigger element (NOT a <select>) -->
<div class="gh-select" data-name="age_18">
  <input type="hidden" name="age_18" class="gh-select__input">
  <div class="gh-select__control">
    <span class="gh-select__value">Select...</span>
    <span class="gh-select__indicator">▼</span>
  </div>
</div>

<!-- Options rendered in PORTAL (separate DOM location) -->
<div id="gh-select-portal">
  <div class="gh-select__menu-portal">
    <div class="gh-select__menu">
      <div class="gh-select__menu-list">
        <!-- NO role="option", uses data-value instead -->
        <div class="gh-select__option" data-value="yes">Yes</div>
        <div class="gh-select__option" data-value="no">No</div>
      </div>
    </div>
  </div>
</div>
```

### Key Challenges Replicated

| Challenge | How It's Replicated | Why It Breaks Extraction |
|-----------|---------------------|-------------------------|
| No native `<select>` | Uses `<div>` with custom classes | `element.tagName === 'SELECT'` fails |
| Portal rendering | Options in `#gh-select-portal` at body level | Can't find options near trigger element |
| Async loading | 150ms delay before options appear | MutationObserver may miss if timing wrong |
| No ARIA roles | No `role="listbox"` or `role="option"` | `[role="option"]` selector fails |
| CSS-in-JS classes | `.gh-select__option` (non-semantic) | Can't rely on semantic class names |
| data-value attrs | `data-value="yes"` instead of `value` | Need to read `data-value` not `value` |

## Form Fields

### Standard Fields (working)

| Field | Type | Required |
|-------|------|----------|
| First Name | text | Yes |
| Last Name | text | Yes |
| Preferred Name | text | No |
| Email | email | Yes |
| Phone | tel | Yes |
| Resume/CV | file | Yes |
| Cover Letter | file | No |
| LinkedIn | url | No |
| Website | url | No |
| Desired Salary | text | Yes |
| Why Strong Fit | textarea | No |
| Additional Info | textarea | No |

### Custom Dropdown Fields (challenging)

| Field | Options | Required |
|-------|---------|----------|
| Phone Country | 160+ countries | Yes |
| Age 18+ | Yes, No | Yes |
| Previous Employee | Yes, No | Yes |
| Work Authorization | Yes, No | Yes |
| Sponsorship | Yes, No | Yes |
| Essential Functions | Yes, No | Yes |

## Technical Implementation

### Custom Dropdown Component

```javascript
// Options data (simulates API response)
const DROPDOWN_OPTIONS = {
  'age_18': [
    { value: 'yes', label: 'Yes' },
    { value: 'no', label: 'No' }
  ],
  // ...
};

// On click, create portal menu with delay
function openDropdown(selectEl, fieldName, hiddenInput, control) {
  // Create menu in portal (KEY: separate DOM location)
  const menu = document.createElement('div');
  menu.className = 'gh-select__menu-portal';
  portal.appendChild(menu);

  // SIMULATE ASYNC LOADING - 150ms delay
  setTimeout(() => {
    const options = DROPDOWN_OPTIONS[fieldName];
    menuList.innerHTML = options.map(opt => `
      <div class="gh-select__option" data-value="${opt.value}">
        ${opt.label}
      </div>
    `).join('');
  }, 150);
}
```

### DOM Structure When Open

```
<body>
  ...
  <div class="application-panel">
    <div class="gh-select" data-name="age_18">
      <!-- Trigger here, but NO options -->
    </div>
  </div>
  ...
  <!-- Options are HERE, far from trigger -->
  <div id="gh-select-portal">
    <div class="gh-select__menu-portal">
      <div class="gh-select__option" data-value="yes">Yes</div>
      <div class="gh-select__option" data-value="no">No</div>
    </div>
  </div>
</body>
```

## Proposed Solution Patterns

To extract options from this form, the extractor needs to:

### 1. Detect Custom Dropdown Triggers

```javascript
// Look for elements that behave like dropdowns but aren't <select>
const customTriggers = document.querySelectorAll(
  '[class*="select__control"], ' +
  '[class*="-control"], ' +
  '[data-testid*="select"]'
);
```

### 2. Find Portal-Rendered Options

```javascript
// After clicking trigger, scan document for new option containers
const findPortalOptions = () => {
  // Look for common portal patterns
  const portals = document.querySelectorAll(
    '[class*="menu-portal"], ' +
    '[class*="__menu"], ' +
    '[class*="-menu"]:not([aria-hidden="true"])'
  );

  // Find option-like elements within
  for (const portal of portals) {
    const options = portal.querySelectorAll(
      '[class*="option"], ' +
      '[class*="-option"], ' +
      '[data-value]'
    );
    if (options.length > 0) return options;
  }
};
```

### 3. Extract from data-value Attributes

```javascript
// Get value from data-value, not native value attribute
options.map(opt => ({
  value: opt.dataset.value || opt.getAttribute('data-value') || opt.textContent,
  label: opt.textContent.trim()
}));
```

### 4. Handle Async Loading

```javascript
// Wait for options to appear in DOM after click
const waitForOptions = async (maxWait = 2000) => {
  const start = Date.now();
  while (Date.now() - start < maxWait) {
    const options = findPortalOptions();
    if (options?.length > 0) return options;
    await new Promise(r => setTimeout(r, 50));
  }
  return [];
};
```

## Testing Scenarios

1. **Option Detection**: Click each dropdown and verify options are found
2. **Portal Scanning**: Verify extractor looks in `#gh-select-portal`
3. **Async Timing**: Options load after 150ms - verify stabilization works
4. **Data Attribute Reading**: Verify `data-value` is extracted correctly
5. **No ARIA Fallback**: Verify extraction works without ARIA roles
6. **Large Dropdown**: Phone country has 160+ options - test paginated/scroll handling

## Comparison with Current Extractor

| Feature | Current Extractor | Greenhouse Form |
|---------|------------------|-----------------|
| Element type | `<select>` | `<div class="gh-select">` |
| Options location | Inside element | In `#gh-select-portal` |
| Option selector | `[role="option"]` | `.gh-select__option` |
| Value attribute | `value` | `data-value` |
| ARIA linking | `aria-controls` | None |
| Loading | Immediate | 150ms delay |

## Success Criteria

The `dropdown_extractor.py` improvements should:

1. Detect `.gh-select` as a dropdown field
2. Click to open and wait for portal to populate
3. Find options in `#gh-select-portal` (or similar)
4. Extract `data-value` and text content
5. Return all 6 dropdown fields with correct options
6. Handle the 150ms async loading delay
7. Handle 160+ options in phone country dropdown

## Files to Modify

1. `ghostwriterdaemon/workday_automation/dropdown_extractor.py`:
   - Add portal-based option scanning
   - Add CSS class pattern matching
   - Add `data-value` attribute extraction

2. `ghostwriterdaemon/form_field_extractor.py`:
   - Detect custom dropdown triggers
   - Add heuristics for non-native dropdowns
