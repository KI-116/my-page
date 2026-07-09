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
