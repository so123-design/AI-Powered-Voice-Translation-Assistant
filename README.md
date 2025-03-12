# **AI-Powered Voice Translation Assistant**

## **Project Overview**

In this project, we develop a **voice translation assistant** using **Watsonx and IBM Watson Speech Libraries**. This assistant allows users to speak in one language, transcribe the speech into text using **IBM Watson Speech-to-Text**, translate the text using **Watsonx’s FLAN-UL2 model**, convert it into speech using **IBM Watson Text-to-Speech**, and play it back in the translated language. The system is designed with a **Flask backend** and a **responsive frontend** built using **HTML, CSS, and JavaScript**.

This project demonstrates the integration of multiple **IBM Watson AI services** to provide seamless voice translation. It can be particularly useful for **real-time language translation applications**, aiding communication across different languages.

---

## **Introduction**

With globalization, language barriers have become a significant challenge. Manual translation is time-consuming, and real-time conversations require instant translations. This project automates the entire process of **speech recognition, translation, and voice synthesis**, providing users with a fast and accurate translation experience.

<p align="center">
  <img src="https://github.com/so123-design/AI-Powered-Voice-Translation-Assistant/blob/90ca8dbf8093abd65d2744f92488c97cefee54ae/GUI%20voice%20translator%20picture%202.PNG " alt="My Image" width="800">
</p>



The **main functionalities** of this application include:
1. **Speech-to-Text**: Converting spoken words into text using **IBM Watson Speech-to-Text**.
2. **Language Translation**: Using **Watsonx FLAN-UL2** to translate text to another language.
3. **Text-to-Speech**: Converting translated text into spoken words using **IBM Watson Text-to-Speech**.
4. **User-Friendly Interface**: Providing a simple and intuitive web interface for seamless interaction.

---

## **Technology Stack**

### **Backend**:
- **Flask**: A lightweight Python web framework for handling HTTP requests.
- **IBM Watson Speech-to-Text**: For converting speech to text.
- **Watsonx FLAN-UL2**: For language translation.
- **IBM Watson Text-to-Speech**: For converting text to speech.

### **Frontend**:
- **HTML, CSS, JavaScript**: For building a responsive user interface.
- **Flask CORS**: For enabling cross-origin requests.

### **Dependencies**:
To run this project, install the required dependencies using:
```bash
pip install flask flask-cors requests ibm-watson-machine-learning
```

---

## **Backend Implementation**

### **Server Code (server.py)**
This file sets up a **Flask server** that handles speech recognition, text translation, and speech synthesis.

```python
import base64
import json
from flask import Flask, render_template, request
from flask_cors import CORS
import os
from worker import speech_to_text, text_to_speech, watsonx_process_message

app = Flask(__name__)
cors = CORS(app, resources={r"/*": {"origins": "*"}})

@app.route('/', methods=['GET'])
def index():
    return render_template('index.html')

@app.route('/speech-to-text', methods=['POST'])
def speech_to_text_route():
    print("Processing Speech-to-Text")
    audio_binary = request.data
    text = speech_to_text(audio_binary)
    response = app.response_class(
        response=json.dumps({'text': text}),
        status=200,
        mimetype='application/json'
    )
    return response

@app.route('/process-message', methods=['POST'])
def process_message_route():
    user_message = request.json['userMessage']
    voice = request.json['voice']
    watsonx_response_text = watsonx_process_message(user_message)
    watsonx_response_speech = text_to_speech(watsonx_response_text, voice)
    watsonx_response_speech = base64.b64encode(watsonx_response_speech).decode('utf-8')
    response = app.response_class(
        response=json.dumps({"watsonxResponseText": watsonx_response_text, "watsonxResponseSpeech": watsonx_response_speech}),
        status=200,
        mimetype='application/json'
    )
    return response

if __name__ == "__main__":
    app.run(port=8000, host='0.0.0.0')
```

### **Worker Code (worker.py)**
This file contains functions for **speech-to-text, text translation, and text-to-speech** using IBM Watson services.

```python
from ibm_watson_machine_learning.foundation_models.utils.enums import ModelTypes
from ibm_watson_machine_learning.foundation_models import Model
import requests

PROJECT_ID = "skills-network"

credentials = {
    "url": "https://us-south.ml.cloud.ibm.com"
}
    
model_id = ModelTypes.FLAN_UL2

from ibm_watson_machine_learning.metanames import GenTextParamsMetaNames as GenParams
from ibm_watson_machine_learning.foundation_models.utils.enums import DecodingMethods

parameters = {
    GenParams.DECODING_METHOD: DecodingMethods.GREEDY,
    GenParams.MIN_NEW_TOKENS: 1,
    GenParams.MAX_NEW_TOKENS: 1024
}

model = Model(
    model_id=model_id,
    params=parameters,
    credentials=credentials,
    project_id=PROJECT_ID
)

def speech_to_text(audio_binary):
    api_url = 'https://sn-watson-stt.labs.skills.network/speech-to-text/api/v1/recognize'
    params = {'model': 'en-US_Multimedia'}
    response = requests.post(api_url, params=params, data=audio_binary).json()
    text = response.get('results', [{}])[-1].get('alternatives', [{}])[-1].get('transcript', 'null')
    return text

def text_to_speech(text, voice=""):
    api_url = 'https://sn-watson-tts.labs.skills.network/text-to-speech/api/v1/synthesize?output=output_text.wav'
    if voice:
        api_url += "&voice=" + voice
    headers = {'Accept': 'audio/wav', 'Content-Type': 'application/json'}
    json_data = {'text': text}
    response = requests.post(api_url, headers=headers, json=json_data)
    return response.content

def watsonx_process_message(user_message):
    prompt = f"""You are an assistant translating English to Spanish.
    Translate: ```{user_message}```."""
    response_text = model.generate_text(prompt=prompt)
    return response_text
```

---

## **Frontend Implementation**
The frontend consists of **HTML, CSS, and JavaScript**, allowing users to record their voice, receive translations, and play back the translated speech.

### **Features**:
- **Record and Upload Audio**
- **Display Transcriptions and Translations**
- **Play Translated Speech**

The frontend interacts with the Flask backend using **AJAX requests** to send and receive translation data.

---

## **Conclusion**
This project showcases how **AI-powered speech processing** can bridge language barriers. By integrating **IBM Watson Speech-to-Text, Watsonx FLAN-UL2, and IBM Watson Text-to-Speech**, we created an effective voice translation assistant. The project highlights **speech recognition, AI-based translation, and text-to-speech conversion** in a user-friendly format.


