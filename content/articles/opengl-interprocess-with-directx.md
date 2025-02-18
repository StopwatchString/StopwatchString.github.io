---
date: '2025-02-15T20:38:16-05:00'
draft: true
title: 'OpenGL Interprocess Resource Sharing via DirectX'
---

Recently I encountered a problem at work where I needed to take a graphic being rendered in one OpenGL context and move it into another OpenGL context, located in another process. But what's the best way to do this?

We already had a solution for this in our codebase- operating system shared memory.

### Process 1

{{< highlight cpp "linenos=inline, hl_Lines=3" >}}
/* Setup */

int main()
{
    std::cout << "Hello World" << std::endl;
    return EXIT_SUCCESS;
}

{{< /highlight >}}

### Process 2

{{< highlight cpp "linenos=inline, hl_Lines=3" >}}

#include <iostream>

int main()
{
    std::cout << "Hello World" << std::endl;
    return EXIT_SUCCESS;
}

{{< /highlight >}}
