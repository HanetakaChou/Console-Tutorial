# Resource Binding  

In legacy OpenGL or Direct3D11 APIs, resources can be bound to each slot separately.  

Evidently, the driver of the legacy OpenGL or Direct3D11 APIs needs to allocate a block of memory, which contains all resource bindings, from the command buffer for each draw call. And the driver needs to provide the address of the allocated memory to the hardware before the draw call is issued.  

Analogous to the pipeline state, it is not efficient to allocate a block of memory from the command buffer for each draw call. It is more efficient for the application to allocate and initialize a block of memory in advance, and reuse the memory at each frame.  

This is the idea of of the descriptors in modern Vulkan or Direct3D12 APIs. The application allocates and initailzes a block of memory in advance, and provides the address of the memory to the hardware before the draw call is issued.

N/A | allocate memory  | initailze memory | bind memeory 
:-: | :-: | :-: | :-:  
Vulkan | vkCreateDescriptorPool <br/> vkAllocateDescriptorSets | vkUpdateDescriptorSets | vkCmdBindDescriptorSets   
Direct3D12 | ID3D12Device::CreateDescriptorHeap | ID3D12Device::CreateShaderResourceView <br/> ID3D12Device::CreateSampler <br/> ID3D12Device::CreateUnorderedAccessView | ID3D12GraphicsCommandList::SetDescriptorHeaps <br/> ID3D12GraphicsCommandList::SetGraphicsRootConstantBufferView <br/> ID3D12GraphicsCommandList::SetGraphicsRootDescriptorTable <br/> ID3D12GraphicsCommandList::SetComputeRootConstantBufferView <br/> ID3D12GraphicsCommandList::SetComputeRootDescriptorTable  

Since the descriptors are essentially blocks of memory, the descriptors in modern Vulkan or Direct3D12 APIs should be simply treated as assets, merely following the rules to store the vertex buffers of a mesh or the textures of a material.  

Here is a typical pipeline layout of Vulkan.  
location              | type                   | readable name               | introduction  
:-                    | :-                     | :-                          | :-  
set = 0               | N/A                    | global descriptor set       | the vkUpdateDescriptorSets is used during rendering module initialization  
set = 0 binding = 0   | UNIFORM_BUFFER_DYNAMIC | per frame dynamic offset    | dynamic offset of the unique global_upload_ringbuffer  
set = 0 binding = 1   | UNIFORM_BUFFER_DYNAMIC | per camera dynamic offset   | dynamic offset of the unique global_upload_ringbuffer  
set = 0 binding = ... | UNIFORM_BUFFER_DYNAMIC | per ... dynamic offset      | dynamic offset of the unique global_upload_ringbuffer  
set = 0 binding = 3   | UNIFORM_BUFFER_DYNAMIC | per drawcall dynamic offset | dynamic offset of the unique global_upload_ringbuffer 
set = 0 binding = 4   | COMBINED_IMAGE_SAMPLER | shadow maps                 | for example, the global shadow maps 
set = 0 binding = 5   | COMBINED_IMAGE_SAMPLER | LUTs                        | for example, the global LUTs  
\-\-\-\-\-\-\-\-      | \-\-\-\-\-\-\-\-       | \-\-\-\-\-\-\-\-            | \-\-\-\-\-\-\-\-  
set = 1               | N/A                    | per mesh descriptor set     | the vkUpdateDescriptorSets is used during mesh initialization  
set = 1 binding = 0   | UNIFORM_BUFFER         | mesh arguments              | for example, the immutable uniform buffer owned by the mesh  
\-\-\-\-\-\-\-\-      | \-\-\-\-\-\-\-\-       | \-\-\-\-\-\-\-\-            | \-\-\-\-\-\-\-\-  
set = 2               | N/A                    | per material descriptor set | the vkUpdateDescriptorSets is used during material initialization  
set = 2 binding = 0   | UNIFORM_BUFFER         | material arguments          | for example, the immutable uniform buffer owned by the material  
set = 2 binding = 1   | COMBINED_IMAGE_SAMPLER | albedo map                  | the albedo map owned by the material  

Evidently, by using this pipeline layout, the **vkUpdateDescriptorSets** is always used during initialization and NOT ever used during rendering.  

In console, an analog of the **ID3D12RootSignature** in Direct3D12 can be used.  
