
# Deploy TensorFlow on a web API

## About

This section deals with how to deploy a web service for categorising images, using one or more trained models. 

The _ML Server_ application repo was developed for part 4 of the workshops. It was based on an actual project which was developed and deployed to a company's production environment as a publicly usable service for classifying images.

Running the application starts a Python _REST API_ server, which waits for incoming requests. The server also serves HTML pages, for easy navigation between services and uploading images. 

The source code is available as a Github repo [here](https://github.com/MichaelCurrin/machine-learning-server). 

## How the server starts

When the application starts, the configured model files are loaded into memory, which makes them quick to access later, especially if they are large files. Then application's tree of endpoints is built. The documentation for the endpoint which handles the classification can be found [here](https://github.com/MichaelCurrin/machine-learning-server/blob/master/docs/api.md#plugin-endpoint).

## How a prediction happens

When a prediction is done as a direct request to the API or using the HTML frontend, the following steps happen on the server:

1. Receive a HTTP _POST_ request.
2. Validate input data and raise an error if necessary.
3. Handle image resizing and cropping as configured.
4. Select appropriate TensorFlow model.
5. Run a session and get results ordered with the most likely items first.
6. Format the results as JSON text and return to the client which made the request.


## Setup and usage instructions

1. Follow the instructions in [Deep Learning Course instructions](https://github.com/LeonMVanDyk/deep-learning-course) then run your container in a terminal session.
2. Open another terminal session on your host machine and execute the following commands.

    1. Get the [ML Server repo](https://github.com/MichaelCurrin/machine-learning-server) onto the container. The application is pre-configured and all dependencies have been setup already.
        ```bash
        $ docker exec -it deep-learning-course git clone https://github.com/MichaelCurrin/machine-learning-server.git
        ```
    2. Start the server application. If successful, no output will be printed. At any point, press _CTRL+C_ in this terminal to stop the server application.
        ```bash
        $ docker exec -it deep-learning-course machine-learning-server/mlserver/app.py
        ```

4. Open another terminal session and view activity as it is added to the application's log files. Any access requests to get and send data are recorded in `access.log`, while logged events and errors are recorded in `error.log`.
    ```bash
    $ docker exec -it deep-learning-course sh -c 'tail -F machine-learning-server/mlserver/var/log/app/*.log'
    ```
5. Open the following link in your browser: [http://localhost:9000]().
6. Navigate to the _Built-in Colour Classifier_ page, upload an image, select X and Y co-ordinates and then click Submit.
7. To prettify the output in the browser, you may wish to install _JSON Viewer_ as a [Chrome Extension](https://chrome.google.com/webstore/detail/json-viewer/gbmdgpbipfallnflgajpaliibnhdgobh) or [Firefox Add-On](https://addons.mozilla.org/en-US/firefox/addon/jsonview/) then press back and do the request again.
8. You may want to view the terminal session which has the log files displayed, to see what activity has been logged.

## Drop in your own colour classifier

The server comes with a built-in colour classifier model. It also has as service which is configured to a do a prediction using a developer's _own_ colour model. If you trained a colour classifier in the the [Colour Challenge](https://github.com/jacowp357/dlw-cape-town/blob/master/notebooks/workshop-3/colour-challenge/colour-challenge.ipynb) of workshop 3 and want to use it in the browser, follow these instructions.

1. Create links in the ML Server project to your own model and labels files. These are only path references, so if the underlying files change, then the links will reflect them.
    
    ```bash
    $ docker exec -it deep-learning-course sh -c \
      'ln -s ~/dlw-cape-town/notebooks/workshop-3/colour-challenge/output_graph.pb \
      machine-learning-server/mlserver/models/dropinColorClassifier/modelGraph.local.pb'
    $ docker exec -it deep-learning-course sh -c \
      'ln -s ~/dlw-cape-town/notebooks/workshop-3/colour-challenge/labels.txt \
      machine-learning-server/mlserver/models/dropinColorClassifier/colors.local.txt'
    ```
2. Stop the server.
3. Start the server again.
    ```bash
    $ docker exec -it deep-learning-course machine-learning-server/mlserver/app.py
    ```
4. Go to the _Drop-in Color Predictor_ page at [http://localhost:9000/classify/dropinColors.html]() and upload an image.
5. Optionally, you can retrain your model to get a new model file and labels file, stop and start the service to load the model onto the server, then do another prediction and compare the results with those from before. Or compare them with the built-in classifier's results for the same image and co-ordinates.
