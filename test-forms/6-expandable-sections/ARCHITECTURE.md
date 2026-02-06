# Expandable Sections Architecture

## Overview
A job application form with expandable, repeatable sections for Work Experience and Education. Mirrors the pattern used by Breezy HR and similar ATS platforms where users can add multiple entries dynamically, each with its own set of sub-fields.

## Purpose
Test the ghostwriter agent's ability to:
- Interact with dynamically added form sections
- Handle multiple instances of the same field group
- Manage collapsible/expandable UI elements
- Fill forms where the number of entries is variable
- Handle "currently working/enrolled" toggles that disable end date fields

## Structure

```
6-expandable-sections/
├── index.html        # Main form with expandable sections
├── confirmation.html # Success page
└── ARCHITECTURE.md   # This file
```

## How It Works

### Initial State
The form loads with:
- Personal information fields (static)
- One work experience entry (expandable)
- One education entry (expandable)
- Additional information fields (static)

### Dynamic Behavior
Users can:
1. **Add entries**: Click "Add Work Experience" or "Add Education" buttons
2. **Remove entries**: Click "Remove" button on any entry (except the first)
3. **Collapse/Expand**: Click entry headers to show/hide fields
4. **Toggle end dates**: Check "I currently work here" / "I am currently enrolled" to disable end date

## Field Inventory

### Personal Information (5 fields, static)
| Field | Type | Required |
|-------|------|----------|
| Full Name | text | Yes |
| Email Address | email | Yes |
| Phone Number | tel | Yes |
| Current Location | text | No |
| Resume | file | Yes |

### Work Experience Entry (8 fields per entry, dynamic)
| Field | Type | Required |
|-------|------|----------|
| Job Title | text | Yes |
| Company | text | Yes |
| Location | text | No |
| Employment Type | select | No |
| Start Date | month | Yes |
| End Date | month | No (disabled if current) |
| I currently work here | checkbox | No |
| Description | textarea | No |

### Education Entry (8 fields per entry, dynamic)
| Field | Type | Required |
|-------|------|----------|
| School / University | text | Yes |
| Degree | select | Yes |
| Field of Study | text | Yes |
| GPA | text | No |
| Start Date | month | No |
| End Date | month | No (disabled if current) |
| I am currently enrolled | checkbox | No |
| Activities / Achievements | textarea | No |

### Additional Information (4 fields, static)
| Field | Type | Required |
|-------|------|----------|
| Cover Letter | textarea | No |
| LinkedIn Profile | url | No |
| Portfolio / Website | url | No |
| How did you hear about us | select | No |

## Field Naming Convention

Dynamic fields use indexed naming:
- `experience_1_title`, `experience_2_title`, etc.
- `education_1_school`, `education_2_school`, etc.

This pattern is critical for:
1. Form data serialization
2. Agent field identification
3. Maintaining uniqueness across entries

## Technical Implementation

### Entry Template Structure
```html
<div class="entry-container" id="experience_1_container">
  <div class="entry-header" onclick="toggleEntry('experience_1')">
    <h3><span class="entry-number">1</span> Work Experience 1</h3>
    <div class="entry-actions">
      <button class="btn-collapse">−</button>
      <button class="btn-remove">Remove</button>
    </div>
  </div>
  <div class="entry-fields" id="experience_1_fields">
    <!-- Fields rendered here -->
  </div>
</div>
```

### JavaScript Functions

```javascript
addExperienceEntry()  // Adds new experience entry
addEducationEntry()   // Adds new education entry
toggleEntry(id)       // Collapse/expand entry fields
toggleEndDate(id)     // Enable/disable end date based on checkbox
removeEntry(id, type) // Remove entry and renumber remaining
updateEntryNumbers()  // Renumber entries after removal
```

### CSS Classes

| Class | Purpose |
|-------|---------|
| `.entry-container` | Wrapper for entire entry |
| `.entry-header` | Clickable header for collapse |
| `.entry-fields` | Container for field grid |
| `.entry-fields.collapsed` | Hidden state (display: none) |
| `.entry-number` | Circular number badge |
| `.btn-add` | Green "Add" button |
| `.btn-remove` | Red "Remove" button |
| `.end-date-field.disabled` | Grayed out end date when current |

## Agent Challenges

### 1. Dynamic Field Discovery
Fields don't exist until entries are added. Agent must:
- Detect "Add" buttons
- Click to create entries
- Re-scan DOM for new fields

### 2. Field Indexing
Multiple entries have similar fields with indexed names:
```
experience_1_title → "Software Engineer"
experience_2_title → "Junior Developer"
```
Agent must track which entry it's filling.

### 3. Conditional Fields
End date field becomes disabled when "currently working" is checked:
```javascript
if (checkbox.checked) {
  endDateInput.disabled = true;
  endDateInput.value = '';
}
```

### 4. Collapsible Sections
Fields may be hidden. Agent should:
- Detect collapsed state
- Expand section before interacting
- Handle click events on headers

### 5. Entry Management
Agent may need to:
- Add additional entries for comprehensive work history
- Remove entries if data doesn't fit
- Reorder entries (not supported in this form)

## Testing Scenarios

### Scenario 1: Minimal Application
1. Fill personal info
2. Fill one experience entry
3. Fill one education entry
4. Submit

### Scenario 2: Multiple Entries
1. Fill personal info
2. Add 2-3 experience entries
3. Add 2 education entries
4. Fill all entries
5. Submit

### Scenario 3: Current Position Handling
1. Add experience entry
2. Check "I currently work here"
3. Verify end date is disabled
4. Attempt to fill end date (should fail gracefully)

### Scenario 4: Entry Removal
1. Add 3 experience entries
2. Fill all
3. Remove middle entry
4. Verify numbering updates
5. Submit

### Scenario 5: Collapsed Sections
1. Add entries
2. Collapse some sections
3. Attempt to fill collapsed fields
4. Agent should expand first

## Real-World Parallels

This pattern is common in:
- Breezy HR application forms
- Greenhouse ATS
- Workday career sections
- LinkedIn job applications
- Most modern ATS platforms

## Field Count Summary

| Section | Min Fields | Max Fields (3 entries each) |
|---------|-----------|------------------------------|
| Personal | 5 | 5 |
| Experience | 8 | 24 |
| Education | 8 | 24 |
| Additional | 4 | 4 |
| **Total** | **25** | **57** |

## Comparison with Other Test Forms

| Aspect | Single-Page | Multi-Page | Conditional | Expandable |
|--------|-------------|------------|-------------|------------|
| Dynamic Fields | No | No | CSS toggle | JS creation |
| Multiple Entries | No | No | No | Yes |
| Field Count Varies | No | No | Yes | Yes |
| DOM Manipulation | No | No | Minimal | Heavy |
| Agent Complexity | Low | Medium | Medium | High |

## Limitations

- No drag-and-drop reordering of entries
- No auto-save / draft persistence
- No field validation beyond HTML5 required
- No date range validation (end > start)
- No duplicate entry detection
- Maximum entries limited only by browser performance

## Extending This Test

To increase challenge:
1. Add minimum entry requirements per section
2. Add date range validation
3. Implement auto-collapse when adding new entry
4. Add skills section with tag-based input
5. Add references section with similar pattern
6. Implement field dependencies (e.g., degree type affects field of study options)
