---
layout: default
title: WAI-ARIA
nav_order: 8
---
# WAI-ARIA

## Definition

WAI-ARIA is a set of additional HTML attributes that can be applied to elements to provide additional semantics and improve accessibility wherever it is lacking.

It is not a related to a specific level, but has the ability to add more accessability, like in cases where Javascript behavior diminishes accessability.

## Attributes
### Roles
ARIA roles provide semantic meaning to content, allowing screen readers and other tools to present and support interaction with object in a way that is consistent with user expectations of that type of object.
For example:

```html
<button role="tab" aria-selected="true" aria-controls="tabpanel-id" id="tab-id">
  Tab label
</button>
```
here the tab `role` should contain the aria-controls property identifying a corresponding tabpanel.

### Properties 

These define properties of elements, which can be used to give them extra meaning or semantics. As an example, `aria-required="true"` specifies that a form input needs to be filled in order to be valid:

```html
<div id="txtboxMultilineLabel">Enter the tags for the article</div>
<div
  role="textbox"
  id="txtBoxInput"
  contenteditable="true"
  aria-labelledby="txtboxMultilineLabel"
  aria-required="true"></div>
```

### States

Special properties that define the current conditions of elements, such as `aria-disabled="true"`, which specifies to a screen reader that a form input is currently disabled.

```html
<div id="saveChanges" tabindex="0" role="button" aria-disabled="true">Save</div>
```

 

 
 