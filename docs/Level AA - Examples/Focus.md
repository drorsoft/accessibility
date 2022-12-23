---
layout: default
title: Focus
parent: Level AA - Examples
nav_order: 6
---


# Focus

## The rule

Pointer hover or keyboard focus triggers additional content to become visible and then hidden. The user can see the point of focus on links, buttons and other interactive elements.

## Example
 
In this example, mouse and keyboard focus indicators have been applied to the link elements. CSS has been used to apply a background color when the link elements receive focus.

Here is the content to be displayed:

```HTML
<ul id="mainnav">
  <li class="page_item">Home</li>
  <li class="page_item"><a href="/services">Services</a></li>
</ul>
```

Here is the CSS that changes the background color for the above elements when they receive mouse or keyboard focus:

```css
#mainnav a:hover, #mainnav a:active, #mainnav a:focus {
  background-color: #DCFFFF;
  color:#000066;
}
```