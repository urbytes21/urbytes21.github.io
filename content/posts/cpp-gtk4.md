---
author: "Phong Nguyen"
title: "GTK4"
date: "2025-08-21"
description: "Gtk4."
tags: ["cpp"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 1000   # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---
Gtk4 with Cpp.

**References:**
[Gtk](https://www.gtk.org/docs/)<br>
## 1. Introduction
- Gtk4 is a widget toolkit for creating graphical user interface (UI). It works on many UNIX-like platform, Windows and Macos.

## 2. Setup the environment
```xml
Note:
    - This is the guides for Windows machine using Windows Subsystem for Linux (WSL).
    - [W]: The action will be performed on Windows Machine.
    - [U]: The action will be performed on WSL (in my case, it named "Ubuntu") on the host Windows Machine.
    - (Guides: ...): The reference link that I refer the step.
    - (Not try yet): This is referred from the guides but I was not in that case, so that I could not try.

-------------------------------
A. Installing from packages
-------------------------------
1. [L] Install required packages.
    1.1. [L] Debian/Ubuntu: sudo apt install libgtk-4-1 libgtk-4-dev gtk-4-examples

2. [L] Verify installation.
    2.1. [L] gtk version - Run: pkg-config --modversion gtk4
    2.2. [L] examples - Run: gtk4-demo

-------------------------------
B. (Optional) Build from Source
-------------------------------
1. [L] Install build tools.
    1.1. [L] sudo apt install build-essential meson ninja-build \
               libglib2.0-dev libpango1.0-dev libgdk-pixbuf-2.0-dev \
               libatk1.0-dev libepoxy-dev libgirepository1.0-dev

2. [L] Clone GTK source.
    2.1. [L] git clone https://gitlab.gnome.org/GNOME/gtk.git
    2.2. [L] cd gtk

3. [L] Build with Meson + Ninja.
    3.1. [L] meson setup builddir
    3.2. [L] ninja -C builddir

4. [L] (Optional) Install system-wide.
    4.1. [L] sudo ninja -C builddir install
```

## 3. HelloWorld

-  **Create a new file** named  `hello-world-gtk4.c` with following content:
```Cpp
#include <gtk/gtk.h>

static void
print_hello (GtkWidget *widget,
             gpointer   data)
{
  g_print ("Hello World\n");
}

static void
activate (GtkApplication *app,
          gpointer        user_data)
{
  GtkWidget *window;
  GtkWidget *button;

  window = gtk_application_window_new (app);
  gtk_window_set_title (GTK_WINDOW (window), "Hello");
  gtk_window_set_default_size (GTK_WINDOW (window), 200, 200);

  button = gtk_button_new_with_label ("Hello World");
  g_signal_connect (button, "clicked", G_CALLBACK (print_hello), NULL);
  gtk_window_set_child (GTK_WINDOW (window), button);

  gtk_window_present (GTK_WINDOW (window));
}

int
main (int    argc,
      char **argv)
{
  GtkApplication *app;
  int status;

  app = gtk_application_new ("org.gtk.example", G_APPLICATION_DEFAULT_FLAGS);
  g_signal_connect (app, "activate", G_CALLBACK (activate), NULL);
  status = g_application_run (G_APPLICATION (app), argc, argv);
  g_object_unref (app);

  return status;
}

```

*- Hints:*
```bash
mkdir hello-world-project
cd hello-world-project
touch hello-world-gtk4.c
vi hello-world-gtk4.c
...
:wq
```

- **Compile** the program using gcc: `gcc [options] source.c -o program [libraries]`
```bash
gcc $(pkg-config --cflags gtk4) hello-world-gtk4.c -o hell-world-gtk4 $(pkg-config --libs gtk4)

```

- **Run** the program
```bash
./hello-world-gtk4
```


