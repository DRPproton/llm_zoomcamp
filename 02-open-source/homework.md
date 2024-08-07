## Homework: Open-Source LLMs

In this homework, we'll experiment more with Ollama

> It's possible that your answers won't match exactly. If it's the case, select the closest one.

## Q1. Running Ollama with Docker

Let's run ollama with Docker. We will need to execute the 
same command as in the lectures:

```bash
docker run -it \
    --rm \
    -v ollama:/root/.ollama \
    -p 11434:11434 \
    --name ollama \
    ollama/ollama
```

What's the version of ollama client? 

To find out, enter the container and execute `ollama` with the `-v` flag.

> Answer => ollama version is 0.1.48

## Q2. Downloading an LLM 

We will donwload a smaller LLM - gemma:2b. 

Again let's enter the container and pull the model:

```bash
ollama pull gemma:2b
```

In docker, it saved the results into `/root/.ollama`

We're interested in the metadata about this model. You can find
it in `models/manifests/registry.ollama.ai/library`

What's the content of the file related to gemma?

#### Answer

> The content of the file is a JSON manifest describes a Docker image comprising various layers, each identified by its media type. The configuration object includes metadata about the image, and the use of hashes (digests) like SHA256 ensures integrity. Custom media types (application/vnd.ollama.*) likely correspond to specific data or functionalities required by the image.

## Q3. Running the LLM

Test the following prompt: "10 * 10". What's the answer?

#### Answers
- When using the functions on the notebook the responce is 
    - "The product of multiplying 10 by 10 is 100.\n\nHere's a breakdown:\n- You have two tens (2 x 10).\n- When you multiply them, the result is one hundred (2 x 10 = 20; then 20 x 5 = 100)."

- If I use the pront in the command line the answer vary, 
    - 1.Sure. 10 * 10.

    - 2.10 * 10 is 100.

## Q4. Donwloading the weights 

We don't want to pull the weights every time we run
a docker container. Let's do it once and have them available
every time we start a container.

First, we will need to change how we run the container.

Instead of mapping the `/root/.ollama` folder to a named volume,
let's map it to a local directory:

```bash
mkdir ollama_files

docker run -it \
    --rm \
    -v ./ollama_files:/root/.ollama \
    -p 11434:11434 \
    --name ollama \
    ollama/ollama
```

Now pull the model:

```bash
docker exec -it ollama ollama pull gemma:2b 
```

What's the size of the `ollama_files/models` folder? 

* 0.6G
* 1.2G
* 1.7G
* 2.2G

Hint: on linux, you can use `du -h` for that.

> Answer => 1.7G

## Q5. Adding the weights 

Let's now stop the container and add the weights 
to a new image

For that, let's create a `Dockerfile`:

```dockerfile
FROM ollama/ollama

COPY ...
```

What do you put after `COPY`?

> Answer => COPY ollama_files/models /app/weights

> Copy the weights file from your local system into the container's /app directory

## Q6. Serving it 

Let's build it:

```bash
docker build -t ollama-gemma2b .
```

And run it:

```bash
docker run -it --rm -p 11434:11434 ollama-gemma2b
```

We can connect to it using the OpenAI client

Let's test it with the following prompt:

```python
prompt = "What's the formula for energy?"
```

Also, to make results reproducible, set the `temperature` parameter to 0:

```bash
response = client.chat.completions.create(
    #...
    temperature=0.0
)
```

> Answer => Sure. Here's the formula for energy in terms of mass and velocity:

        **E = (1/2)*m*v^2**
        
        where:
        
        * **E** is the energy in joules (J)
        * **m** is the mass in kilograms (kg)
        * **v** is the velocity in meters per second (m/s)

How many completion tokens did you get in response?

* **304**
* 604
* 904
* 1204

