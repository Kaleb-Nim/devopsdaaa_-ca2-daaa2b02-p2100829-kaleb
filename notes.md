# Deploy exported DL model Locally

Once Model comes out save as folder shit TF

cmd prompt + Docker Desktop running

`docker run --name digit_server -p 8501:8501 -v
"D:/models/dooa/img_classifier:/models/img_classifier" -e MODEL_NAME=img_classifier -t
tensorflow/serving &`

Basically runs TF server 

* `img_classifier` called during REST API

## Results

* Docker Desktop shld have container
* hit url `http://localhost:8501/v1/models/img_classifier`
  * shld see like a json thingy appear

# connect both our development container and digit_server to the same network

`digit_server` is the u gave for the docker run thingy

So at this point u shld have 2 docker containers 
* one digit server
* one DL Model server??? ( where did this come from ) 

cmd prompt

`
## Create a network
docker network create ml_network
## Connect digit_server to ml_network
docker network connect ml_network digit_server
## Connect DLMODEL_Server to ml_network
docker network connect ml_network DLMODEL_Server
`

## Testing if network works

`
apt-get update
apt-get install iputils-ping
`

`
ping digit_server
`

## Results:

pinging shit comes out

# Unit Testing for Model

Create folder directly after the repo folder called `tests` --> `test_docker.py`

* Load CIFAR100 Dataset and subset [] for prediction
* Server url to make POST req
  * `'http://digit_server:8501/v1/models/img_classifier:predict'`
    * NOTE: Need to update this URL after deploy to render, just change the http://digit_server:8501 to whatever
* make_prediction function(subset of data frm above)
  * `def make_prediction(instances):
 data = json.dumps({"signature_name": "serving_default",
 "instances": instances.tolist()}) #see [C]
 headers = {"content-type": "application/json"}
 json_response = requests.post(url, data=data, headers=headers)
 predictions = json.loads(json_response.text)['predictions']
 return predictions
 ` 
 * pip install pytest and import
## Results:

normal pytest shit the 1 passed in 2.76s

# DockerFile 4 deployment to render

Create one directly after progect repo also

Just chug this into docker file lmao

`
FROM tensorflow/serving
COPY / /
# RUN apt-get -y update
# && apt-get install -y git && git reset --hard
ENV MODEL_NAME=img_classifier MODEL_BASE_PATH=/
EXPOSE 8500
EXPOSE 8501
RUN echo '#!/bin/bash \n\n\
tensorflow_model_server \
--rest_api_port=$PORT \
--model_name=${MODEL_NAME} \
--model_base_path=${MODEL_BASE_PATH}/${MODEL_NAME} \
"$@"' > /usr/bin/tf_serving_entrypoint.sh \
&& chmod +x /usr/bin/tf_serving_entrypoint.sh
`

After this just commit to main 

## Render deploy with `Docker` as the environment

## Results:

After change url --> run pytest again

Should get the same pytest shit woo
