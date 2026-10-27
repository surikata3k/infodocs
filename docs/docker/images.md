# Create images

Both methods build the same small Flask app from these two files:

=== "app.py"

    ```python title="app.py"
    from flask import Flask
    app = Flask(__name__)

    @app.route('/')
    def hello_world():
        return 'Hello World Python Program.'

    if __name__ == "__main__":
       app.run(host="0.0.0.0", port=9999)
    ```

=== "requirements.txt"

    ```text title="requirements.txt"
    flask
    ```

## From a Dockerfile

```dockerfile title="Dockerfile"
FROM python:alpine3.17
WORKDIR /app
COPY . /app
RUN pip install -r requirements.txt
ENTRYPOINT ["python", "app.py"]
```

```bash
docker build . -t hello-python:1.0     # build
docker images                          # list images
docker run -d -p 9998:9999 hello-python:1.0
```

## From a container

1.  Create a container from the base image:

    ```bash
    docker run -itd --name python-base python:alpine3.17
    ```

2.  Copy the files into the container:

    ```bash
    docker cp app.py python-base:/
    docker cp requirements.txt python-base:/
    ```

3.  Install the dependencies:

    ```bash
    docker exec -it python-base pip install -r requirements.txt
    ```

4.  Create the image from the container:

    ```bash
    docker commit --change='ENTRYPOINT ["python","app.py"]' python-base python-hello:1.0
    ```

5.  Run it:

    ```bash
    docker run -itd -p 9998:9999 python-hello:1.0
    ```
