---
title: FastAPI调用MLP模型
date: 2024-09-05 10:38:21
tags:
  - FastAPI
  - MLP
  - Tensorflow
  - Python
---


## 前言
在实现[仔猪腹泻管理系统](http://pig.foolcoder.com/)(Piglet Diarrhea Detection System, 简称PDDS)时，为了实现系统内部可以直接智能检测病原类型，因此尝试过多种模型嵌入后端的方法。最终选取FastAPI来实现该功能。

## FastAPI程序
将Tensorflow模型的Predict代码改为FastAPI程序的调用代码。
```python
from fastapi import FastAPI, UploadFile, File
from pydantic import BaseModel
import cv2
import numpy as np
import tensorflow as tf

# Load the pre-trained TensorFlow model
model = tf.keras.models.load_model('./model')

# Define the class labels for the model predictions
class_labels = ["health", "Ecoli", "rotavirus", "coronavirus"]

# Define the target size for image resizing
img_height = 224
img_width = 224

# Initialize the FastAPI application
app = FastAPI()


# Define the prediction response model
class Prediction(BaseModel):
    filename: str
    prediction: str


# Define the prediction endpoint
@app.post("/predict", response_model=Prediction)
async def predict(file: UploadFile = File(...)):
    # Read the uploaded image file
    contents = await file.read()
    # Convert the image file to a NumPy array
    np_img = np.frombuffer(contents, np.uint8)
    # Decode the image from the NumPy array
    img = cv2.imdecode(np_img, cv2.IMREAD_COLOR)
    # Resize the image to the target size
    img = cv2.resize(img, (img_height, img_width))
    # Convert the image to a NumPy array
    img_array = np.array(img)
    # Ensure the image array is of type uint8
    img_array = img_array.astype(np.uint8)
    # Expand dimensions to match the model's input shape
    img_array = np.expand_dims(img_array, axis=0)

    # Make a prediction using the model
    predictions = model.predict(img_array)

    # Get the index of the highest probability class
    predicted_class_index = np.argmax(predictions[0])
    # Map the index to the corresponding class label
    predicted_class = class_labels[predicted_class_index]

    # Return the prediction result
    return Prediction(filename=file.filename, prediction=predicted_class)


# Run the FastAPI application
if __name__ == "__main__":
    import uvicorn

    # Start the Uvicorn server to serve the FastAPI application
    uvicorn.run(app='main:app', host="127.0.0.1", port=8000)

```
然后在终端运行如下指令：
```shell
uvicorn main:app --reload
```
即可启动

## Request测试代码
提供一个request.py的demo代码
```python
import requests

# Define the URL of the FastAPI prediction endpoint
url = "http://127.0.0.1:8000/predict"

# Path to the image file you want to upload
image_path = "./img/111.jpg"

# Open the image file in binary mode
with open(image_path, "rb") as image_file:
    # Prepare the files dictionary with the image file
    files = {"file": image_file}

    # Send a POST request to the FastAPI endpoint
    response = requests.post(url, files=files)

    # Print the response from the server
    print(response.json())

```
在运行该代码之前，要将主程序跑起来。

## Java后端调用代码
API主要目的为JAVA后端可以调用，因此提供一个小demo。我们使用OkHttp库来实现该操作。
首先添加依赖：
```java
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>4.9.3</version>
</dependency>
```
JAVA代码
```java
import okhttp3.*;
import org.json.JSONArray;
import org.json.JSONObject;

import java.io.File;
import java.io.IOException;
import java.util.concurrent.TimeUnit;

public class FastApiClient {
    public static void main(String[] args) throws IOException {
        OkHttpClient client = new OkHttpClient.Builder()
                .connectTimeout(30, TimeUnit.SECONDS)
                .writeTimeout(30, TimeUnit.SECONDS)
                .readTimeout(30, TimeUnit.SECONDS)
                .build();

        String url = "http://localhost:8000/predict";

        MultipartBody.Builder multipartBuilder = new MultipartBody.Builder().setType(MultipartBody.FORM);

        File folder = new File("G:/mlp/test_data/coronavirus");
        File[] listOfFiles = folder.listFiles();

        if (listOfFiles != null) {
            for (File file : listOfFiles) {
                if (file.isFile()) {
                    multipartBuilder.addFormDataPart("files", file.getName(),
                            RequestBody.create(file, MediaType.parse("image/jpeg")));
                }
            }
        }

        RequestBody requestBody = multipartBuilder.build();

        Request request = new Request.Builder()
                .url(url)
                .post(requestBody)
                .build();

        try (Response response = client.newCall(request).execute()) {
            if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

            String responseBody = response.body().string();
            JSONArray jsonArray = new JSONArray(responseBody);

            for (int i = 0; i < jsonArray.length(); i++) {
                JSONObject jsonObject = jsonArray.getJSONObject(i);
                String filename = jsonObject.getString("filename");
                String prediction = jsonObject.getString("prediction");
                System.out.println("Image " + filename + " is predicted as: " + prediction);
            }
        }
    }
}
```