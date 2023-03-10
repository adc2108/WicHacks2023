# WicHacks2023
import cv2
import numpy as np
import mediapipe as mp

# Initialize the video capture object
cap = cv2.VideoCapture(0)

# Initialize the hand tracking module
mp_hands = mp.solutions.hands
hands = mp_hands.Hands(static_image_mode=False, max_num_hands=2, min_detection_confidence=0.7)

# Get the frame size
frame_width = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))
frame_height = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))

# Initialize the drawing canvas and the previous point
canvas = np.zeros((frame_height, frame_width, 3), dtype=np.uint8)
canvas.fill(0)  # Set the canvas to black

prev_x, prev_y = 0, 0

while True:
    # Capture a frame from the video feed
    ret, frame = cap.read()
    
    # Flip the frame horizontally (mirror effect)
    frame = cv2.flip(frame, 1)
    
    # Convert the frame to RGB for hand tracking
    frame_rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
    
    # Detect the hand landmarks in the frame
    results = hands.process(frame_rgb)
    
    # Draw the landmarks and connect them with lines
    if results.multi_hand_landmarks:
        for hand_landmarks in results.multi_hand_landmarks:
            # Check if this is the right hand
            if hand_landmarks.landmark[mp_hands.HandLandmark.WRIST].x < hand_landmarks.landmark[mp_hands.HandLandmark.PINKY_MCP].x:
                finger_tip = hand_landmarks.landmark[mp_hands.HandLandmark.INDEX_FINGER_TIP]
                cx, cy = int(finger_tip.x * frame_width), int(finger_tip.y * frame_height)
                
                # Connect the finger tip with the previous point
                if prev_x != 0 and prev_y != 0:
                    cv2.line(canvas, (prev_x, prev_y), (cx, cy), (0, 0, 255), thickness=5)

                # Update previous point to current point
                prev_x, prev_y = cx, cy
    
    # Combine the frame and the canvas
    frame_with_canvas = cv2.add(frame, canvas)
    
    # Display the resulting image
    cv2.imshow('Webcam', frame_with_canvas)
    
    # Quit the program if 'q' is pressed
    if cv2.waitKey(1) == ord('q'):
        break

# Release the resources
cap.release()
cv2.destroyAllWindows()
