# Resource Binding  

In legacy OpenGL or Direct3D11 APIs, resources can be bound to each slot separately.  

Evidently, the driver of the legacy OpenGL or Direct3D11 APIs needs to allocate a block of memory, which contains all resource bindings (essentially, the address of the resources), from the command buffer for each draw call. And then the driver needs to provide this allocated block of memory to the hardware before the draw call is issued.  

Analogous to the pipeline state, it is not efficient to allocate a block of memory from the command buffer for each draw call. It is more efficient for the application to allocate and initialize this block of memory in advance, and reuse this memory at each frame.  

This is the idea of of the descriptors in modern Vulkan or Direct3D12 APIs. The application allocates and initailzes a block of memory (namely, the descriptors) in advance, and then keep reusing this block of memory by providing this memory to the hardware before the draw call is issued.

N/A | allocate memory  | initailze memory | bind memeory 
:-: | :-: | :-: | :-:  
Vulkan | vkCreateDescriptorPool <br/> vkAllocateDescriptorSets | vkUpdateDescriptorSets | vkCmdBindDescriptorSets   
Direct3D12 | ID3D12Device::CreateDescriptorHeap | ID3D12Device::CreateShaderResourceView <br/> ID3D12Device::CreateSampler <br/> ID3D12Device::CreateUnorderedAccessView | ID3D12GraphicsCommandList::SetDescriptorHeaps <br/> ID3D12GraphicsCommandList::SetGraphicsRootConstantBufferView <br/> ID3D12GraphicsCommandList::SetGraphicsRootDescriptorTable <br/> ID3D12GraphicsCommandList::SetComputeRootConstantBufferView <br/> ID3D12GraphicsCommandList::SetComputeRootDescriptorTable  

Since the descriptors are essentially blocks of memory (which essentially contains the address of the resources), the descriptors in modern Vulkan or Direct3D12 APIs should be simply treated as assets, merely following the rules to store the vertex buffers of a mesh or the textures of a material.  

Here is a typical pipeline layout of Vulkan. By convention, different descriptor set denotes different update frequency. But it should be noted that this update frequency denotes the updating of the resource bindings instead of the updating of the resource data (e.g., CPU writing into the uniform buffer).    
location              | type                   | readable name                          | introduction  
:-                    | :-                     | :-                                     | :-  
set = 0               | N/A                    | update none descriptor set             | the vkUpdateDescriptorSets is used during rendering module initialization  
set = 0 binding = 0   | UNIFORM_BUFFER_DYNAMIC | upload buffer                          | e.g., camera data, screen data, parameters of different rendering passes, etc <br/> the resource data may be the updated per frame, but the resource binding is NOT updated 
set = 0 binding = 1   | COMBINED_IMAGE_SAMPLER | shadow map                             | e.g., the shadow map of the unique directional light  
set = 0 binding = 2   | COMBINED_IMAGE_SAMPLER | gbuffer                                | may be updated when resizing window   
set = 0 binding = 3   | COMBINED_IMAGE_SAMPLER | LUT (Lookup Table)                     | e.g., the HDR (hemispherical directional reflectance)  
\-\-\-\-\-\-\-\-      | \-\-\-\-\-\-\-\-       | \-\-\-\-\-\-\-\-                       | \-\-\-\-\-\-\-\-  
set = 1               | N/A                    | update per mesh descriptor set         | the vkUpdateDescriptorSets is used during mesh initialization  
set = 1 binding = 0   | UNIFORM_BUFFER         | mesh information                       | e.g., the index type of the mesh, whether the mesh is skinned, etc  
\-\-\-\-\-\-\-\-      | \-\-\-\-\-\-\-\-       | \-\-\-\-\-\-\-\-                       | \-\-\-\-\-\-\-\-  
set = 2               | N/A                    | update per material descriptor set     | the vkUpdateDescriptorSets is used during material initialization  
set = 2 binding = 0   | UNIFORM_BUFFER         | material information                   | e.g., whether the normal texture is used, whether the emissive texture is used, etc  
set = 2 binding = 1   | COMBINED_IMAGE_SAMPLER | normal texture                         | the normal texture of the material   
set = 2 binding = 2   | COMBINED_IMAGE_SAMPLER | emissive texture                       | the emissive texture of the material   
set = 2 binding = 3   | COMBINED_IMAGE_SAMPLER | base color texture                     | the base color texture of the material   
set = 2 binding = 4   | COMBINED_IMAGE_SAMPLER | metallic roughness texture             | the metallic roughness texture of the material   


Evidently, by using this pipeline layout, the **vkUpdateDescriptorSets** is always used during initialization and NOT ever used during rendering.  

In console, an analog of the **ID3D12RootSignature** in Direct3D12 can be used.  
