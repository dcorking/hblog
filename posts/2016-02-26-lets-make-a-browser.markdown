---
title: Let's make a webkit browser in Objective-C++
tags: darwin, objective-c, c++, webkit
description: making a simple webbrowser
---

This is a short post showing you how to write a simple `OS X` based
web browser using `Objective-C` in just 80 lines of code and with no
XCode. Its very valuable because making a Cocoa application without
XCode is sometimes kind of hard.

First let's start with the `Makefile`

```makefile
opts := -std=c++11 -fobjc-arc
fws := -framework Cocoa -framework Webkit
libs := -lc++
exec := Prog
wrapper := xcrun -r

all:
	${wrapper} clang++ ${opts} ${fws} ${libs} main.mm -o ${exec}
	./${exec}
```

We're going to use Webkit, ARC, C++11 and immediately run the program.

Now the program, I'm going to assume you are familiar with Cocoa
development, Objective-C. 

```objective-c
/* -*- objc -*- */

#import <Foundation/Foundation.h>
#import <Webkit/WebKit.h>
#import <iostream>

@interface Window_delegate : NSObject <NSWindowDelegate>
@end

@implementation Window_delegate

-(void)windowWillClose:(id)a
{
  NSLog(@"Exiting");
  exit(0);
}
@end

@interface App_delegate : NSObject <NSApplicationDelegate>
@property (weak, nonatomic) NSWindow *_window;
@end

@implementation App_delegate

-(instancetype)init:(NSWindow*)parent_window
{
  if (self = [super init]) {
    self._window = parent_window;
    return self;
  }
  return nil;
}

-(void)applicationDidFinishLaunching:(NSNotification *)a_notification
{
  WebView *wb = [[WebView alloc] initWithFrame:self._window.contentView.frame];
  [self._window.contentView addSubview:wb];
  [[wb mainFrame]
    loadRequest:[NSURLRequest
		  requestWithURL:[NSURL URLWithString:@"http://hyegar.com"]]];
}

@end

void start_program(void)
{
  NSApplication *app = [NSApplication sharedApplication];
  // Critical to have this so that you can add menus
  [NSApp setActivationPolicy:NSApplicationActivationPolicyRegular];

  NSWindow *window =
    [[NSWindow alloc]
      initWithContentRect:NSMakeRect(0, 0, 800, 600)
		styleMask: NSTitledWindowMask | NSClosableWindowMask | NSMiniaturizableWindowMask
		  backing:NSBackingStoreBuffered
		    defer:NO];

  [window center];
  window.title = @"Hello World";
  [window makeKeyAndOrderFront:window];

  Window_delegate *d = [Window_delegate new];
  window.delegate = d;

  App_delegate *application_delegate = [[App_delegate alloc] init:window];
  app.delegate = application_delegate;
  [app activateIgnoringOtherApps:YES];
  [app run];
}

int main(int argc, char **argv)
{
  std::cout << "Starting Application\n";
  @autoreleasepool {
    start_program();
  }
}
```

a simple `make` invocation at the command line will build and run this
program which should open up a page to this blog.

Notice that `C++` wasn't really used, I added it here to make this
blog post content rich and code sample very reusable for you as a
starter post.

A post in the future will show you how to add `NSMenu`(s) to this
application.
