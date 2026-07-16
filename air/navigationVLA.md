# 0307
## architecture
decoupled architecture for high-level visual language understanding and low-level locomotion control
- high-level visual language understanding 
    - a VLM(**VILA**) to process single-view images and generate waypoint instructions in natural language
    - VILA: a vision encoder, a projector, and an LLM
        - memory frames / current observation
- low-level locomotion control
    - locomotion policy translates into precise joint movements for real-time robot control

## zero-shot deployment

- the test scenes span  Workspace, Home, and Outdoor, and the model achieved an overall success rate of 88% without fine-tuning for each scene. 
- try zero-shot deployment as a starting point for further fine-tuning and adaptation to outdoor navigation.

## fine-tuning

### dataset
- original SFT dataset
    - YouTube Human Touring videos
    - Navigational data from simulations(R2R, RxR, Habitat)
    - envdrop: auxiliary task of navigation trajectory, sample the first frame and  historical frames and annotate
    - Visual Question Answering:ScanQA, real-world 3D scan QA pairs
- fine-tuning dataset (LoRA/Adapter)
    - processing egocentric human touring video~200 trajectories
        - MASt3R: estimate camera poses ->  VLM description ->  LLM generated NLP instruction ->  waypoint instruction
        - add envdrop labels
    - original SFT dataset

# 0907
# Deploy

## Edge Deploy (Jetson Orin NX 16GB)

- Model < 4B
- Quantization

## Server-Client (Same Local Network)

- Inference (System 2, <10 Hz) on server
- Client <--HTTP--> Server
    - Robot: Client → Server (image)
    - Inference Server: Server → Client
        - `discrete_action`
        - `trajectory`
        - `pixel_goal`
- Low-level robot control (~30 Hz) on device

# Outdoor Navigation Task

```text
GPS Module 
    │
    ▼
Global Path Planner
    │
    │ Generate a coarse waypoint sequence
    ▼
Waypoint middle Layer
    │
    │ Convert the next waypoint or a sequence of future
    │ waypoints into navigation conditions for the VLA model,
    │ such as textual navigation instructions
    │ (e.g. "Move forward. The target is approximately
    │ 15 meters ahead on your left.")
    │ or spatial coordinate embeddings appended to the VLA input.
    ▼
InternVLA-N1 or other VLA model
    │
    │ Perform local perception, obstacle avoidance,
    │ and trajectory generation conditioned on the injected
    │ waypoint information.
    ▼
ros_bridge.py
    │
    ▼
cmd_vel
```