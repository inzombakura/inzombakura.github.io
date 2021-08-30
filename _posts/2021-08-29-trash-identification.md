---
title: Trash Material Identifier
categories:
- Projects
tags:
- ML
- python
- fastai
- CV
- NN
- colab
- pythoneverywhere
- flask
- javascript
---

![Metal can classified](/assets/img/tia.jpg)


Garbage is everywhere. One of the major annoyances we all face when dealing with it is correctly sorting it into it's proper categories and receptacles. A strong goal for the future is to automate detection of material and then sorting of that material through artificial intelligence and robotics. This weekend I worked on implementing my own neural network using fastai to train on images of various categories of garbage and then be able to classify new images as paper, trash, cardboard, metal, glass, and plastic. After training my network on [my google colab](https://colab.research.google.com/drive/13viY0eAHK3gvBmLJn0dBxnr0I2ojFE52?usp=sharing), I stored it as a pickle file then transported it to pythoneverywhere where it would be served on a flask server as a web API to identify images that were sent to it. From there I built a simple frontend [demo](https://inzombakura.github.io/tia.html) so that a user can upload a local image and have it be sent to the api. 

The development process was briefly as follows:

I used a pre labelled [image dataset of garbage](https://github.com/garythung/trashnet/blob/master/data/dataset-resized.zip) which I uploaded to my notebook. Each image is sorted into a folder containing it's label so I rearranged the data into train, validation, and test sets so that I could begin the machine learning process with it.

```python
## move files to destination folders for each waste type
for waste_type in waste_types:
    source_folder = os.path.join('dataset-resized',waste_type)
    train_ind, valid_ind, test_ind = split_indices(source_folder,1,1)
    
    ## move source files to train
    train_names = get_names(waste_type,train_ind)
    train_source_files = [os.path.join(source_folder,name) for name in train_names]
    train_dest = "data/train/"+waste_type
    move_files(train_source_files,train_dest)
    
    ## move source files to valid
    valid_names = get_names(waste_type,valid_ind)
    valid_source_files = [os.path.join(source_folder,name) for name in valid_names]
    valid_dest = "data/valid/"+waste_type
    move_files(valid_source_files,valid_dest)
    
    ## move source files to test
    test_names = get_names(waste_type,test_ind)
    test_source_files = [os.path.join(source_folder,name) for name in test_names]
    ## I use data/test here because the images can be mixed up
    move_files(test_source_files,"data/test")
```


I created a new convolutional neural network learner through the fastai library using the pretrained resnet50 library as the beginning of the network. I would have used a larger network like resnet152 but I needed to keep my network small to fit within pythoneverywhere's limits.

```python
learn = cnn_learner(data,models.resnet50,metrics=error_rate)
```

I searched the most effective learning rate on the first epoch as 5e-03. This is chosen as around the center of the main descent of this graph between loss and learning rate:


![Loss V LR](/assets/img/tia_lr.png)


I ran the network for 40 epochs and eventually reached a 95% accuracy on the validation and then testing set after less than an hour.


```python
learn.fit_one_cycle(40,max_lr=5e-03)
```

At the end, you can see that the confusion matrix was not too confused.


![Confusion matrix](/assets/img/tia_confusion.png)


I exported my learner using `learn.export()` and then uploaded it to my new pythoneverywhere project where I also had my Flask server:

```Python
from flask import Flask
from flask import request
from flask_cors import CORS, cross_origin
from fastai.vision import load_learner
from fastai.vision import open_image

app = Flask(__name__)
model = load_learner('')
cors = CORS(app, resources={r"/": {"origins": "*"}})

app.config['CORS_HEADERS'] = 'Content-Type'


@app.route('/', methods=['GET', 'POST'])
@cross_origin()
def tia():
    if request.method == 'GET':
        return "hello"
    img = request.files['garbage']
    what, _, prob = model.predict(open_image(img))
    return str(what)


if __name__ == '__main__':
    app.run()
```

I allowed all sources to call the API as no special request is ever transported or triggered. Then I called the API using a simple XMLHttpRequest through an HTML form in javascript on my demo:

```javascript
function formSubmit(event) {
        var url = "https://inzombakura.pythonanywhere.com/";
        var request = new XMLHttpRequest();
        request.open('POST', url);
        submitBtn.innerText = "Loading";
        submitBtn.style.cursor = "auto";
        request.onload = function() { // request successful
        // we can use server response to our request now
          if (request.status === 200) {
            labelText.innerText = request.responseText;
          }
          submitBtn.innerText = "Submit";
          submitBtn.style.cursor = "pointer";
        };

        request.onerror = function() {
          labelText.innerText = "Unidentified";
          submitBtn.innerText = "Submit";
          submitBtn.style.cursor = "pointer";
        };

        request.send(new FormData(event.target)); // create FormData from form that triggered event
        event.preventDefault();
}
```

Again you try out my demo and inspect the rest of the code for yourself [here](https://inzombakura.github.io/tia.html).
