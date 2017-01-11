---
title: Calling Google Closure programmatically
tags: java, google closure, api
description: Call the Closure API
---

This is a pretty neat post, I'll show you how to programmatically
call [Google Closure](https://github.com/google/closure-library) in
Java. Closure is an amazing project that actually optimizes
`JavaScript` rather than just minifies it. (This is pretty much one of
the few times you'll see me write Java, I think the language sucks).

Its mostly a 2017 update of
this
[post from 2009](http://blog.bolinfest.com/2009/11/calling-closure-compiler-from-java.html)

Here's the Java

```java
package com.hyegar.closure;

import java.io.IOException;
import java.util.List;
import java.util.ArrayList;

import com.google.javascript.jscomp.CompilationLevel;
import com.google.javascript.jscomp.Compiler;
import com.google.javascript.jscomp.CompilerOptions;
import com.google.javascript.jscomp.SourceFile;
import com.google.javascript.jscomp.CommandLineRunner;
import com.google.javascript.jscomp.CompilerOptions.LanguageMode;

/**
 * An example of how to call the Closure Compiler programmatically, 
 * this is LICENSED AS GPL-3.0.
 *
 * @author edgar.factorial@gmail.com (Edgar Aroutiounian)
 */

public class CallCompiler {

    /**
     * @param code JavaScript source code to compile.
     * @return The compiled version of the code.
     */
    public static String compile(String code) {
	Compiler compiler = new Compiler();

	CompilerOptions options = new CompilerOptions();

	// See :
	// closure-compiler/src/com/google/javascript/jscomp/CompilerOptions.java
	// lines 2864-2896
	options.setLanguageIn(LanguageMode.ECMASCRIPT_2015);
	options.setLanguageOut(LanguageMode.ECMASCRIPT5_STRICT);

	CompilationLevel
	    .ADVANCED_OPTIMIZATIONS
	    .setOptionsForCompilationLevel(options);

	List<SourceFile> list = null;

	try {
	    list =
		CommandLineRunner
		.getBuiltinExterns(CompilerOptions.Environment.BROWSER);
	} catch (IOException e) {
	    System.out.println("Exception raised");
	}

	list.add(SourceFile.fromCode("input.js", code));
	compiler.compile(new ArrayList<SourceFile>(), list, options);
	return compiler.toSource();
    }

    public static void main(String[] args) {
      String compiled_code = compile("var a = 1 + 2; console.log(a)");
      System.out.println(compiled_code);
    }

}
```

You'll need to have the Closure Compiler installed, on OS X this is
easy with `brew install closure-compiler`

Then in a new directory that you can use for this build, you can copy
the `jar` from `/usr/local/Cellar/closure-compiler/20161201/libexec`
to where the root is of your directory, let's call that directory
`build`. 

Here's a Makefile which will build, run the code.

```makefile
blog_code:
	javac -Xlint:deprecation -cp ./closure-compiler-v20161201.jar:./ com/hyegar/closure/CallCompiler.java
	java -cp ./closure-compiler-v20161201.jar:./ com.hyegar.closure.CallCompiler	
```
