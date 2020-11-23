---
layout: inner
position: right
title: An Intoroduction to GraalVM
date: 2020-06-14 14:15:00
categories: GraalVM
tags: Software Java GraalVM
featured_image: '/img/posts/graalvm-Java.png'
lead_text: 'GraalVm is a polyglot VM engine built using Java. It provides an
interoperability runtime environment with JVM languages; guest languages
like WebAssembly, Javascript, Ruby, Python, and R; LLVM supported
languages like C, C++, and Rust. And on top of interoperability, it
provides a high-performance platform for execution. GraalVM has several
patented Just In Time performance optimization capabilities.'
project_link: 'http://steephengeorge.github.io/graalvm/2020/06/14/introduction_graalVm.html'
---
# An Introduction to GraalVM

GraalVm is a polyglot &#39;VM&#39; engine built using Java. It provides an interoperability runtime environment with JVM languages; guest languages like WebAssembly, Javascript, Ruby, Python, and R; LLVM supported languages like C, C++, and Rust. And on top of interoperability, it provides a high-performance platform for execution. GraalVM has several patented Just In Time performance optimization capabilities.

## Runtime Environments

GraalVM provides multiple runtime options:

- Java Hotspot VM with the GraalVM compiler enabled
- js runtime environment
- LLVM runtime environment

## GraalVM Architecture

GraalVM provides a Language implementation framework known as Tuffle, and it is an open-source library. It provides a mechanism to create language interpreters for GraalVM by generating a self-modifying Abstract Syntax Tree(AST) for guest languages. Hot paths in ASTs are optimized by GraalVM during the compilation process using JIT.

In the case of LLVM supported languages, they are usually statically compiled. GraalVM provides a tool `lli` that first interprets the program and later compiles it with GraalVM to optimize the hot path.

Let&#39;s make our hands dirty now.

GraalVm documentation provides detailed steps to set up with IntelliJ IDE. If you prefer to use IntelliJ use this [guide](https://www.graalvm.org/guides/#run-java-applications-on-graalvm-from-an-ide).

## Set up GralVM with Eclipse

1. Download the GraalVM community edition package from the [Github](https://github.com/graalvm/graalvm-ce-builds/releases/tag/vm-20.3.0)repository depends on your operating system.
2. Unzip and untar to your local directory. Ex: C:/Tools/graalVm
3. Set environment variable JAVA\_HOME and append the JAVA\_HOME to the PATH variable. Ex: JAVA\_HOME=C:/Tools/graalVm/bin and PATH= $PATH:$JAVA\_HOME
4. Create a new java project in Eclipse
5. Right-click on the project and go to properties
6. From the left pan, double click &#39;Build path&#39;
7. From the right pan, select the tab &#39;Libraries&#39;
8. Now you will see the &#39;JRE System Library&#39; listed below. Select it.
9. On the right pan, press the button &#39;Edit&#39;. It will pop up next window.
10. Press the button &#39;Installed JREs&#39;. It will pop up next window
11. Press the button &#39;Add&#39; and from the pop-up window select the &#39;Standard VM&#39;
12. In the next pop-up window, press the button &#39;Directory&#39; to browse and choose the GraalVM home directory. Ex: C:\Tools\graalvm
13. Press button Finish
14. From the &#39;Installed JREs&#39; window, choose the newly added GraalVM as our JRE for this project and press the button &#39;Apply and Close&#39;.
15. In the next pending opened window press the buttons Finish
16. Hope now you are back to properties windows for your project. Select Java compiler from the left pane.
17. On the right pan, against &#39;the compiler compliance level&#39; chooses the matching Java version from the drop-down box.
18. Now you are all set, press the button &#39;Apply and Close&#39;

Now we can write our first java program.
