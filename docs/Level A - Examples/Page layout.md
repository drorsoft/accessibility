---
layout: default
title: Page layout
parent: Level A - Examples
nav_order: 3
---

# Correct Page layout 

Use the hTML tags like `header`, `footer` and `nav`:

 
❌ **Don’t** use tables or only `div` elements to build the layout:
```html

<tr id="heading">
<!-- page header row -->
    <td colspan="6">
        <h1>Header</h1>
    </td>
</tr>
<!-- nav menu row -->
<tr id="nav">
    <td width="200">
        <a href="#" align="center">Home</a>
    </td>
</tr>
```  
✅ **Do** use tags with navigational meaning like `header` and `nav`:
```html
<header>
  <h1>Header</h1>
</header>
<nav>
  <!-- main navigation in here -->
</nav>
```
 