# Conditional Fields Architecture

## Overview
A form where certain fields appear or hide based on the user's responses to other fields. This pattern is extremely common in job applications.

## Purpose
Test the ghostwriter agent's ability to:
- Detect fields that appear after selecting certain options
- Fill dynamically revealed fields
- Handle form state changes triggered by user input
- Recognize which fields are required based on context

## Structure

```
4-conditional/
├── index.html        # Form with conditional logic
├── confirmation.html # Success page
└── ARCHITECTURE.md   # This file
```

## Conditional Field Groups

### 1. Visa/Work Authorization
**Trigger**: `work_authorization` = "visa" or "no"

| Revealed Field | Type | Required When Visible |
|----------------|------|----------------------|
| Current Visa Type | select | Yes |
| Visa Expiration Date | date | Yes |
| Sponsorship Needed | select | No |

### 2. Senior Experience
**Trigger**: `years_experience` = "5-8" or "8+"

| Revealed Field | Type | Required When Visible |
|----------------|------|----------------------|
| Team Size Led | select | Yes |
| High-Scale Architecture | select | No |
| Mentorship Experience | textarea | No |

### 3. Relocation
**Trigger**: `willing_relocate` = "yes"

| Revealed Field | Type | Required When Visible |
|----------------|------|----------------------|
| Preferred Locations | multi-select | No |
| Relocation Timeline | select | No |
| Relocation Assistance | select | No |

## Field Inventory

### Always Visible (10 fields)
| Field | Type | Required |
|-------|------|----------|
| Full Name | text | Yes |
| Email | email | Yes |
| Phone | tel | Yes |
| Work Authorization | select | Yes |
| Years Experience | select | Yes |
| Current Location | text | Yes |
| Willing to Relocate | select | Yes |
| Resume | file | Yes |
| GitHub Profile | url | No |
| Why Interested | textarea | No |

### Conditionally Visible (9 fields)
Visa fields (3), Senior fields (3), Relocation fields (3)

**Total Possible Fields: 19**

## Technical Implementation

### CSS Visibility
```css
.hidden {
  display: none;
}
```

### JavaScript Toggle Functions
```javascript
function toggleVisaFields() {
  const select = document.getElementById('work_authorization');
  const visaFields = document.getElementById('visa-fields');

  if (select.value === 'visa' || select.value === 'no') {
    visaFields.classList.remove('hidden');
    // Also make fields required
  } else {
    visaFields.classList.add('hidden');
    // Remove required attribute
  }
}
```

### Event Binding
```html
<select onchange="toggleVisaFields()">
```
- Uses inline `onchange` handlers for simplicity
- Each trigger field has its own toggle function
- Functions immediately evaluate and show/hide sections

### Dynamic Required Attributes
When visa fields become visible, their required status is set:
```javascript
visaFields.querySelectorAll('select, input').forEach(el => {
  if (el.closest('.form-group').querySelector('.required')) {
    el.required = true;
  }
});
```

## Agent Testing Scenarios

### Scenario 1: US Citizen (Minimal)
1. Select "US Citizen" for work authorization → No visa fields
2. Select "0-2 years" experience → No senior fields
3. Select "No" for relocation → No relocation fields
4. **Result**: Only 10 fields visible

### Scenario 2: Visa Holder Relocating
1. Select "Work Visa" → 3 visa fields appear
2. Select "8+ years" → 3 senior fields appear
3. Select "Yes" for relocation → 3 relocation fields appear
4. **Result**: All 19 fields visible

### Scenario 3: Mid-Level Remote
1. Select "Green Card" → No visa fields
2. Select "3-5 years" → No senior fields
3. Select "No, remote only" → No relocation fields
4. **Result**: Only 10 fields visible

## Detection Challenges

| Challenge | Description |
|-----------|-------------|
| Hidden vs Absent | Fields exist in DOM but hidden via CSS |
| Dynamic Required | Required attribute changes with visibility |
| Trigger Order | Changing trigger resets conditional fields |
| Multi-Trigger | Some fields may depend on multiple conditions |

## How Agent Should Handle

1. **Fill visible fields first** - Handle the always-visible fields
2. **Check for conditionals** - After each dropdown, check if new fields appeared
3. **Re-scan form** - After triggering conditionals, identify new fields
4. **Fill revealed fields** - Handle the newly visible fields
5. **Submit** - Ensure all visible required fields are filled

## DOM Structure

```html
<!-- Trigger -->
<select id="work_authorization" onchange="toggleVisaFields()">

<!-- Conditional Section -->
<div id="visa-fields" class="hidden">
  <div class="form-group">
    <label class="required">Current Visa Type</label>
    <select id="visa_type">...</select>
  </div>
  ...
</div>
```

The agent needs to:
1. Recognize the container's visibility state (`class="hidden"`)
2. Understand that selecting certain options removes the `hidden` class
3. Wait for DOM changes after selection events

## Limitations

- No nested conditionals (conditional within conditional)
- No conditionals based on text input (only selects)
- No API-driven conditionals (all client-side)
- Visibility toggle is immediate (no animations)
