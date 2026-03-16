---
layout: single
title: "Building a Drone Digital Twin for Infrastructure Inspection"
date: 2026-03-16
author_profile: true
author: granza
tags: [Drones, System Design, Digital Twins]

sidebar:
  - title: "Further Reading"
    text: |
      [**Noctua**](https://www.linkedin.com/company/noctua-tecnologias)
      
header:
  overlay_image: /assets/images/Drone_Vision_Area.png
  overlay_filter: 0.8
  caption: "Virtual Drone Field of View"
excerpt: "Using simulation to prototype a backend architecture for autonomous drone-based infrastructure inspection."
---

<script src="https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.min.js"></script>
<script>
  mermaid.initialize({ startOnLoad: true });
</script>

In this project we built a **digital twin of an inspection drone using Unity**, capable of capturing images and synchronizing them with a backend API.

This work is part of a project developed at **Noctua**, where we are building systems for **automated infrastructure inspection using aerial imagery**.

In this example scenario, the drone flies over industrial roofs and captures images that can later be analyzed to detect potential issues such as:

- structural damage
- corrosion
- leaks
- maintenance risks

Instead of immediately deploying a real drone fleet, we first validate the **inspection workflow and data pipeline using a digital twin simulation**.

This post focuses on the **system design behind the data pipeline**, not the internal implementation of the AI models.

---

# The Drone Controller Mock

To simulate the drone behavior we implemented a **simple state machine** that mocks the drone controller.

<div class="mermaid" style="text-align: center;">
---
title: Drone Movement
---
graph LR
    START(( ))
    CS["<b>Charging Station</b><br/><i>Act: Connect to API</i>"]
    SP["<b>Safe Position</b>"]
    RP["<b>Roof Position [i]</b><br/><i>Act: Take Photo</i>"]

    START --> |Start| CS
    CS -->|Battery High| SP
    SP -->|Battery Low| CS
    CS --> |Finish| START

    SP -->|Next Unvisited| RP
    RP -->|Battery Low| SP
    
    RP -->|Move to Next Roof| RP
    RP -->|Finished All Roofs| SP
    SP -->|Finished All Roofs| CS

    style SP fill:#ffffff,stroke:#333,stroke-width:2px
    style CS fill:#e6f3ff,stroke:#333,stroke-width:1px
    style RP fill:#fffde6,stroke:#333,stroke-width:1px
</div>

The drone follows a simple inspection loop.

1. It starts at the **charging station**.
2. It moves to a **safe position**.
3. It visits multiple **roof inspection points**.
4. At each inspection point, the drone captures images.
5. When the battery becomes low, the drone returns to the charging station.

A key constraint in real-world deployments is that **network connectivity is not guaranteed everywhere**.

Because of that, the drone **does not upload images immediately**.  
Instead, the captured images are synchronized with the backend API **only when the drone returns to the base**, where internet connectivity is available.

This behavior is reproduced in the simulation.

---

# Digital Twin

The drone simulation runs inside **Unity**, where we generate a digital twin of the inspection system.

The goal is not perfect physical realism. Instead, the simulation focuses on reproducing the **inspection workflow**:

- drone navigation between inspection points
- camera image capture
- generation of inspection data

Below is an example of the drone flying over an industrial roof.

![Drone over the Roof](../../../assets/gifs/drone-spray.gif)

During the simulation, the drone periodically captures images from its camera.

![Drone Vision Area](../../../assets/images/Drone_Vision_Area.png)

These images represent the **inspection data** that would normally be collected in a real infrastructure monitoring scenario.

---

## Usage in Other Simulations

One advantage of building a digital twin is that the simulation infrastructure can be reused for other inspection scenarios.

For example, the same framework can be adapted to simulate **warehouse inspection**, where a drone scans shelves and identifies stored items.

![Drone Inside a Warehouse](../../../assets/gifs/drone-industry.gif)

Digital twins allow us to experiment with:

- inspection workflows
- data pipelines
- perception systems

without requiring expensive or risky real-world testing.

---

# The Data Journey

Once images are captured, they must travel through our backend system.

The objective of this pipeline is to transform **raw aerial images into infrastructure inspection data**.

<div class="mermaid" style="display: flex; justify-content: center; margin: 30px 0;">
graph TD
    DR(("Drone"))
    AI["AI API"]
    RD[("Redis Queue")]

    subgraph SYS ["Our API System"]
        API["FastAPI Kernel"]
        DB[("MongoDB")]
        S3["MinIO (S3)"]
        BI["Grafana DB"]
    end

    DR  --> |Push Photo + MetaData| API
    API --> |Save Binary| S3
    API --> |Save Document| DB
    API --> |Enqueue Task| RD
    RD  --> |Process Task| AI
    AI  --> |Update Result| DB
    DB  --> |Visualize| BI

    style DR fill:#e6f3ff,stroke:#333
    style AI fill:#fffde6,stroke:#333
    style RD fill:#ffe6e6,stroke:#333
    style SYS fill:#f5f5f5,stroke:#999
</div>

The pipeline works as follows:

1. The drone sends **images and metadata** to the API.
2. The API stores the **raw image binary** in S3-compatible storage using **MinIO**.
3. The metadata describing the inspection is stored in **MongoDB**.
4. The API enqueues a processing task in a **Redis queue**.
5. A separate service processes the image.
6. The processing results are written back to the database.
7. The inspection results can then be visualized using **Grafana dashboards**.

Separating **data ingestion**, **storage**, and **processing** allows each component to scale independently.

This architecture also makes the system resilient to temporary spikes in image ingestion.

---

# A Very Simple API

On the backend side, the API is intentionally simple.

Its responsibilities are limited to:

- receiving drone inspection images
- storing the image binary
- storing metadata
- dispatching asynchronous processing tasks

The AI system that analyzes the images runs as a **separate service** and consumes tasks from the queue.

This separation allows the inspection pipeline to evolve independently from the machine learning models.

---

# Final Thoughts

Building a digital twin proved to be a powerful way to validate the **inspection workflow and backend architecture** before deploying a real drone system.

Even with a simplified simulation we were able to test:

- the inspection data pipeline
- the API design
- asynchronous task processing
- storage and visualization infrastructure
