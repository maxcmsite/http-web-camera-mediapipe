# http-web-camera-mediapipe
_main.py__________________________________________________________________________________________________________
| from kivy.app import App                                                                                   |\n
| from kivy.core.window import Window                                                                        |
| from kivy.uix.label import Label                                                                           |
| from kivy.uix.button import Button                                                                         |
| from kivy.uix.textinput import TextInput                                                                   |
| from kivy.uix.floatlayout import FloatLayout                                                               |
| def server():                                                                                              |
|     from flask import Flask                                                                                |
|     app = Flask(__name__,template_folder='')                                                               |
|     def ii():                                                                                              |
|         global imge;                                                                                       |
|         import cv2                                                                                         |
|         import mediapipe as mp                                                                             |
|         mp_drawing = mp.solutions.drawing_utils                                                            |
|         mp_drawing_styles = mp.solutions.drawing_styles                                                    |
|         mp_hands = mp.solutions.hands                                                                      |
|         # For static images:                                                                               |
|         IMAGE_FILES = []                                                                                   |
|         with mp_hands.Hands(                                                                               |
|             static_image_mode=True,                                                                        |
|             max_num_hands=2,                                                                               |
|         min_detection_confidence=0.5) as hands:                                                            |
|           for idx, file in enumerate(IMAGE_FILES):                                                         |
|             # Read an image, flip it around y-axis for correct handedness output (see                      |
|             # above).                                                                                      |
|             image = cv2.flip(cv2.imread(file), 1)                                                          |
|             # Convert the BGR image to RGB before processing.                                              |
|             results = hands.process(cv2.cvtColor(image, cv2.COLOR_BGR2RGB))                                |
|                                                                                                            |
|             # Print handedness and draw hand landmarks on the image.                                       |
|             print('Handedness:', results.multi_handedness)                                                 |
|             if not results.multi_hand_landmarks:                                                           |
|                 continue                                                                                   |
|             image_height, image_width, _ = image.shape                                                     |
|             annotated_image = image.copy()                                                                 |
|             for hand_landmarks in results.multi_hand_landmarks:                                            |
|                 print('hand_landmarks:', hand_landmarks)                                                   |
|                 print(                                                                                     |
|                     f'Index finger tip coordinates: (',                                                    |
|                     f'{hand_landmarks.landmark[mp_hands.HandLandmark.INDEX_FINGER_TIP].x * image_width}, ' |
|                     f'{hand_landmarks.landmark[mp_hands.HandLandmark.INDEX_FINGER_TIP].y * image_height})' |
|                 )                                                                                          |
|                 mp_drawing.draw_landmarks(                                                                 |
|                     annotated_image,                                                                       |
|                     hand_landmarks,                                                                        |
|                     mp_hands.HAND_CONNECTIONS,                                                             |
|                     mp_drawing_styles.get_default_hand_landmarks_style(),                                  |
|                     mp_drawing_styles.get_default_hand_connections_style())                                |
|             cv2.imwrite(                                                                                   |
|                 '/tmp/annotated_image' + str(idx) + '.png', cv2.flip(annotated_image, 1))                  |
|             # Draw hand world landmarks.                                                                   |
|             if not results.multi_hand_world_landmarks:                                                     |
|                 continue                                                                                   |
|             for hand_world_landmarks in results.multi_hand_world_landmarks:                                |
|                 mp_drawing.plot_landmarks(                                                                 |
|                     hand_world_landmarks, mp_hands.HAND_CONNECTIONS, azimuth=5)                            |
|                                                                                                            |
|         # For webcam input:                                                                                |
|         cap = cv2.VideoCapture(0)                                                                          |
|         with mp_hands.Hands(                                                                               |
|             model_complexity=0,                                                                            |
|             min_detection_confidence=0.5,                                                                  |
|             min_tracking_confidence=0.5) as hands:                                                         |
|           while cap.isOpened():                                                                            |
|             success, image = cap.read()                                                                    |
|             if not success:                                                                                |
|                 print("Ignoring empty camera frame.")                                                      |
|                 # If loading a video, use 'break' instead of 'continue'.                                   |
|                 continue                                                                                   |
|                                                                                                            |
|             # To improve performance, optionally mark the image as not writeable to                        |
|             # pass by reference.                                                                           |
|             image.flags.writeable = False                                                                  |
|             image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)                                                 |
|             results = hands.process(image)                                                                 |
|                                                                                                            |
|             # Draw the hand annotations on the image.                                                      |
|             image.flags.writeable = True                                                                   |
|             image = cv2.cvtColor(image, cv2.COLOR_RGB2BGR)                                                 |
|             if results.multi_hand_landmarks:                                                               |
|                 for hand_landmarks in results.multi_hand_landmarks:                                        |
|                     mp_drawing.draw_landmarks(                                                             |
|                         image,                                                                             |
|                         hand_landmarks,                                                                    |
|                         mp_hands.HAND_CONNECTIONS,                                                         |
|                         mp_drawing_styles.get_default_hand_landmarks_style(),                              |
|                         mp_drawing_styles.get_default_hand_connections_style())                            |
|             # Flip the image horizontally for a selfie-view display.                                       |
|             imge=image                                                                                     |
|             #cv2.imshow('MediaPipe Hands', cv2.flip(image, 1))                                             |
|             if cv2.waitKey(5) & 0xFF == 27:                                                                |
|                 break                                                                                      |
|         cap.release()                                                                                      |
|     from threading import Thread                                                                           |
|     Thread(target=ii,daemon=True).start()                                                                  |
|     def video_stream():                                                                                    |
|         while True:                                                                                        |
|             global imge;                                                                                   |
|             import cv2                                                                                     |
|             ret,buffer=cv2.imencode('.jpeg',imge)                                                          |
|             frame = buffer.tobytes()                                                                       |
|            yield (b'--frame\r\n' b'Content-type: image/jpeg\r\n\r\n' + frame + b'\r\n')                    |
|     @app.route('/')                                                                                        |
|     def siteTest():                                                                                        |
|         from flask import render_template                                                                  |
|         return render_template('index.html')                                                               |
|     @app.route('/video_feed')                                                                              |
|     def video_feed():                                                                                      |
|         from flask import Response                                                                         |
|         return Response(video_stream(), mimetype= 'multipart/x-mixed-replace; boundary=frame')             |
|     if __name__=='__main__':                                                                               |
|         from threading import Thread                                                                       |
|         global ports;                                                                                      |
|         Thread(target=app.run(host ='0.0.0.0', port= ports, debug=False),daemon=True).start()              |
| class main(App):                                                                                           |
|     def build(self):                                                                                       |
|         fl1=FloatLayout()                                                                                  |
|         btn1=Button(text='start',size_hint=(.1,.05),pos_hint={"center_x":.5,"center_y":.85})               |
|         lb1=Label(text='',size_hint=(.1,.05),pos_hint={"center_x":.5,"center_y":.75})                      |
|         ti1=TextInput(text='8080',size_hint=(.1,.05),pos_hint={"center_x":.5,"center_y":.95})              |
|         fl1.add_widget(ti1)                                                                                |
|         fl1.add_widget(lb1)                                                                                |
|         fl1.add_widget(btn1)                                                                               |
|         def start(self):                                                                                   |
|             global ports;                                                                                  |
|             ports=int(ti1.text)                                                                            |
|             from threading import Thread                                                                   |
|             Thread(target=server,daemon=True).start()                                                      |
|             btn1.disabled=True                                                                             |
|             ti1.disabled=True                                                                              |
|             from socket import gethostbyname,gethostname                                                   |
|             lb1.text=gethostbyname(gethostname())+':'+str(ports)                                           |
|         btn1.bind(on_press=start)                                                                          |
|         return fl1                                                                                         |
| if __name__=='__main__':                                                                                   |
|     main().run()                                                                                           |
￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣
