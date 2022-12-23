---
layout: default
title: HTML Tags
parent: Level A - Examples
nav_order: 2
---

# HTML Tags

## Correct use of tags - 

Use the tags to the purpose they are intended:

 
❌ **Don’t** use div tags for tags that function as buttons:
```html
<div>Play video</div>
```  
✅ **Do** use buttons tags for tags that function as buttons:
```html
<button>Play video</button>
```


These may look the same (if styled the same) but: <br>
Different tags also have built-in keyboard accessibility — users can navigate between buttons using the Tab key and activate their selection using Space, Return or Enter.

## Good tag semantics

Like in the previous example, these two can be styled the same, but can help or confuse screen readers: 

❌ **Don’t** use span or div tags for headers:
```html
<span style="font-size: 3em">My heading</span>
<br /><br />
This is the first section of my document.
```
✅ **Do** use header tags for headers and p tags for paragraphs:
```html
<h1>My heading</h1>
<p>This is the first section of my document.</p>
```

