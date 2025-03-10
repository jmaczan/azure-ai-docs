---
title: Embedded content safety - Azure AI Content Safety
titleSuffix: Azure AI services
description: Embedded content safety is designed for on-device scenarios where cloud connectivity is intermittent or unavailable.
author: PatrickFarley
manager: nitinme
ms.service: azure-ai-content-safety
ms.topic: how-to
ms.date: 9/24/2024
ms.author: zhanxia
---

# Embedded content safety (preview)

Embedded content safety is designed for on-device scenarios where cloud connectivity is intermittent or prefer on-device for privacy reason. For example, you can use embedded content safety in a PC to detect harmful content generated by foundation model, or a car that might travel out of range. You can also develop hybrid cloud and offline solutions. For scenarios where your devices must be in a secure environment like a bank or government entity, you should first consider [disconnected containers](../../containers/disconnected-containers.md).

> [!IMPORTANT]
> Microsoft limits access to embedded content safety. You can apply for access through the Azure AI content safety [embedded content safety limited access review](https://aka.ms/aacs-embedded-application). For more information, see [Limited access](../limited-access.md).

## Platform requirements

Embedded content safety is included with the content safety C++ SDK. 

**Choose your target environment**

Embedded content safety only supports Windows right now. Contact your Microsoft account contact if you need to run embedded content safety on a different platform. 

# [Windows X64](#tab/windows-target)

Requires Windows 10 or newer on x64 hardware.

The latest [Microsoft Visual C++ Redistributable for Visual Studio 2015-2022](/cpp/windows/latest-supported-vc-redist?view=msvc-170&preserve-view=true) must be installed regardless of the programming language used with the content safety SDK.

---

## Limitations

Embedded content safety is only available with C++ SDK. The other content safety SDKs, and REST APIs don't support embedded content safety.


## Embedded content safety SDK packages


For C++ embedded applications, install following content safety SDK for C++ packages:

|Package  |Description  |
| --------- | --------- |
|[Azure.AI.ContentSafety.Extension.Embedded.Text](https://www.nuget.org/packages/Azure.AI.ContentSafety.Extension.Embedded.Text)|Required to run text analysis on device|
|[Azure.AI.ContentSafety.Extension.Embedded.Image](https://www.nuget.org/packages/Azure.AI.ContentSafety.Extension.Embedded.Image)|Required to run image analysis on device|




## Models

For embedded content safety, you need to download the content safety to your device. Microsoft limits access to embedded content safety. You can apply for access through the [embedded content safety limited access review](https://aka.ms/aacs-embedded-application). Instructions are provided upon successful completion of the limited access review process.

The embedded content safety supports [analyze text](../quickstart-text.md) and [analyze image](../quickstart-image.md) features. These features scan text or image content for sexual content, violence, hate, and self-harm with multiple severity levels. It should be noted that these embedded models have been optimized for on-device execution with less computational resources compared to the Azure API. Therefore, it's possible that the output generated from the embedded content safety model may vary from that of the Azure API.


## Embedded content safety code samples

Below is the ready to use embedded content safety samples. Follow the readme file to run the sample.

1. [C++ sample](https://github.com/Azure/azure-ai-content-safety-sdK)



## Performance evaluations 

Embedded content safety models run fully on your target devices. Understanding the performance characteristics of these models on your devices’ hardware can be critical to delivering low latency experiences within your products and applications. This section provides information to help answer the question, "Is my device suitable to run embedded content safety for text analysis or image analysis?"

### Factors that affect performance
Device specifications – The specifications of your device play a key role in whether embedded content safety models can run without performance issues. CPU clock speed, architecture (for example, x64, ARM processor, etcetera), and memory can all affect model inference speed.

CPU/GPU load – In most cases, your device is running other applications in parallel to the application where embedded content safety models are integrated. The amount of CPU/GPU load your device experiences when idle and at peak can also affect performance.

For example, if the device is under moderate to high CPU load from all other applications running on the device, it's possible to encounter performance issues for running embedded content safety in addition to the other applications, even with a powerful processor.

Memory load – An embedded content safety text analysis process consumes about 900 MB of memory at runtime. If your device has less memory available for the embedded content safety process to use, frequent fallbacks to virtual memory and paging can introduce more latencies. This can affect both the real-time factor and user-perceived latency.

### SDK parameters that can impact performance

Below SDK parameters can impact the inference time of the embedded content safety model. 

-  `gpuEnabled` Set as **true** to enable GPU, otherwise CPU is used. Generally inference time is shorter on GPU.
-  `numThreads` This parameter only works for CPU. It defines number of threads to be used in a multi-threaded environment. We support a maximum number of four threads.

See next section to for the performance benchmark data on popular PC CPUs and GPUs




### Performance benchmark data on popular CPUs and GPUs

As stated above, there are multiple factors that impact the performance of embedded content safety model. We highly suggest you test it on your device and tweak the parameters to fit for your application's requirement. 

We also conduct performance benchmark tests on various popular PC CPUs and GPUs. Keep in mind that even with the same CPU, performance can vary depending on the CPU and memory load. The benchmark data provided should serve as a reference when considering if the embedded content safety can operate on your device. For optimal results, we advise testing on your intended device and in your specific application scenario.


The [sample code](https://github.com/Azure/azure-ai-content-safety-sdk) includes code snippet to monitor performance metrics like memory, inference time. 

#### [Text analysis performance](#tab/text)

**CPU: Intel(R) Core(TM) Ultra i9 149000HX**  

| AACS_NUM_THREADS | CPU Memory Utilization | Latency |  
|------------------|------------------------|---------|  
| 1                | 940 MB                  | 260 ms   |  
| 2                | 940 MB                  | 164 ms   |  
| 3                | 940 MB                  | 125 ms   |  
| 4                | 940 MB                  | 103 ms   |  

**CPU: Intel(R) Core(TM) Ultra 5 125H**  

| AACS_NUM_THREADS | CPU Memory Utilization | Latency |  
|------------------|------------------------|---------|  
| 1                | 940 MB                  | 320 ms   |  
| 2                | 940 MB                  | 180 ms   |  
| 3                | 940 MB                  | 125 ms   |  
| 4                | 940 MB                  | 105 ms   |  

**GPU: NVIDIA GeForce RTX 4060 Laptop GPU**  

| GPU Memory Utilization | Latency |  
|------------------------|---------|  
| 940 MB                  | 26 ms    |  

**GPU: Intel(R) Arc(TM) Graphics** 

| CPU Memory Utilization | Latency |  
|------------------------|---------|  
| 940 MB                  | 80 ms    |  

#### [Image analysis performance](#tab/image)


**CPU: Intel(R) Core(TM) Ultra i9 149000HX**

| AACS_NUM_THREADS | CPU Memory Utilization | Latency |  
|------------------|------------------------|---------|  
| 1                | 1 GB                    | 350 ms   |  
| 2                | 1 GB                    | 220 ms   |  
| 3                | 1 GB                    | 200 ms   |  
| 4                | 1 GB                    | 150 ms   |  

**CPU: Intel(R) Core(TM) Ultra 5 125H**  

| AACS_NUM_THREADS | CPU Memory Utilization | Latency |  
|------------------|------------------------|---------|  
| 1                | 1 GB                    | 440 ms   |  
| 2                | 1 GB                    | 250 ms   |  
| 3                | 1 GB                    | 190 ms   |  
| 4                | 1 GB                    | 180 ms   |  


**GPU: NVIDIA GeForce RTX 4060 Laptop GPU**  

| GPU Memory Utilization | Latency |  
|------------------------|---------|  
| 1 GB                    | 80 ms    |  

**GPU:Intel(R) Arc(TM) Graphics**  

| CPU Memory Utilization | Latency |  
|------------------------|---------|  
| 1 GB                    | 280 ms   |  

---

## Related Content

- [Limited access to Content Safety](../limited-access.md)