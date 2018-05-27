
# Deploy TensorFlow on a web API

## About

This section deals with how to deploy a web service for categorising images, using one or more trained models. 

The _ML Server_ application repo was developed for part 4 of the workshops. It was based on an actual project which was developed and deployed to a company's production environment as a publicly usable service for classifying images.

Running the application starts a Python _REST API_ server, which waits for incoming requests. The server also serves HTML pages, for easy navigation between services and uploading images. 

The source code is available as a Github repo [here](https://github.com/MichaelCurrin/machine-learning-server). 

## How the server starts

When the application starts, the configured model files are loaded into memory, which makes them quick to access later, especially if they are large files. Then the application's tree of endpoints is built and added on the server's path. The documentation for the endpoint which handles the classification can be found [here](https://github.com/MichaelCurrin/machine-learning-server/blob/master/docs/api.md#plugin-endpoint).

## How a prediction happens

When a prediction is done as a direct request to the API or using the HTML frontend, the following steps happen on the server:

1. Receive an [HTTP _POST_](https://en.wikipedia.org/wiki/POST_(HTTP)) request.
2. Validate input data and raise an error if necessary.
3. Handle image resizing and cropping as configured.
4. Select appropriate TensorFlow model.
5. Run a session and get results ordered with the most likely items first.
6. Format the results as JSON text and return to the client which made the request.


## Setup and usage instructions

1. Get the _ML Server_ sample images onto your local machine. Download a zip [here](https://github.com/MichaelCurrin/machine-learning-server/raw/master/mlserver/sampleImages/digits_and_photos.zip) and extract the contents using your preferred method.
2. To optionally prettify the JSON output in the browser, you can install _JSON Viewer_ as a [Chrome Extension](https://chrome.google.com/webstore/detail/json-viewer/gbmdgpbipfallnflgajpaliibnhdgobh) or [Firefox Add-On](https://addons.mozilla.org/en-US/firefox/addon/jsonview/).
3. Follow the instructions in [Deep Learning Course instructions](https://github.com/LeonMVanDyk/deep-learning-course) then run your container in a terminal session.
4. Open another terminal session on your host machine and execute the following commands.

    1. Get the [ML Server repo](https://github.com/MichaelCurrin/machine-learning-server) onto the container. The application is pre-configured and all dependencies have been setup already.
        ```bash
        $ docker exec -it deep-learning-course git clone https://github.com/MichaelCurrin/machine-learning-server.git
        ```
    2. Start the server application. If successful, no output will be printed. 
        ```bash
        $ docker exec -it deep-learning-course machine-learning-server/mlserver/app.py
        ```

5. Open another terminal session and view activity as it is added to the application's log files. Any access requests to get and send data are recorded in `access.log`, while logged events and errors are recorded in `error.log`.
    ```bash
    $ docker exec -it deep-learning-course sh -c 'tail -F machine-learning-server/mlserver/var/log/app/*.log'
    ```
6. Open the following link in your browser: [http://localhost:9000]().
7. Do a **colour prediction** using the supplied colour model.

    1. Navigate to the _[Built-in Colour Classifier](http://localhost:9000/classify/builtinColors.html)_ page. 
    2. Upload an image, possibly one from the _photos_ directory extracted from the zip file earlier. 
    3. Choose a point on the image then set the X and Y co-ordinates using the sliders. (In practice you would click on the image itself, but to keep the HTML simple the input is limited to sliders.)
    4. Click _Submit_ and view the predicted results (only results with at least 5% accuracy are shown).
    5. Press the back button and upload a different image or choose a different point.

9. Do a **digit prediction** using the supplied digit model. Repeat the above steps, except use the _[Built-in Digit Classifier](http://localhost:9000/classify/builtinDigit.html)_ page instead and upload some images from the _digits_ directory extracted from the zip file.
10. Open your the terminal session which shows the log output. See what has been logged since starting the application.

## Drop in your own colour classifier

The server comes with a built-in colour classifier model which is accessed using a certain HTML page and API endpoint. It also has as service which is configured to a do a prediction using a developer's _own_ colour model. So if you hav trained a colour classifier in the [Colour Challenge](https://github.com/jacowp357/dlw-cape-town/blob/master/notebooks/workshop-3/colour-challenge/colour-challenge.ipynb) of workshop 3 and want to use try it out in the browser, follow these instructions.

1. Stop the server by pressing _CTRL+C_ in this terminal window where `app.py` is running.

2. Create symbolic links in the ML Server project to point to your own model and labels files in the _dlw-cape-town_ project.
    
    ```bash
    $ docker exec -it deep-learning-course sh -c \
      'ln -fs ~/source/dlw-cape-town/notebooks/workshop-3/colour-challenge/output_graph.pb \
      machine-learning-server/mlserver/models/dropinColorClassifier/modelGraph.local.pb'
    $ docker exec -it deep-learning-course sh -c \
      'ln -fs ~/source/dlw-cape-town/notebooks/workshop-3/colour-challenge/labels.txt \
      machine-learning-server/mlserver/models/dropinColorClassifier/colors.local.txt'
    ```

3. Start the server again.

    ```bash
    $ docker exec -it deep-learning-course machine-learning-server/mlserver/app.py
    ```

4. Go to the _Drop-in Color Predictor_ page at [http://localhost:9000/classify/dropinColors.html]() and upload an image.
5. Optionally, you can retrain your model to get a new model file and labels file, stop and start the service to load the model onto the server, then do another prediction and compare the results with those from before. Or compare them with the built-in classifier's results for the same image and co-ordinates.

## Command-line predictions

For interest, you may wish to use the terminal to do an image prediction. These methods were used in development for the HTML frontend was created.


```bash
$ docker exec -it deep-learning-course bash
```

Execute the commands within the Docker container bash terminal opened above.


### Plugin

Each model on the server uses a plugin, which handles loading of the model and labels files and doing a prediction. Plugins have been setup with a python command-line interface, which allows testing a model prediction without the server.

```bash
$ cd machine-learning-server/mlserver
$ # See available photos.
$ ls ../sampleImages/photos
pexels-architecture.jpg  pexels-multi-colour.jpeg     pexels-stack-of-books.jpg
pexels-gears.jpeg        pexels-rose-gold-clips.jpeg  pexels-taxi.jpeg
$ python -m lib.plugins.colourClassifier --help
$ # Do a prediction on the  using for a particular image using X and Y 
# values each set from 0 to 100.
$ python -m lib.plugins.colorClassifier ../sampleImages/photos/pexels-architecture.jpg 10 50
[
    {
        "label": "orange",
        "score": "54.71%"
    },
    {
        "label": "red",
        "score": "42.85%"
    }
]
```

### Web request

Do a POST request to an API endpoint using cURL.

Ensure the server is running in one terminal tab.

```bash
$ docker exec -it deep-learning-course machine-learning-server/mlserver/app.py
```

Provide the path to the image and cURL will send the image on the body of the request for you.

```bash
$ curl localhost:9000/services/classify/builtinColors \ 
-F 'imageFile=@../sampleImages/photos/pexels-architecture.jpg' \
-F x=10 -F y=50
[{"label": "orange", "score": "54.71%"}, {"label": "red", "score": "42.85%"}]
```
