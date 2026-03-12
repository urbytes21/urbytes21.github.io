---
author: "Phong Nguyen"
title: "Markdown Syntax Guide"
date: "2024-08-09"
description: "Basic Markdown syntax."
tags: ["markdown"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 10    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---
## Introduction

Markdown is a lightweight markup language that use to add formatting elements to plaintext documents.

## Table of contents

## 1. Heading:

To create heading, add one to six `#` symbols before your heading. 
The number of `#` will determine the size of heading.

### Syntax
```markdown
# H1
## H2
### H3
#### H4
##### H5
###### H6
```
### Preview
# H1
## H2
### H3
#### H4
##### H5
###### H6


## 2. Emphasis:

You can add emphasis by making text bold or italic.

### Syntax
```markdown
**This text is bold.**
__This text is bold.__

*This text is italic.*
_This text is italic._
```
### Preview
**This text is bold.**
__This text is bold.__

*This text is italic.*
_This text is italic._


## 3. Lists:

Markdown supports ordered or unordered lists.
- Ordered: using number
- Unordered: using `-`,`*` or `+`

### Syntax
```markdown
- item1
  - item1.1
    - item1.1.1
* item2
+ item3

1. item1
2. item2
```

### Preview
- item1
  - item1.1
    - item1.1.1
* item2
+ item3

1. item1
2. item2


## 4. Links:

To create the link, enclose the link text in brackets `[]`, and then follow it immediately with the URL in parentheses`()`.

### Syntax
```markdown 
Click this link [urlName](https://www.github.com)
```

### Preview
Click this link [urlName](https://www.github.com)


## 5. Images:

Images are similar to link but with and exclamation mark `!` in front.

### Syntax
```markdown
This is my image ![imageName](C:\Users\phong.nguyen-van\Desktop\datasets\blog\phongnv.es\resources\_gen\images\images\msg_hu1084214322766098934.png)
``` 

### Preview
This is my image ![imageName](C:\Users\phong.nguyen-van\Desktop\datasets\blog\phongnv.es\resources\_gen\images\images\msg_hu1084214322766098934.png)


## 6. Code Blocks:

For inline code, use backticks __`__.
For block code, use triple backticks __```__.

### Syntax
```markdown
`printf("This is a line code.");`
```java \n printf("This is a line code."); \n ```
```

### Preview
`printf("This is a line code.");`
```java
printf("This is a line code.");
pulic class Java(){}
```


## 7. Tables:

Tables in Markdown use the pipes `|` and dashes `-` to create a grid.

### Syntax
```markdown
| Header 1 | Header 2 |
|----------|----------|
| Row 1    | Data 1   |
| Row 2    | Data 2   |
```

### Preview
| Header 1 | Header 2 |
|----------|----------|
| Row 1    | Data 1   |
| Row 2    | Data 2   |

## 8. Blockquote:

Use the `>` symbol to create a blockquote.

### Syntax
```markdown
> This is blockquote.
> This is blockquote.
> This is blockquote.
```

### Preview
> This is blockquote.
> This is blockquote.
> This is blockquote.


## 9. Horizontal Rules:

Use `---`,`***` or `___` on a new line to create a horizontal rule.

### Syntax
```markdown
---
a
b
c
```

### Preview
---
a
b
c


## 10. Advanced Features:

### 10.1. Task Lists

#### Syntax
```markdown
- [x] Task 1
- [ ] Task 2
```

#### Preview
- [x] Task 1
- [ ] Task 2

### 10.2. Footnotes

#### Syntax
```markdown
Here is a simple footnote[^1].

[^1]: This is the footnote.
```

#### Preview
Here is a simple footnote[^1].

[^1]: This is the footnote.
