---
date: '2025-02-15T20:38:16-05:00'
draft: false
title: 'OpenGL Interprocess Resource Sharing via DirectX'
---

Here's a problem I recently tackled. The setup:

You have two processes running, each with their own OpenGL context. In context A you have full source code control of the graphics context. In context B you have limited control- you're interfacing with someone else's render loop via a dll interface, and can't modify the underlying machinery without breaking contract for what your plugin is doing to their engine. You need to move the color output of the framebuffer from context B into context A to be used for render every frame, with minimal latency and performance impact.

There's a handful of ways to go about this. The simplest is to approach is to share that data the same way you share any other data between processes- using OS-level shared memory constructs.

This approach was already in our codebase, and looked something like this:

### Context A

{{< highlight cpp "linenos=inline, hl_Lines=3" >}}
/* Setup */

int main()
{
    std::cout << "Hello World" << std::endl;
    return EXIT_SUCCESS;
}

{{< /highlight >}}

### Context B

{{< highlight cpp "linenos=inline, hl_Lines=3" >}}

#include <iostream>

int main()
{
    std::cout << "Hello World" << std::endl;
    return EXIT_SUCCESS;
}

{{< /highlight >}}

Now functionally, this works great! We move the framebuffer's color component in Context B and push it across process boundaries into a texture in Context A. The only issue? Latency and performance! Let's break it down.

- The graphic being downloaded from context B is being done in a **blocking** way. We're adding a significant and variable amount of time to someone else's render loop. This is really not good.

- The os shared memory is being uploaded to our texture, again, in a **blocking** way. This again creates highly variable render time in the second context, as uploading a texture at 1080p was taking anywhere from 0.2ms to 7ms on my test machine.

This desynchronized blocking is a real beast to solve. You can add asynchronous upload/download from the GPU to solve the blocking issue, and use fences and interprocess signalling to fix synchronization. But this all comes at a cost of latency. And remember, we're trying to do this in real time.

## DirectX Interop

