# Video-Analytics

  

[![Kafka](https://img.shields.io/badge/streaming_platform-kafka-black.svg?style=flat-square)](https://kafka.apache.org)

[![Docker Images](https://img.shields.io/badge/docker_images-confluent-orange.svg?style=flat-square)](https://github.com/confluentinc/cp-docker-images)

[![Python](https://img.shields.io/badge/python-3.5+-blue.svg?style=flat-square)](https://www.python.org)

  

Horizontaly Scalable, Distributed system to churn out video feeds &amp; infer analytics

## Goals 
Intent of this project is to build a Minimalistic, Horizontally Scalable, Distributed system to churn out volumes of data & provide meaningful insights into the same.
The use case we are pursuing is that of churning Computer Vision data e.g. CCTV footage, You tube videos or images.

##  Why this project?
One can find numerous solutions for solving problems related to Computer Vision & data analytics in-general. However, most of these solutions are either in the form of "technology" (e.g. training a good face recognition model) or "full blown product".
We do not want to re-invent the wheel. This project intends to build a minimalistic product that is usable, as well as can act as starting code for building a more complex product.

## Code Organisation
- Each top-level folder at the root acts as "an app". 
- Each app consists of multiple microservices.
- An App is nothing but composition of these microservices.
- A docker-compose file is a place to "compose" the microservices to build the app.
- Individual microservices can talk to each other through Service Discovery or through external message broker "Kafka"
- Kafka acts as common message broker for all the apps. 
- The code common to multiple apps, is placed either at root or in a separate folder. 
TODO: We still need to come up with an effective way for multiple apps to reuse the code without stepping into each other area / breaking them.

##  Usage: CCTV-Surveillance App

 1. Change director to that of the app
```
cd cctv-surveillance
```
 2. Spin up Kafka Cluster
```
docker-compose -f ../docker-compose-kafka.yml up -d
```
3. The app looks for environment variable "VIDEO_ANALYTICS_DATA" which points to the file system path where input data can be found.
Edit the ".env" file to update the environment variable to work with your machine.

4. The input folder should have following structure
```
root |
     |- movies			# drop all your video files here (currently support .webm format)
     |- known-faces		# jpg images of known faces (e.g. manoj.jpg)
     |- outut   		# output tagged images produced by the app
     |- transient-data	# temporary files generated by the app, which are eventually deleted
```

5. Spin up the cctv surveillance app
```
docker-compose up -d
```
 6. Optional:  If you are experimenting with the applications & want to clean up everything befor you restart appliction...
```
docker exec kafka bash remove_all_topics.sh   # Removes all kafka topics used by the app
bash restart_containers.sh                    # stops running containers, clear docker logs & restart them
```

## How does it work?
The app consists of following microservices communicating to each other through Kafka topics:
```mermaid
graph LR
A[movie-feeder] -- raw-frames --> B[motion-detector]
B[motion-detector] -- enriched frames --> C[face-detector]
C[face-detector] -- frames --> D[face-matcher]
D[face-matcher] -- frames --> E[filesystem-consumer]
```
> **movie-feeder**: reads frames from the list of movie files using open-cv

> **motion-detector**: Nothing happens in the scene most of the time, when we are working with cctv. This microservice, detects the frame where something happens. Discards other frames & sends these frames for further processing

> **face-detector**: Checks if the given frame has a human face in it

> **face-matcher**: If there is face, see if it matches with known-faces. Otherwise, update library with new face

> **fileystem-consumer**: Reads recognized / matches faces & create output to file system

> **Disclaimer: I have tested this on CCTV feed generated by cameras installed in my housing society.
> While I works in-general, I havn't done any kind of exhaustive testing**

## Credits

 - Motion Detection code is taken from an excellent tutorial: https://florimond.dev/blog/articles/2018/09/building-a-streaming-fraud-detection-system-with-kafka-and-python/
 - Face Detection library: https://ourcodeworld.com/articles/read/841/how-to-install-and-use-the-python-face-recognition-and-detection-library-in-ubuntu-16-04

    
## Run linter & style checker
```
pip install autopep8
autopep8 --in-place --aggressive --aggressive <filename>
```
  

## Feature Backlog
### Refactoring
There are a number of #TODOs annotated in the code. Its a technical debt that was taken to reach a usable system at the earlier. I plan to repay this debt
  
### Features
#TODO: Create JIRA issues.
  
### Deployment
- Horizontal Scaling
- Deploy to AWS Cloud
- Profiling
