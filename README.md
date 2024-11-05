# CARLA RAI Challenge 2024 Starter kit

This repository contains the starter kit for the CARLA RAI Challenge 2024. It is based on [Neural Attention Fields for End-to-End Autonomous Driving (NEAT)](https://github.com/autonomousvision/neat).

## Neural Attention Fields for End-to-End Autonomous Driving (NEAT) 
### [Paper](http://www.cvlibs.net/publications/Chitta2021ICCV.pdf) | [Supplementary](http://www.cvlibs.net/publications/Chitta2021ICCV_supplementary.pdf) | [Video](https://www.youtube.com/watch?v=gtO-ghjKkRs) | [Talk](https://www.youtube.com/watch?v=hYm6LPTyHHA) | [Poster](https://www.cvlibs.net/publications/Chitta2021ICCV_poster.pdf) | [Slides](https://www.cvlibs.net/publications/Chitta2021ICCV_slides.pdf)

<img src="neat/assets/neat_clip.GIF" height="270" hspace=30>

The `main` branch of this repository contains the original code for NEAT.

## Clone
To clone the repository with all submodules 

```shell
git clone -b starter-kit --recurse-submodules https://github.com/cognitive-robots/rai-neat/tree/starter-kit 
```

## Setup
Please follow the installation instructions from our [TransFuser repository](https://github.com/autonomousvision/transfuser/tree/cvpr2021) to set up the CARLA simulator. The conda environment required for NEAT can be installed via:
```Shell
conda env create -f environment.yml
conda install pytorch torchvision torchaudio cudatoolkit=11.1 -c pytorch -c nvidia
```

## Data Generation (from NEAT)
The training data is generated using ```team_code/auto_pilot.py```. Data generation requires routes and scenarios. Each route is defined by a sequence of waypoints (and optionally a weather condition) that the agent needs to follow. Each scenario is defined by a trigger transform (location and orientation) and other actors present in that scenario (optional). We provide several routes and scenarios under ```leaderboard/data/```. The [TransFuser repository](https://github.com/autonomousvision/transfuser) and [leaderboard repository](https://github.com/carla-simulator/leaderboard/tree/master/data) provide additional routes and scenario files.

### Running a CARLA Server

#### With Display

Without Docker:

```Shell
./CarlaUE4.sh
```

With Docker:

Instructions for setting up docker are available [here](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker). The following command will pull the docker image of CARLA 0.9.10.1 and run the container.

```shell
docker run --privileged --gpus all --net=host -e DISPLAY=${DISPLAY} -it -e SDL_VIDEODRIVER=x11 -v /tmp/.X11-unix:/tmp/.X11-unix carlasim/carla:0.9.10.1 /bin/bash
```
Once inside the container, run the Carla server with

```shell
./CarlaUE4.sh
```


### Running the Autopilot

Once the CARLA server is running, rollout the autopilot to start data generation.
```Shell
bash rai/scripts/run_evaluation.sh
```
The expert agent used for data generation is defined in ```team_code/auto_pilot.py```. Different variables which need to be set are specified in ```rai/scripts/run_evaluation.sh```. The expert agent is originally based on the autopilot from [this codebase](https://github.com/bradyz/2020_CARLA_challenge).

## Training (from NEAT)
The training code and pretrained models are provided below.
```Shell
mkdir model_ckpt
wget https://s3.eu-central-1.amazonaws.com/avg-projects/neat/models.zip -P model_ckpt
unzip model_ckpt/models.zip -d model_ckpt/
rm model_ckpt/models.zip
```

There are 5 pretrained models provided in ```model_ckpt/```:
- [AIM-MT (2D)](leaderboard/team_code/aim_mt_2d_agent.py): ```aim_mt_sem``` and ```aim_mt_sem_depth```
- [AIM-MT (BEV)](leaderboard/team_code/aim_mt_bev_agent.py): ```aim_mt_bev```
- [AIM-VA](leaderboard/team_code/aim_va_agent.py): ```aim_va```
- [NEAT](leaderboard/team_code/neat_agent.py): ```neat```

Additional baselines are available in the [TransFuser repository](https://github.com/autonomousvision/transfuser).

## Evaluation and Quick Test

### Running CARLA Server
With Docker:

Instructions for setting up docker are available [here](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker). The following command will pull the docker image of CARLA 0.9.10.1 and run the container.

```shell
docker run --privileged --gpus all --net=host -e DISPLAY=${DISPLAY} -it -e SDL_VIDEODRIVER=x11 -v /tmp/.X11-unix:/tmp/.X11-unix carlasim/carla:0.9.10.1 /bin/bash
```
Once inside the container, run the Carla server with

```shell
./CarlaUE4.sh
```

### Running NEAT Agent

Update the required variables in ```rai/scripts/env_var.sh``` and run the agent, with or without Docker.

*Note*: For a quick test, change `CUSTOM_ROUTE_TIMEOUT` (e.g. =15) in `rai/scripts/env_var.sh`.


#### Without Docker

```Shell
bash rai/scripts/run_evaluation.sh
```

#### With Docker

Build the Docker image by running:

```shell
bash rai/scripts/make_docker.sh -t <image:tag> # for example neat:0.1
```

Once the image is built, run the Docker image with:

```shell
docker run --ipc=host --gpus all --net=host -e DISPLAY=$DISPLAY -it -e SDL_VIDEODRIVER=x11 -v /tmp/.X11-unix:/tmp/.X11-unix <image:tag> /bin/bash
```
Finally, run the evaluation script within the Docker container:

```shell
bash rai/scripts/run_evaluation.sh
```