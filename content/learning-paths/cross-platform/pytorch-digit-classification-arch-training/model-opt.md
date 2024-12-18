---
# User change
title: "Create a PyTorch model for MNIST"

weight: 12

layout: "learningpathall"
---

Now, similarly as before, you will create and train a feedforward neural network to classify handwritten digits from the MNIST dataset. To remind this dataset contains 70,000 images, comprising 60,000 training and 10,000 testing images, of handwritten numerals (0-9), each with dimensions of 28x28 pixels. 

You will, however, introduce several changes to enable model quantization and fusing.

# Model architecture
Start by creating a new notebook, pytorch-digits-model-optimisations.ipynb. Then define the model architecture as follows (you can find the complete source code [here](https://github.com/dawidborycki/Arm.PyTorch.MNIST.Inference.Python))

```Python
import torch
from torch import nn
from torchsummary import summary

class_names = range(10)

class NeuralNetwork(nn.Module):
    def __init__(self, use_dropout=True):
        super(NeuralNetwork, self).__init__()
        self.use_dropout = use_dropout
        self.flatten = nn.Flatten()
        
        self.linear1 = nn.Linear(28*28, 96)
        self.relu1 = nn.ReLU()
        self.dropout1 = nn.Dropout(0.2) if use_dropout else nn.Identity()
        
        self.linear2 = nn.Linear(96, 256)
        self.relu2 = nn.ReLU()
        self.dropout2 = nn.Dropout(0.2) if use_dropout else nn.Identity()
        
        self.linear3 = nn.Linear(256, len(class_names))
        # Softmax is removed from the model

    def forward(self, x):
        x = self.flatten(x)
        x = self.linear1(x)
        x = self.relu1(x)
        x = self.dropout1(x)
        x = self.linear2(x)
        x = self.relu2(x)
        x = self.dropout2(x)
        x = self.linear3(x)
        return x  # Outputs raw logits
```

This code defines a neural network in PyTorch for digit classification, consisting of three linear layers with ReLU activations and optional dropout layers for regularization. The network first flattens the input (a 28x28 image) and passes it through two linear layers, each followed by a ReLU activation and a dropout layer (if enabled). The final layer produces raw logits as the output. Notably, the **Softmax layer has been removed** to enable **quantization and layer fusion** during model optimization, allowing better performance when deploying the model on mobile or edge devices. The output is left as logits, and the softmax function can be applied during post-processing, particularly during inference.

This model includes **dropout layers**, which are used during training to randomly set a portion of the neurons to zero in order to prevent overfitting and improve generalization. The `use_dropout` parameter allows you to **enable or disable dropout**, with the option to bypass dropout by replacing it with an `nn.Identity` layer when set to `False`, which is typically done during inference or quantization for more consistent behavior.

Then, add the following lines to display the model arhitecture:

```Python
model = NeuralNetwork()

summary(model, (1, 28, 28))
```

After running the code, you will see the following output:
```output
----------------------------------------------------------------
        Layer (type)               Output Shape         Param #
================================================================
           Flatten-1                  [-1, 784]               0
            Linear-2                   [-1, 96]          75,360
              ReLU-3                   [-1, 96]               0
           Dropout-4                   [-1, 96]               0
            Linear-5                  [-1, 256]          24,832
              ReLU-6                  [-1, 256]               0
           Dropout-7                  [-1, 256]               0
            Linear-8                   [-1, 10]           2,570
================================================================
Total params: 102,762
Trainable params: 102,762
Non-trainable params: 0
----------------------------------------------------------------
Input size (MB): 0.00
Forward/backward pass size (MB): 0.01
Params size (MB): 0.39
Estimated Total Size (MB): 0.41
----------------------------------------------------------------
```

The output shows the structure of the neural network, including the layers, their output shapes, and the number of parameters.
* The network starts with a Flatten layer, which reshapes the input from [1, 28, 28] to [1, 784] without adding any parameters.
* This is followed by two Linear (fully connected) layers with ReLU activations and optional Dropout layers in between, contributing to the parameter count.
* The first linear layer (from 784 to 96 units) has 75,360 parameters, while the second (from 96 to 256 units) has 24,832 parameters.
* The final linear layer, which outputs raw logits for the 10 classes, has 2,570 parameters.
* The total number of trainable parameters in the model is 102,762, with no non-trainable parameters.

# Training the model
Now add the data loading, train, and test loops to actually train the model (this proceeds exactly the same as in the original model):

```
from torchvision import transforms, datasets
from torch.utils.data import DataLoader

# Training data 
training_data = datasets.MNIST(
    root="data",
    train=True,
    download=True,
    transform=transforms.ToTensor()
)

# Test data
test_data = datasets.MNIST(
    root="data",
    train=False,
    download=True,
    transform=transforms.ToTensor()
)

# Dataloaders
batch_size = 32

train_dataloader = DataLoader(training_data, batch_size=batch_size)
test_dataloader = DataLoader(test_data, batch_size=batch_size)

learning_rate = 1e-3;

loss_fn = nn.CrossEntropyLoss()
optimizer = torch.optim.Adam(model.parameters(), lr=learning_rate)

def train_loop(dataloader, model, loss_fn, optimizer):
    size = len(dataloader.dataset)
    for batch, (x, y) in enumerate(dataloader):
        # Compute prediction and loss
        pred = model(x)
        loss = loss_fn(pred, y)

        # Backpropagation
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()


def test_loop(dataloader, model, loss_fn):
    size = len(dataloader.dataset)
    num_batches = len(dataloader)
    test_loss, correct = 0, 0

    with torch.no_grad():
        for x, y in dataloader:
            pred = model(x)
            test_loss += loss_fn(pred, y).item()
            correct += (pred.argmax(1) == y).type(torch.float).sum().item()

    test_loss /= num_batches
    correct /= size
    
    print(f"Accuracy: {(100*correct):>0.1f}%, Avg loss: {test_loss:>8f} \n")

epochs = 10

for t in range(epochs):
    print(f"Epoch {t+1}:")
    train_loop(train_dataloader, model, loss_fn, optimizer)
    test_loop(test_dataloader, model, loss_fn)
```

In this script, we begin by preparing the MNIST dataset for training and testing our neural network model. Using the torchvision library, we download the MNIST dataset and apply a transformation to convert the images into tensors, making them suitable for input into the model. We create two data loaders: one for the training set and one for the test set, each configured with a batch size of 32. These data loaders allow us to easily feed batches of images into the model during training and testing.

Next, we define a training loop, which is the core of the model’s learning process. For each batch of images and labels, the model generates predictions, and we calculate the cross-entropy loss to measure how far off the predictions are from the true labels. The Adam optimizer is then used to perform backpropagation, updating the model’s weights to reduce this error. This process repeats for every batch in the training dataset, gradually improving the model’s accuracy over time.

To ensure our model is learning effectively, we also define a testing loop. Here, the model is evaluated on a separate set of test images that it hasn’t seen during training. We calculate both the average loss and the accuracy of the predictions, giving us a clear sense of how well the model is performing. Importantly, this evaluation is done without updating the model’s weights, as the goal is simply to measure its performance.

Finally, we run the training and testing loops over the course of 10 epochs. With each epoch, the model trains on the full training dataset, and afterward, we test it to monitor its progress. By the end of the process, we expect the model to have learned to classify the MNIST digits with a high degree of accuracy, as reflected in the final test results.

This setup efficiently trains and evaluates the model for digit classification, providing feedback after each epoch on accuracy and loss.

After running the code you will see the following output:

```output
Epoch 1:
Accuracy: 94.0%, Avg loss: 0.196315 

Epoch 2:
Accuracy: 95.3%, Avg loss: 0.155560 

Epoch 3:
Accuracy: 95.9%, Avg loss: 0.138764 

Epoch 4:
Accuracy: 95.4%, Avg loss: 0.156163 

Epoch 5:
Accuracy: 95.5%, Avg loss: 0.163152 

Epoch 6:
Accuracy: 96.3%, Avg loss: 0.129509 

Epoch 7:
Accuracy: 96.8%, Avg loss: 0.124872 

Epoch 8:
Accuracy: 96.6%, Avg loss: 0.127252 

Epoch 9:
Accuracy: 96.4%, Avg loss: 0.134298 

Epoch 10:
Accuracy: 96.5%, Avg loss: 0.137004 
```

The above shows a similar accuracy as the original model.

You now have the trained model with the modified architecture. In the next step you will optimise it for mobile inference.