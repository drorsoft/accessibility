---
layout: default
title: Audio and Video
parent: Level AA - Examples
---

# Audio and Video

 

Use the tags to the purpose they are intended:

{: .warning }
A paragraph

> {: .wrong }
> > A paragraph
> >
> > Another paragraph
> >
> > The last paragraph

{: .right}
```html
<div>Play video</div>
```

```html
<button>Play video</button>
```

These may look the same (if styled the same) but: <br>
Different tags also have built-in keyboard accessibility — users can navigate between buttons using the Tab key and activate their selection using Space, Return or Enter.

## Good tag semantics

Like in the previous example, these two can be styled the same, but can help or confuse screen readers: 

```
<span style="font-size: 3em">My heading</span>
<br /><br />
This is the first section of my document.
```

```
<h1>My heading</h1>
<p>This is the first section of my document.</p>
```
