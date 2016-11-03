---
title: Using JavaScriptCore with examples
tags: javascriptcore, javascript
description: practical JavaScriptCore
---

## Motivation

I like mixing languages and I especially like `OCaml`,
`JavaScript`. Lately I've been writing `OCaml` bindings to
`JavaScriptCore`, JSC is what underpins Safari both iOS and desktop
variants, and ReactNative. When you're writing bindings, you kinda
need to know the `C/C++` library that you're binding to.

## Example

We're going to provide a pretty meaty example, we'll make a custom
object in C++ code, then we'll use it from a `JavaScript`
script. Imagine we want to expose a `File` like object, JS the
language itself has no notion of a File, so it will need support from
the C/C++ level; you can imagine this as the objects that say `[native
code]`. 

Here's the code, afterwards is a detailed breakdown (The `R` C++
feature confuses the syntax highlighter)

```c++
#include <string>
#include <iostream>
#include <JavaScriptCore/JavaScriptCore.h>

std::string getcwd_string(void)
{
   char buff[PATH_MAX];
   getcwd(buff, PATH_MAX);
   std::string cwd(buff);
   return cwd;
}

const char*
jsvalue_to_utf8_string(JSGlobalContextRef ctx, JSValueRef v)
{
  JSStringRef valueAsString = JSValueToStringCopy(ctx, v, NULL);
  size_t jsSize = JSStringGetMaximumUTF8CStringSize(valueAsString);
  char* jsBuffer = (char*)malloc(jsSize);
  JSStringGetUTF8CString(valueAsString, jsBuffer, jsSize);
  JSStringRelease(valueAsString);
  return jsBuffer;
}

void run_example(void)
{
  JSClassDefinition definition = kJSClassDefinitionEmpty;
  definition.className = "File";
  definition.callAsConstructor = [](auto ctx,
				    auto object,
				    auto argumentCount,
				    auto arguments[],
				    auto *exception) -> JSObjectRef {
    auto file_name =
    JSValueMakeString(ctx, JSStringCreateWithUTF8CString(getcwd_string().c_str()));
    auto prop_name = JSStringCreateWithUTF8CString("cwdName");
    auto example_obj = JSObjectMake(ctx, nullptr, nullptr);

    JSObjectSetProperty(ctx,
			example_obj,
			prop_name,
			file_name,
			kJSPropertyAttributeNone,
			nullptr);
    return example_obj;
  };

  auto ctx = JSGlobalContextCreate(nullptr);
  auto js_example_class = JSClassCreate(&definition);
  auto example_obj = JSObjectMake(ctx, js_example_class, nullptr);
  // auto called_result = JSObjectCallAsConstructor(ctx, example_obj, 0, nullptr, nullptr);
  // auto property_check =
  //   JSObjectGetProperty(ctx,
  // 			called_result,
  // 			JSStringCreateWithUTF8CString("cwdName"),
  // 			nullptr);

  // std::cout << jsvalue_to_utf8_string(ctx, property_check) << std::endl;
  std::string code_to_eval = R"(
const example_code = new File;
example_code.cwdName;
)";
  JSValueRef exn;

  auto global_object = JSContextGetGlobalObject(ctx);
  JSObjectSetProperty(ctx,
		      global_object,
		      JSStringCreateWithUTF8CString("File"),
		      example_obj,
		      kJSPropertyAttributeNone,
		      nullptr);

  auto sanity_check =
    JSEvaluateScript(ctx,
		     JSStringCreateWithUTF8CString(code_to_eval.c_str()),
		     nullptr,
		     nullptr,
		     1,
		     &exn);
  if (exn)
    std::cout << jsvalue_to_utf8_string(ctx, exn) << std::endl;

  std::cout << jsvalue_to_utf8_string(ctx, sanity_check) << std::endl;
}

int main(int argc, char **argv)
{
  run_example();
  return 0;
}
```

compile with, should work just fine on Linux as well, but using: 

`libjavascriptcoregtk-4.0` instead of the `-framework JavaScriptCore`

```shell
$ clang++ -framework JavaScriptCore -std=c++14 jsc_examples.cpp -oF 
```

## Explanations

The real meat of the code starts at `run_example`. We start by
creating a `JSClassDefinition`, this creates a template that lets us
control all the behavior of our custom object. Then we provide an
implementation for what code ought to be run when our custom object
will created with `new`, notice using `C++14`'s nice ability to use
`auto` for the parameter names, also notice that we explicitly have to
give back the return type of `JSObjectRef`, you get a bit spoiled by
the `OCaml` type system. Then we create a `JavaScript` object, set the
property `cwdName` and return that new object from the constructor
call.

Then we need to make a global context, this is like the environment
that your JavaScript runs in and we grab the global object out of it,
like `window` in the browser. (The commented out code is if you wanted
to call the constructor and get the result directly in code)

We set the class object as a property to the global, aka like doing
`window.d3 = //d3's code`. Then we evaluate a script, some fun trival
all your `<script>` tags actually have a return value, its the last
value.

Tada.

FWIW I'm doing the OCaml
bindings [here](https://github.com/fxfactorial/ocaml-javascriptcore)
github stars appreciated :)
