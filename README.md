Documentation to Manually Deploy Click Tracker & Consumers on Kubernetes - Docker
-----------------------------------------------------------------------------------

What it is?

The documentation explains you how to manually deploy click tracker & consumers on Kubernetes cluster via docker image. Running Kafka consumers on Kubernetes enables organizations to simplify operations such as updates, restarts, and monitoring that are more or less integrated into the Kubernetes platform.

Terms used:
Kafka consumers:
Kafka consumers are the subscribers responsible for reading records from one or more topics and one or more partitions of a topic from the Kafka server. Topic is a category/feed name to which records are stored and published. 

Kubernetes:
Kubernetes manages the cluster of worker and master nodes and allows you to deploy, scale, and automate containerized workloads.

Docker: 
Docker image: A Docker image is a read-only template that contains a set of instructions for creating a container.

Requirements:
Administrative access to GCP(Google cloud platform).
Administrative access to ansible server.
Read permission of Github repo.


Ansible server for building docker images :
To manually deploy the application and to perform the task we need a server/system on which required packages & applications are installed.
Ansible server which is running on the GCP(Google cloud platform) refer to img1.jpg. This ansible server is used here for performing a task like building docker images and fetching repo from GitHub. When you log in to the server, Docker and git is already installed on the server & you can check it by typing the command (docker --version). The docker image that we build for deploying consumers on Kubernetes is build in this ansible server.
To start the ansible server:-
gcloud compute instances start ansible --project trackier-mmp --zone europe-west1-c<img width="687" alt="img1" src="https://user images.githubusercontent.com/42905470/174430878-2d33e794-9dab-49ca-8711-8f7db1f297dc.png">


Steps to follow To deploy click Tracker and Consumers manually on Kubernetes.

STEP:- 1 General info

To build the docker image we need Dockerfile and bash scripts for automating the task. The resources are found in the GitHub repo (github.com/trackier/streaming-mmp). refer img2.jpg <img width="1139" alt="img2" src="https://user-images.githubusercontent.com/42905470/174433288-c818d80f-394c-4ff7-afc0-168a84e67711.png">



STEP:- 2 Gathering resources.

After login to the ansible server, We do git pull under directory /root/Kafka-consumer/streaming-mmp/ to build the new docker image for click consumer. After git pull You can see there is Dockerfile & init.sh file under the click consumer directory (/root/kafka-consumer/streaming-mmp/click/). The init.sh file is used for automating kafka consumers in mmp and Dockerfile building new kafka-consumer images refer img3 and img4.jpg
<img width="765" alt="img3" src="https://user-images.githubusercontent.com/42905470/174433305-3d876e64-a8f4-4361-b01f-5d8f6e4197ab.png">


Explaining init.sh and Dockerfile commands

init.sh 

cd /root/kafka-consumer/streaming-mmp/              (move to streaming-mmp dir )

eval "$(ssh-agent -s)" & ssh-add ~/.ssh/dev_readonly_cloudstuff_inc (this cmd starts the ssh agent for the github)

git pull                                            (pulling the git repo)

cd /root/kafka-consumer/streaming-mmp/consumer/clickhouse/click (move to the clickhouse dir)
rm click                                            (removing click file)
/usr/local/go/bin/go mod tidy                       (download dependencies)
/usr/local/go/bin/go build -x click.go              (build click.go)
docker image | grep "kafka-consumer-click" | awk '{print $2 $3}' | cut -d " " -f 2 | xargs docker rmi (search kafka-consumer old docker image and deleting it )
cd /root/kafka-consumer/                            (move to kafka-consumer dir)
docker build -t eu.gcr.io..$1 -f click/Dockerfile . (building new docker image with name <eu.gcr.io.. >)

Dockerfile
FROM eu.gcr.io/trackier-mmp/go:1.17.8 
COPY streaming-mmp /root/streaming-mmp (copying streaming-mmp directory to docker container under /root/)
CMD ["bash","-c", "export PATH"]       (setting PATH variable)



STEP:-3 Building new docker image for click consumer using init.sh script

Now, we build the click-consumer docker image by executing the script ./init.sh help2 (help2 is the new docker image name)
As the script run the older image is deleted and a new image for click consumer is created with the name help2 as you can see in img5jpg.
you can verify it by typing the cmd docker images ls and also on the GCP image registory. refer to img6.jpg


STEP:- 4 Deploying click consumer on Kubernetes. 

The new docker image of click consumer is ready now. To deploy click consumer on Kubernetes
Go to GCP->Kubernetes engine->workloads then click on kafka-clicks-deployment select rolling update on the action panel and change the name of the image with the new docker image name(help2) you have just created in step 3 and hit on update. refer img7
The new kafka-click-consumer is deployed on Kubernetes using the newly created docker image. refer to img8.jpg
