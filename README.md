Topic4 
=====================
This project consists of a client and server.   
The client captures images and audio from the camera and microphone, sends data with each server, and displays the response results to the user.   
The roles of each server are as follows:   
 - person_info_server: receives video from the client and extracts the name, facial expression, and engagement.   
 - nonperson_info_server: receives an image from the client and extracts the location.   
 - ASR_server: receives audio data and extracts the query.   
 - LLM_server: receives a prompt and generates a response.   

Environment Setup
---------------------------------
Each folder contains a requirements.txt file.   
The client, person_info_server, and ASR_server can operate in the same virtual environment.   
The nonperson_info_server and LLM_server can operate in the same virtual environment.   
The GPU must be at least an RTX 3090.

File structure
---------------------------------
topic4_code   
  ├── topic4_client   
  │   ├── client.py   
  │   ├── data_handling.py  
  │   └── requirements.txt   
  └── topic4_server   
      ├── person_info_server   
      │   ├── person_info_server.py   
      │   ├── requirements.txt   
      │   ├── face.csv   
      │   ├── FD_model   
      │   │   └── scrfd_500m_bnkps.onnx   
      │   ├── FE_model   
      │   │   └── w600k_r50.onnx   
      │   └── VER_model   
      │       └── MobileNetV2_Demo.onnx   
      ├── nonperson_info_server   
      │   ├── place_server.py   
      │   └── requirements.txt   
      ├── ASR_server   
      │   ├── asr_server.py   
      │   └── requirements.txt   
      └── LLM_server   
          ├── LLM_server.py   
          └── requirements.txt   


System Operation
---------------------------------
## IP addresses and port numbers 
The IP addresses and port numbers of the client and server must be matched.  
Each server needs to modify the code at the bottom line.   
uvicorn.run(app, host=IP_num, port=port_num)   

In the client.py, the URI should be modified under the main function.   
vision_uri = "ws://IP_num:port_num/vision_seq" #It should be matched with person_info_server.py.   
LLM_uri = "ws://IP_num:port_num/LLM"   #It should be matched with LLM_server.py.   
place_uri = "ws://IP_num:port_num/place"   #It should be matched with nonperson_info_server.py.   
ASR_uri = "ws://IP_num:port_num/stt"   #It should be matched with asr_server.py.   

In the data_handling.py, the URI should be modified under the main function.   
IP_ADDR = IP_num   
uri = f"ws://{IP_ADDR}:port_num/data"   

## data enrollment
The user's name and face embeddings should be saved in advance.   

First, run person_info_server.py.
```sh
python topic4_server/person_info_server/person_info_server.py
```

Next, run data_handling.py to perform retrieval, addition, and deletion of data.   
```sh
python topic4_client/data_handling.py
```
The user registration process is as follows:   
1. Enter the user's name.   
2. Capture the user's face.   
3. Press the "Esc" key to send the data to the server and register it.   
4. Pressing any key other than "Esc" will prompt the system to return to step 2.   

## Inference
The 4 servers must be started first.  
If there is an issue with the model checkpoint, you can refer to the "model checkpoint download for person info server.txt" to download it again.   

```sh
python topic4_server/person_info_server/person_info_server.py
python topic4_server/nonperson_info_server/place_server.py
python topic4_server/ASR_server/asr_server.py
python topic4_server/LLM_server/LLM_server.py
```

Next, run the client.
```sh
python topic4_client/client.py
```

## Inference process
1. client
 - performing video capture and audio capture.
 - Using VAD (Voice Activity Detection), the spoken segments are extracted, and the corresponding image sequences are extracted.
 - The image sequence is sent to the person info server, the last image is sent to the non-person server, and the audio data is transmitted to the ASR server.
2. each server
 - person info server: from the image sequence, name, emotion, and engagement are extracted and sent to the client.
 - non-person info server: the location is extracted from the image and sent to the client.
 - ASR server: the query is extracted from the audio and sent to the client.
 3. client
 - The client sends the received name, emotion, engagement, and query to the LLM server.
 4. LLM server
 - The LLM server generates a response by inputting a prompt composed of the name, emotion, engagement, and query.
 5. client
 - The received response is then displayed.
