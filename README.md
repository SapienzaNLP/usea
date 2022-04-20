# Universal Semantic Annotator (USeA)

Universal Semantic Annotator (USeA) is the **first unified API for three primary tasks in Natural Language Understanding (NLU)**:

* **Word Sense Disambiguation (WSD):** the task of assigning the most appropriate sense to a word in context;
* **Semantic Role Labeling (SRL):** the task of extracting the predicate-argument structures within a sentence;
* **Semantic Parsing (Abstract Meaning Representation, AMR):** the task of representing a text in a structured semantic graph.

The main motivations behind USeA are manifold: i) the creation of an easy-to-use tool and service for the automatic annotation of explicit semantic knowledge in 100 languages, ii) enabling the use of explicit semantics in multilingual and cross-lingual real-world applications, iii) the democratization of state-of-the-art systems that would otherwise require expert knowledge of the field for their development and implementation, and last but not least iv) fostering further research in NLU and other fields on the interplay between semantics and other modalities, e.g., computer vision, speech recognition, video understanding.

## Image Description

This repo is for the Docker image of the USeA service proxy. This image takes care of:

1. Receiving an HTTP/S request to process an input text;
2. Sending task-specific requests to task-specific endpoints (preprocessing, WSD, SRL, AMR parsing);
3. Processing and merging the results from the task-specific endpoints;
4. Returning all the annotations.
**NOTE:** This image does not perform any preprocessing or annotation. For these tasks, please refer to `usea-preprocessing`, `usea-wsd`, `usea-srl`, and `usea-amr` (soon to be released).

### How to start a `usea-service` container

Make sure you have installed [docker](https://docs.docker.com/get-docker/) before proceeding, then run the following command:

```bash
docker run --name sapienzanlp/usea-service:1.0.2 -p 22000:8000 usea-service
```

If you want to run the container in the background, simply use the flag `-d` as follows:

```bash
docker run -d --name sapienzanlp/usea-service:1.0.2 -p 22000:8000 usea-service
```

If everything went well, the service will become available at `localhost:22000/process`.
You can check that everything is fine with the following Python script:

```python
import requests
import json

text = "La volpe veloce salta sopra il cane pigro."
response = requests.post(
    "http://localhost:8000/process", json={"type": "text", "content": text}
)
print(json.dumps(response.json(), indent=2))
```

#### Specifying your own endpoints

By default, the proxy image sends the requests to our online servers.
You can specify the URL of the preprocessing, WSD, SRL and AMR parsing enpoints to point to your own (local) instances, as follows:

```bash
BASE_HOST=https://nlp.uniroma1.it/usea
PREPROCESSING_ENDPOINT="$BASE_HOST"/preprocessing
WSD_ENDPOINT="$BASE_HOST"/wsd
SRL_ENDPOINT="$BASE_HOST"/srl
AMR_ENDPOINT="$BASE_HOST"/amr

docker run --name usea-service -p 8000:8000 usea-service \
    -e PREPROCESSING_ENDPOINT=$PREPROCESSING_ENDPOINT \
    -e WSD_ENDPOINT=$WSD_ENDPOINT \
    -e SRL_ENDPOINT=$SRL_ENDPOINT \
    -e AMR_ENDPOINT=$AMR_ENDPOINT 
```

### Stopping the container

Simply run:

```bash
docker stop usea-service
```

If you also want to remove the container:

```bash
docker stop usea-service
docker rm usea-service
```

### Deploy USeA locally

#### Using Docker Compose

USeA can be started using [Docker Compose](https://docs.docker.com/compose) by simply runing the following command:

```bash
docker-compose up -d
```

It will be available at `localhost:22000/process`. By default, it will run the CPU version of the images.

##### GPU Support

If you want to use the GPU, you can use one (or more) configuration files inside `docker-compose-files` folder. Let's say, for example, we want to deploy USeA with GPU support for `usea-amr`. What you have to do is running the following command:

```bash
docker-compose -f docker-compose.yaml -f docker-compose-files/docker-compose.amr.cuda.yaml up -d
```

You can read [here](https://docs.docker.com/compose/extends/) for more information about using multiple configutation file.

#### Without Docker Compose

If you don't want to use Docker Compose, you can manually pull the images from our [Docker Hub](https://hub.docker.com/repository/docker/sapienzanlp), and run them as usual.

## Acknowledgements

The authors gratefully acknowledge the support of the [European Language Grid project No. 825627 (Universal Semantic Annotator, USeA)](https://live.european-language-grid.eu/catalogue/project/5334/) under the European Union’s Horizon 2020 research and innovation programme.

## License

This work is under the Attribution-NonCommercial 4.0 International (CC BY-NC 4.0) license.
