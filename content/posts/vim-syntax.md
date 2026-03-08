---
author: "Phong Nguyen"
title: "Vim Syntax"
date: "2026-03-08"
description: "Basic Vim syntax."
tags: ["vim"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 0    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---

## 1. Modes

| Key | Mode | Description |
|-----|------|-------------|
| `ESC` | Normal Mode | Default mode. Used for navigation and simple editing. |
| `i` | Insert Mode | Insert and edit text. |
| `:` | Command Mode | Run commands like save, quit, search, etc. |

## 2. Basic Commands

- **Open File**

    ``` bash
    vi <file>
    ```

- **Quit**
    ``` bash
    :q!
    ```

- **Insert Text**
    ``` bash
    i
    ```

- **Save File**
    ``` bash
    :w
    ```

- **Save and Quit**
    ``` bash
    :wq
    ```

## 3. Navigation

- **Enable Line Numbers**
    ``` bash
    :set number
    ```

- **Jump to Line**
    ``` bash
    :<line_number>
    ```

Example:

    :8

- **Jump to End of File**
    ``` bash
    ESC
    :$
    ```

## 4. Editing

- **Delete Line**
    ``` bash
    dd
    ```

- **Undo**
    ``` bash
    u
    ```

- **Create New Line**
    ``` bash
    o
    ```

- **Paste**
    ``` bash
    p
    ```

## 5. Selection

- **Select Text**
    ``` bash
    v
    ```

Then move cursor.

- **Indent / Unindent**
    ``` bash
    >
    <
    ```

- **Copy (Yank)**
    ``` bash
    y
    ```

## 6. Search

- **Search Text**
    ``` bash
    :/<keyword>
    ```

Example:

    :/main

- **Next Result**
    ``` bash
    n
    ```

## 7. Split Window

- **Split Screen**
    ``` bash
    :split <file>
    ```

- **Switch Between Windows**
    ``` bash
    Ctrl + ww
    ```

## 8. Practice

- **Practice 1 --- Basic Editing**

1.  Create a file:

``` bash
vi test.txt
```

2.  Enter Insert mode:
``` bash
    i
```
3.  Type:

``` bash
    Hello Vim
    Learning Vim is fun
```

4.  Save and quit:

``` bash
    ESC
    :wq
```

------------------------------------------------------------------------

- **Practice 2 --- Navigation**

1.  Open file:

``` bash
vi test.txt
```

2.  Enable line numbers:

``` bash
    :set number
```

3.  Jump to line 2:

``` bash
    :2
```

------------------------------------------------------------------------

- **Practice 3 --- Editing**

Delete a line:

    dd

Undo:

    u

Add a new line below:

    o

------------------------------------------------------------------------

- **Practice 4 --- Search**

Search for a word:

    :/Hello

Next match:

    n

------------------------------------------------------------------------

- **Practice 5 --- Copy & Paste**

Enter visual mode:

    v

Copy:

    y

Paste:

    p



