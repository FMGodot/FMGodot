## FM Godot

Below is a copy from my original Blog: [Original Blog(The Latest)](https://frozenmist.com/blog/[Research]%20FMGodot%20Real%20Time%20360%20Stereoscopic%20Omni-Directional%20Panorama)

- Customised edition based on Godot Engine 4.7
- Adding support of real time 360 stereo capture support by Leoson Cheong (CHEONG Tai Leong)
- Email: support@frozenmist.com
- Forum: [Godot Official Discussion](https://forum.godotengine.org/t/142834)
- Forum: [Godot Community Discussion](https://godotforums.org/d/44134)
- [Discord Discussion @FMLeoson](https://frozenmist.com/app_fmurls/fm_redirect.php?url=discord)

---

## **Breaking the Stitching Barrier: Real-Time 12K 360 Stereoscopic Rendering in Godot**
![FMODS_Demo](./Media/FMODS_Demo_Gif/FMODS_Demo_CoverPreview.gif)

Achieving high-fidelity, real-time omnidirectional stereoscopic (ODS) rendering has traditionally been a brute-force endeavor. Rendering a scene for VR, domed projections, or immersive video usually requires massive computational overhead to avoid visual artifacts. 

Recently, I’ve been developing a purely mathematical approach to 360 stereoscopic rendering that bypasses these traditional bottlenecks. The theory is engine-agnostic—originally tested in Unreal Engine 5.7 as a lightweight alternative to NDisplay—but for this proof-of-concept, I’ve implemented it directly into a custom source build of Godot, which I’m calling **FMGodot**. 

Here is a look at how this mathematical approximation works, why it outperforms traditional stitching, and how it handles 12K resolutions in real-time.

[![Watch the video](./Media/FMODS_Cover/FMGodot_FMODS_YoutubeCover.webp)](https://www.youtube.com/watch?v=b8zXncL1XCo)

### - Omnidirectional Result Projected onto a Planar Surface
|  **Camera3D (Left Eye)** | **Camera3D (Right Eye)** |
|--------|--------|
| ![FMODS_Demo_Godot_Editor](./Media/FMODS_Demo_Screenshots_webp/FMODS_Demo_Camera3D_Left.webp) | ![FMODS_Demo_Godot_Editor](./Media/FMODS_Demo_Screenshots_webp/FMODS_Demo_Camera3D_Right.webp) |
|exaggerated 80cm IPD|exaggerated 80cm IPD|

### - [Github repository: FMGodot ](https://github.com/FMGodot/FMGodot)
> Work-in-progress...

<!-- truncate -->


---

|  **The latest FM Godot Implementation based on Godot 4.7** |
|--------|
| ![FMODS_Demo_Godot_Editor](./Media/FMODS_Demo_Screenshots_webp/FMODS_Demo_Godot_Editor.webp) |
| Testing Capture Method: Multi-Pass ODS(Single-Pass might be possible for future version) |


|  **Early Test in Unreal 5.7 (Correct alignment and better performance than NDisplay)** |
|--------|
| ![FMODS_Demo_Unreal_Editor](./Media/FMODS_Demo_Screenshots_webp/FMODS_Demo_Unreal_Editor.webp) |
| Testing Capture Method: Single-Pass ODS |

|  **Original Implementation in Unity3D (default 360 Stereo Capture)** |
|--------|
| ![FMODS_Demo_Unreal_Editor](./Media/FMODS_Demo_Screenshots_webp/FMODS_Demo_Unity3D_Editor.webp) |
| Testing Capture Method: Default 360 Stereo Capture(per eye) |

## **The Bottleneck of Traditional 360 Rendering**

Standard game engine approaches to 360-degree rendering typically rely on multi-camera rigs—most commonly, rendering out to a cubemap (6 directional cameras) or capturing multiple overlapping frustums and blending them together. 

### - This traditional method introduces several critical limitations

*   **Incompatible with Real-Time Slicing:** Mathematically true ODS relies on raytracing or rendering discrete vertical pixel slices to shift the eye vector dynamically. For a game engine rasterizer, evaluating per-pixel camera shifts or hundreds of vertical passes is far too computationally expensive for real-time playback.
*   **Multi-Pass Camera Rig Overhead:** Standard rasterization attempts to fake ODS using multi-camera arrays or stereoscopic cubemaps. Rendering 12+ total passes per frame (6+ faces per eye) multiplies draw calls and geometry pipeline work exponentially, tanking framerates.
*   **Stitching Seams & Depth Distortion:** Multi-camera setups inevitably produce visible stitching artifacts at frustum boundaries, while failing to maintain accurate interpupillary distance (IPD) across the 360-degree view.

---

## **The Engine Dilemma: Why Move Beyond Unity3D?**

Historically, standard 360 stereoscopic capture relied heavily on Unity3D's Built-in Render Pipeline. However, as Unity shifted focus toward Scriptable Render Pipelines (URP/HDRP) and officially initiated the deprecation of the Built-in pipeline, built-in 360 stereoscopic tools and camera workflows were largely left unmaintained or broken across pipeline revisions.

This transition underscores a major architectural challenge for developers building custom rendering systems:

*   **Pipeline Migration & Feature Deprecation:** Unity’s history of deprecating core rendering pipelines every few years forces developers to continually rewrite custom graphics passes, with niche features like native 360 ODS capture frequently getting dropped or broken.
*   **Proprietary Source Restrictions:** Unity’s core source code is private and closed. Developers cannot patch low-level engine code, inject custom view-projection matrices, or maintain custom ODS renderers natively inside the engine when Unity updates or breaks rendering APIs.

### - Seeking Engine Independence: Godot & Unreal Engine

To establish a future-proof foundation, we turned to engines that offer deep source-level access:

*   **Godot Engine (FMGodot):** Godot is completely open-source under the permissive MIT license. With zero licensing fees, royalty structures, or proprietary lock-in, developers have full access to modify C++ core engine code. This allowed us to build **FMGodot**—a custom engine build that natively supports low-level 360 stereoscopic projection math directly inside the engine source.
*   **Unreal Engine 5.7:** Unreal provides full source access for compilation and internal project development, making it an ideal candidate for high-end rendering research. Our early tests in UE 5.7 demonstrated that this mathematical ODS approach offers a far more performant and lightweight alternative to heavy solutions like NDisplay (though external distribution of modified engine binaries remains subject to Unreal's EULA).

---

## **The Mathematical Approximation (Inspired by Google ODS)**

Google’s Omnidirectional Stereo (ODS) logic solves the IPD mismatch by effectively simulating a continuous circle of cameras, ensuring every vertical slice of the panoramic image has the correct stereo disparity. However, traditional ODS is computationally heavy and typically reserved for offline rendering.

My solution mirrors the results of Google’s ODS logic but uses a mathematical approximation injected directly into the engine's projection math. 

Instead of relying on multi-camera rigs and post-process stitching, the rendering pipeline calculates the panoramic stereoscopic projection in a single, continuous sweep. By modifying the core engine code in **FMGodot**, we bypass the need for stitching entirely. The result is a mathematically continuous image that perfectly maintains the stereoscopic illusion at a fraction of the computational cost. 

### - Applying an analytic coordinate mapping
> Translating 3D spatial positions directly into a target coordinate system using exact, closed-form mathematical equations. Instead of relying on multi-step sampling, lookup tables, or image-based blending, coordinates are mapped continuously and deterministically in a single calculation step.

### - Non-linear view-projection transformation
> A projection method that maps 3D scene data into 2D display space using curved geometric math rather than standard flat, linear perspective matrices. This enables native rendering for non-planar outputs—such as domes, cylinders, or panoramas—where standard perspective projection would otherwise cause severe stretching or distortion.

### - Evaluating panoramic disparity within the pipeline
> Calculating stereoscopic depth and eye separation (interpupillary distance) dynamically across all 360-degree viewing angles inside the primary rendering pass. Rather than stitching pre-rendered left and right eye images together in post-processing, stereoscopic depth differences are evaluated natively as scene data moves through the graphics engine.

### - Omnidirectional space transformation
> An execution model where spatial coordinates are transformed prior to rasterization, allowing standard, linear planar viewports to directly record mathematically correct ODS perspective. Even when leveraging multiple standard engine viewports to cover a full 360-degree FOV due to engine constraints, each viewport captures a natively aligned stereoscopic output—eliminating the need for complex multi-camera ODS setups, depth-buffer merging, or post-process stitching logic.

---

## **Why Is This Solution Necessary?**

### - Real-World Use Cases
Traditional rendering methods often force developers to choose between real-time interactivity and visual fidelity. By moving to a lightweight, mathematically continuous projection, this solution unlocks high-resolution (12K+) real-time stereoscopic rendering for several demanding applications:

*   **360 Stereoscopic Projection & LED Wall Systems:** Modern virtual production and large-scale immersive installations demand massive resolutions. Rendering 12K stereo in real-time allows for dynamic, interactive environments on massive LED volumes without the latency, sync issues, or visual tearing introduced by stitching multiple camera feeds.
*   **Stereoscopic 360 Animation & Video:** For creators publishing VR movies to platforms like YouTube VR, offline rendering of 360 stereoscopic video is notoriously slow. This engine-level approach allows creators to instantly preview, capture, or even directly stream high-fidelity 360 stereo output, drastically cutting down production and encoding pipelines.
*   **Immersive VR Experiences:** In virtual reality, maintaining the correct IPD across the entire field of view is critical. This mathematical approach guarantees flawless depth perception and eliminates the eye strain caused by stitched seams. Because the computational overhead is so low, it makes high-fidelity stereoscopic environments highly scalable across diverse hardware—from standalone mobile-based headsets to high-end tethered desktop rigs.
*   **Real-Time Immersive Systems (CAVE & Dome Theaters):** Planetariums, dome theaters, and CAVE setups often rely on complex multi-projector rigs that require perfectly warped and blended visuals. Generating a seamless, distortion-free stereoscopic projection in a single pass ensures precise mapping onto curved surfaces, making interactive, real-time dome presentations much more accessible and performant.

### - Performance & Compatibility 

Because this method relies on optimized mathematical approximations rather than brute-force multi-pass rendering, it is incredibly lightweight. The custom pipeline is highly adaptable across different hardware targets and rendering backends. 

**Current Testing Benchmarks:**

| Metric | Detail |
| :--- | :--- |
| **Engine** | FMGodot (Custom Godot Source) |
| **Renderer** | Forward+ & Forward (Mobile) Compatible |
| **Resolution** | 12K (3840 x 3 Horizontal Pixels) |
| **Hardware** | Apple M5 Max MacBook Pro |
| **Performance** | 60+ FPS |
| **IPD** | 6.4 cm |
| **Cylindrical Screen Simulation** | Radius: 3.825 meter, Height: 3.84 meter |
| **Zero Parallax Distance** | 3.825 meter |

Hitting a sustained 60+ FPS at 12K resolution on a laptop—even a powerful Apple Silicon machine—demonstrates the viability of handling high-end immersive rendering without needing a server rack of dedicated GPUs. 

---

## **Demo Projection Matrices**

The underlying math supports various projection matrix outputs depending on the display target. Below are real-time screenshots from the FMGodot engine demonstrating the custom projection mapping:

### - Cylindrical Projection

> for 360-degree stereo immersive LED Wall system or projection system

|  **FM Godot Cylindrical Projection** |
|--------|
| ![FMODS_Demo](./Media/FMODS_Demo_Screenshots_webp/FMODS_Demo_Cylindrical_capture1.webp) |
| ![FMODS_Demo](./Media/FMODS_Demo_Screenshots_webp/FMODS_Demo_Cylindrical_capture2.webp) |
| ![FMODS_Demo](./Media/FMODS_Demo_Screenshots_webp/FMODS_Demo_Cylindrical_capture3.webp) |
| ![FMODS_Demo](./Media/FMODS_Demo_Screenshots_webp/FMODS_Demo_Cylindrical_capture4.webp) |
| ![FMODS_Demo](./Media/FMODS_Demo_Screenshots_webp/FMODS_Demo_Cylindrical_capture5.webp) |
| ![FMODS_Demo](./Media/FMODS_Demo_Screenshots_webp/FMODS_Demo_Cylindrical_capture6.webp) |


### - Equirectangular Projection

> for stereo VR headmounted Display, Full Dome or Half Dome cinematic system

|  **FM Godot  Equirectangular Projection** |
|--------|
| ![FMODS_Demo](./Media/FMODS_Demo_Screenshots_webp/FMODS_Demo_Equirectangular_capture1.webp) |
| ![FMODS_Demo](./Media/FMODS_Demo_Screenshots_webp/FMODS_Demo_Equirectangular_capture2.webp) |
| ![FMODS_Demo](./Media/FMODS_Demo_Screenshots_webp/FMODS_Demo_Equirectangular_capture3.webp) |
| ![FMODS_Demo](./Media/FMODS_Demo_Screenshots_webp/FMODS_Demo_Equirectangular_capture4.webp) |
| ![FMODS_Demo](./Media/FMODS_Demo_Screenshots_webp/FMODS_Demo_Equirectangular_capture5.webp) |
| ![FMODS_Demo](./Media/FMODS_Demo_Screenshots_webp/FMODS_Demo_Equirectangular_capture6.webp) |
| ![FMODS_Demo](./Media/FMODS_Demo_Screenshots_webp/FMODS_Demo_Equirectangular_capture7.webp) |
| ![FMODS_Demo](./Media/FMODS_Demo_Screenshots_webp/FMODS_Demo_Equirectangular_capture8.webp) |
| ![FMODS_Demo](./Media/FMODS_Demo_Screenshots_webp/FMODS_Demo_Equirectangular_capture9.webp) |

### - Cubic Projection
> for CAVE, half CAVE or Wall Projection system

|  **FM Godot Cubic Projection (Left,Front,Right,Back,Top,Bottom)** |
|--------|
| ![FMODS_Demo](./Media/FMODS_Demo_Screenshots_webp/FMODS_Demo_Cubic_capture1.webp) |
| ![FMODS_Demo](./Media/FMODS_Demo_Screenshots_webp/FMODS_Demo_Cubic_capture2.webp) |
| ![FMODS_Demo](./Media/FMODS_Demo_Screenshots_webp/FMODS_Demo_Cubic_capture3.webp) |

---

### - The Demo Scene: FM Polygon Japan Sensoji

To push the renderer, I used the [**FM Polygon Japan Sensoji**](https://sketchfab.com/3d-models/fm-polygon-japan-sensoji-b57471898c00466381b2395f0a204309) environment. This asset (formerly a Staff Pick on Sketchfab) features intricate geometry, rich textures, and complex architectural lines that would immediately reveal any stitching artifacts or stereoscopic tearing if they existed. 

You can check out the original 3D asset on Epic’s Fab platform here: 
[FM Polygon Japan Sensoji on Fab](https://www.fab.com/listings/d4e70338-ecc8-4efc-8206-bb2c92194487)
