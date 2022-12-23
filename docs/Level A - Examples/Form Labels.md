---
layout: default
title: Form labels
parent: Level A - Examples
nav_order: 6
---

# Form labels

Use form labels for screen readers:

 
❌ **Don’t** use simple text instead of label:
```html
<div>
  <p>Enter your name</p>
  <input type="text" id="name" name="name" />
</div>
```  
✅ **Do** add `label` with `for` attribute:
```html
<div>
  <label for="name">Enter your name</label>
  <input type="text" id="name" name="name" />
</div>
```
 