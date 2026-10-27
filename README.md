# Synthetic Data Generation & AI-Powered Computer Vision for Quality Inspection

**Author:** Kai Yen Chang  
**Organization:** Mercedes-Benz (MO/PB)  
**Date:**  14.11.2025 - 11.05.2025  

Note: Due to NDA, information which I can reveal is limited 

---

## Executive Summary

High-output manufacturing lines in automotive factories demand robust, real-time quality control. Training production-grade deep learning models requires massive amounts of precisely labeled training data. However, real-world data collection and manual annotation are notoriously slow, labor-intensive, and prone to severe class imbalance (defects are naturally rare in standard production).

This repository shows the two projects I developed during my internship in Mercedes Benz for Quality Control in the factory line using with **AI and Synthetic Data Generation in Simulation (Digital Twins & Procedural Generation)**:

1. **Project 1:** Procedural synthetic image and defect generation for EV battery cells utilizing **NVIDIA Omniverse** and **OpenCV**.
2. **Project 2:** Physics-based digital twin data generation and deep learning model training in **Unity** (via **realvirtual.io**) for automated connector latch inspection ona plug, validated and deployed on the production line in Berlin.

---

## Table of Contents

- [Project 1: Synthetic Image Generation for Battery Cells (NVIDIA Omniverse)](#project-1-synthetic-image-generation-for-battery-cells-nvidia-omniverse)
  - [Problem Statement & Motivation](#problem-statement--motivation)
  - [Pipeline Architecture](#pipeline-architecture)
  - [Defect Types & Procedural Generation](#defect-types--procedural-generation)
  - [Project Structure & Script Implementation](#project-structure--script-implementation)
  - [Status & Key Takeaways](#status--key-takeaways)
- [Project 2: Digital Twin Generation & AI Vision for EEC Connectors (Unity)](#project-2-digital-twin-generation--ai-vision-for-eec-connectors-unity)
  - [Context & Scope](#context--scope)
  - [Digital Twin Pipeline & Domain Randomization](#digital-twin-pipeline--domain-randomization)
  - [Model Training & Evaluation](#model-training--evaluation)
  - [Edge-Case Handling & Production Deployment](#edge-case-handling--production-deployment)
  - [Changing UR10 joint positions for inspection of different switches](#changing-ur10-joint-positions-for-inspection-of-different-switches)
- [Key Engineering Takeaways & Lessons Learned](#key-engineering-takeaways--lessons-learned)

---

## Project 1: Synthetic Image Generation for Battery Cells (NVIDIA Omniverse)

### Problem Statement & Motivation
Production lines inspect battery cell surfaces for defects using AI vision systems. High-performing neural networks need thousands of annotated defect samples.

Manual labeling of these micro-defects is exhausting, slow, and expensive. Synthetically synthesizing both non-defective (**IO** - *In Ordnung*) and defective (**NIO** - *Nicht In Ordnung*) images drastically reduces data acquisition lead times while automatically generating pixel-perfect bounding boxes and masks.

### Pipeline Architecture
<br/>

<div align="center">
  <img src="images\NVIDIA_Pipeline.png" width="650" alt="2D PCB Layout">
  <figcaption><b>Figure 1:</b> Project 1's Pipeline</figcaption>
</div>
<br/>


1. **Texture Map & Label Creation:** Procedural scripts in Python generate normal maps and opacity maps representing microscopic physical defects along with exact 2D bounding box coordinates in JSON format.
2. **Omniverse Scene Composition:** The 3D model of the battery cell is loaded into NVIDIA Omniverse. Dynamic material shaders apply the procedural normal/opacity maps onto the surface.
3. **Lighting & Camera Randomization:** Environmental reflections, light angles, intensities, and camera transforms are dynamically perturbed to simulate variations found on physical inspection stations.
4. **Rendering & Camera Transformation:** Ground-truth coordinates are mapped through the camera projection matrix to match the synthesized render outputs.


### Defect Types & Procedural Generation

The procedural pipeline simulates single and combined surface defects:
- **Blasen (Bubbles):** Circular/elliptical surface elevations affecting specular reflection.
- **Kratzer (Scratches):** Linear and curved abrasions with variable width, depth, and intensity (categorized into `light`, `mid`, and `heavy`).
- **Lunker (Pits / Cavities):** Surface voids and micro-holes.
- **Kombinationen (Multi-Defect Scenarios):** Simultaneous occurrences of scratches, bubbles, and cavities on a single cell face.
<br/>

<div align="center">
  <img src="images\Defect_types.png" width="650" alt="2D PCB Layout">
  <figcaption><b>Figure 2:</b> Defect types</figcaption>
</div>
<br/>

### Project Conclusion
- AI model was deployed inside Docker container for easy plug-and-play with robot 
- Due to issues on the factories side, real data was not given in time before the end of my internship
- However, based the the available data on hand, the model was able to train with 90% accuracy
- Since the AI only performs as well as the synthetic data(defects) generated, data(defects) could be further accurately reconstructed

---

## Project 2: Digital Twin Generation & AI Vision for EEC Connectors (Unity)

Note: Severe lack of images due to NDA

### Context & Scope
Building upon prior foundational work in Unity for a Car Plug, this initiative scaled synthetic data generation and deep learning model training to complex harness plugs on a car:
- **Primary Failure Mode:** Improperly inserted connectors 
- **Goal:** Train object detection and classification models in simulation and deploy directly into the **Mercedes-Benz plant in Berlin**.

### Digital Twin Pipeline & Domain Randomization

```
+-------------------------------------------------------------+
|                Vary position of connector
+------------------------------+------------------------------+
                               |
                               v
+-------------------------------------------------------------+
|  Vary position of lighting, wires & camera            |
+------------------------------+------------------------------+
                               |
                               v
+-------------------------------------------------------------+
|            Generate Dataset
+------------------------------+------------------------------+
                               |
                               v
+-------------------------------------------------------------+
| Export Dataset -> Train AI Model -> Deploy to Line          |
+-------------------------------------------------------------+
```

1. **Mesh Separation in Blender:** Factory CAD models initially arrived as single unified meshes. Blender was used to separate the latch/connector from the main housing, allowing dynamic translation along its insertion axis.
2. **Realistic Wire Wrapping & Texturing:** High-voltage and low-voltage harness bundles were rigged with custom striped spiral wraps and flex physics to replicate production environments.
3. **Automated Bounding Box Tagging:** Ground-truth classification (`OK` vs. `Defekt`) is generated procedurally from the digital twin's physical insertion depth parameter.

### Model Training & Evaluation

The object detection and classification model was trained and evaluated directly against real-world camera captures:

- **Training Dataset:** $3{,}000$ synthetic images generated via Unity.
- **Training Duration:** $100$ epochs.
- **Performance Metrics:**
  - **Precision:** $\approx 97\%$
  - **Recall:** $\approx 99\%$
  - **Loss:** Demonstrated steady, consistent convergence across training and validation splits.


### Edge-Case Handling & Production Deployment

#### The "Semi-Inserted" Ambiguity (*Grenzfall*)
A major challenge on the line occurs when a connector is partially plugged in. In this state, even human operators can be uncertain during visual inspection.

- **Model Output on Semi-Inserted Connectors:**
  - `Defekt`: **98.3% confidence**
  - `OK`: **40.6% confidence**

---

### Changing UR10 joint positions for inspection of different switches
As the UR10 in the factory had to inspect multiple switches on the car part, the UR10 inside the digital twin also had to be moved according to the real UR10 in the factory. Therefore, URSim was also used in manipulating the UR10 and the End-Effector  


## Key Engineering Takeaways & Lessons Learned

1. **Constant communication with Plant Engineers:** Continuous alignment with supervisors and machine vision technicians on the shop floor prevented miscommunication regarding region-of-interest (ROI) definitions, connector tolerances, and camera mounting constraints.
2. **Sim-to-Real Feasibility:** Domain randomization across illumination, harness deflection, and fine mechanical positioning allows purely synthetic data to generalize effectively to physical production vision stations.
3. **Always add a time buffer:** It is common something goes wrong in any given project. Therefore, always plan for extra flexibilisation and time.

