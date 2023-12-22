We are seeking an experienced developer or team to create a cutting-edge face recognition project that operates seamlessly with live streams from web cameras and IP cameras. The project will be a Python web application with a user-friendly frontend, employing frameworks such as Django, or React in combination with FastAPI/Flask.

Key Requirements:

- it should be able to receive live streams from web cameras and ip cameras as well
- it should be a python web app with frontend (for example django, or react + fastapi / flask)
- implement milvus (https://milvus.io/) for indexing and querying the face vectors
- the frontend should have a page where it should show the list of connected streams
- there should be stream details page, that should be open by clicking one of the stream list
- the stream details page (low-fidelity design is attached as an image) should have:
a) a video player that shows the streaming video in real time from web-camera or ip-camera (so, choose a streaming protocol which is suitable in such cases)
b) during the stream, face recognition should be performed:
I) detect faces (it should be multi-face detection)
II) for each face detected, get vectors
III) perform opensearch query to find the most similar faces (get top 3)
IV) once the similar faces are found, then return back to the frontend
V) frontend then should draw the results in the stream details page. There should also be a separate layout next to live-video-play, to show the results that show the detected face (image), and similar faces (along with their full name)

- there should also be face management page:
- it should show the list of images in the DB
- it should have a `create` button, when pressed, open a page where it shows inputs (name and image, where face will be detected and inserted into the DB)

The real-time drawing right into the video-player might be challenging (since face detecting / vectorizing / querying from db might take some time), so we can take one of the actions:
- implement face-recognition only when a face in video-stream is somehow stabilized (i.e. coordinates do not differ much)
- implement face-recognition every 2 (or 3) seconds and do not draw on video, rather just show on the separate results layout


Other requirements:
- use arcface for face recognition, no exact model for face detection, but it should be fast and accurate enough
- We should be able to stream from streaming tools, such as obs-studio.
- we should be able to support at least 15-20 fps for streaming
- we should be able to receive multiple streams at the same time
- dockerize the project components

This is the first iteration, and we might continue with other iterations if the above requirements are met.

Project Timeline:
The deadline for the project is strict, with the expectation of completion by the 3rd of December.

How to Apply:
Interested candidates or teams should provide a detailed proposal outlining their relevant experience, approach to meeting the project requirements, and a realistic timeline for completion. Include examples of past projects that demonstrate proficiency in real-time video processing and face recognition.

We look forward to reviewing your proposals and working with a skilled developer or team to bring this exciting project to fruition.