# Tutorials
## Welcome to the GStreamer Tutorials!
The following sections introduce a series fo tutorials designed to help you learn how to use GStreamer, the multi-platform, modular, open-source, media streaming framework.  
## Prerequisites (DK tien quyet)
Before follwing these tutorials, you need to set up your development environment according to your platform. If you have not done so yet, get to the [installing GStreamer](https://gstreamer.freedesktop.org/documentation/installing/index.html?gi-language=c) and come back here afterwards (sau).  
The tutorials are currently written only in the C programming language, so you need to comfortable with it. Even though C is not an Object-Oriened (OO) language per se, the GStreamer framework uses `GObject`s, so some knowledge of OO concepts will come in handy (tien dung). Knowledge of the `GObject` and `GLib` libraries is not mandatory (bat buoc), but will make the trop easier.  
## Source code
Every tutorial represents a self-contained (khep kin) project, will full source code in C (and eventually (cuoi cung) in other languages too. Source code snippets (doan trich) are introduced alongside (cung voi) the text, and the full code (with any other required files like makefiles or project files) is distributed with GStreamer, as explained in the installation instructions.  
## A short note on GObject and GLib
GStreamer is built on top of the `GObject` (for object orientation) and `GLib` (for common algorithms0 libraries, which means that every now and then you will have to call functions of these libraties. Even though the tutorials will make sure the deep knowledge of these libraries is not required, familiarity with them will certainly ease the process of learning GStreames.

You can always tell which library you are calling because all GStreamer function, structures and types have the `gst_` prefix (loi noi dau), whereas (trong khi) Glib and GObject use `g_`.
## Sources of documentation
You have the `GObject` and `GLib` reference guides, and, of course the upstream [GStreamer documention](https://gstreamer.freedesktop.org/documentation/?gi-language=c).
## Structure (ket cau)
The tutorials are organized in sections, revoling (quay vong) about a common theme (chu de):
* [Basic tutorial:]() Descibe general topics repuired to understand the rest of tutorial in GStreamer.
* [Playback tutorial:]() Explain everything you need to know to produce a media playback application using GStreamer.
* [Table of Concept](https://gstreamer.freedesktop.org/documentation/tutorials/table-of-concepts.html?gi-language=c)
## Sample media 
The audio and video clips used throughout these tutorials are all publicly available and the copyright remains (còn lại) with their respective (tương ứng) authors. In some cases they have been re-encoded for demonstration purposes (muc dich).

