# Pipeline State  

In legacy OpenGL or Direct3D11 APIs, each state can be set separately.  

Evidently, it is not efficient to change the state of the hardware frequently. And by "State Changes" of "18. Pipeline Optimization" of [Real-Time Rendering Fourth Edition](https://www.realtimerendering.com/), the application may issue redundant state changing calls. This makes the situation even worse.  

Thus, the driver of the legacy OpenGL or Direct3D11 APIs will allocate a block of memory, which contains all state, from the command buffer for each draw call. The state changing calls are merely modifying the content of the allocated memory rather than actually changing the state.  
  
Evidently, it is not efficient to allocate a block of memory from the command buffer for each draw call. It is more efficient for the application to allocate and initialize a block of memory in advance, and reuse the memory at each frame.  

This is the idea of the pipeline state object in modern Vulkan or Direct3D12 APIs. All states are grouped into a large single pipeline state object. When the application creates the pipeline state object, the driver will allocate and initialize a block of memory which contains all states. And this memory will be reused at each frame when the application binds the pipeline state object.  
