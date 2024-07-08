# PS 13: Vehicle Movement Analysis and Insight Generation in a College Campus using Edge AI

# Unique Idea Brief

### Vehicle Matching

1. Use YOLOv8 to detect license plate and EasyOCR to determine license plate number.
2. Various image pre-processing techniques like bilateral filtering, image sharpening etc. to get correct readings.
3. Store the result in a CSV file.
   1. If the license plate numbers exists in CSV, then update entry/ exit time.
   2. Else generate an alert and store the new entry with entry time.

---

# Methodology

## License Plate Validation and Processing

### Example 1: 9-character Indian License Plate

**OCR Result:** "KA05AB1234"

- **Validation Process:**
  1. **State Code:** "KA" (Karnataka)
  2. **District Code:** "05" (Bengaluru)
  3. **Character After District Code:** "A" (Alphabet, valid)
  4. **Next 2 Alphabets:** "B" and "A" (valid)
  5. **Last 4 Digits:** "1234" (valid)
- **Processing Steps:**
  - Remove unwanted characters like "IND" at the beginning.
  - Convert characters like 'A' (which can represent a digit) to '4' if necessary.
  - Validate the format: "KA05AB1234" meets the 9-character format requirements.

### Example 2: 10-character Indian License Plate

**OCR Result:** "MH02CD5678"

- **Validation Process:**
  1. **State Code:** "MH" (Maharashtra)
  2. **District Code:** "02" (Mumbai)
  3. **Next 2 Characters:** "C" and "D" (valid)
  4. **Character After District Code:** "5" (Digit, valid)
  5. **Next 2 Alphabets:** "6" and "7" (valid)
  6. **Last 4 Digits:** "5678" (valid)
- **Processing Steps:**
  - Remove unwanted characters like "IND" at the beginning.
  - Convert characters like 'A' (which can represent a digit) to '4' if necessary.
  - Validate the format: "MH02CD5678" fits the 10-character format as specified.

### Additional Considerations:

- **Character Mapping:** Ensure correct mapping for characters like 'O' to '0', 'I' to '1', 'A' to '4' to handle OCR errors.
- **Cleanup:** Remove non-alphanumeric characters to clean up OCR artifacts.
- **Validation:** Verify cleaned license plate string adheres to 9-character or 10-character format for Indian plates.

---

# Process Flow

### Process Flow for License Plate Detection and OCR

1. **Read Video Frame by Frame**
   - Use OpenCV to read video frames sequentially.
2. **Detect Cars using YOLOv8 Model**
   - Apply a pre-trained YOLOv8 model to detect cars in each frame.
3. **Crop and Detect License Plate**
   - For each detected car:
     - Use YOLOv8 to identify and crop out the license plate area.
     ![plate_detected.png](PS%2013%20Vehicle%20Movement%20Analysis%20and%20Insight%20Genera%2043441ba432854ab2918956557c0acbaa/plate_detected.png)
     - Deskew the license plate image to correct any skew.
     ![deskewed_plate.png](PS%2013%20Vehicle%20Movement%20Analysis%20and%20Insight%20Genera%2043441ba432854ab2918956557c0acbaa/deskewed_plate.png)
4. **Image Processing for OCR**

   - Convert the cropped license plate image to grayscale:
     ```python
     gray = cv.cvtColor(corrected_plate, cv.COLOR_BGR2GRAY)
     ```

   ![gray_scale_before.png](PS%2013%20Vehicle%20Movement%20Analysis%20and%20Insight%20Genera%2043441ba432854ab2918956557c0acbaa/gray_scale_before.png)

   - Apply noise reduction using a bilateral filter:
     ```python
     bfilter = cv.bilateralFilter(gray, 11, 17, 17)
     ```

   ![bfilter.png](PS%2013%20Vehicle%20Movement%20Analysis%20and%20Insight%20Genera%2043441ba432854ab2918956557c0acbaa/bfilter.png)

   - Perform edge detection using Canny:
     ```python
     edged = cv.Canny(bfilter, 30, 200)
     ```

   ![edged.png](PS%2013%20Vehicle%20Movement%20Analysis%20and%20Insight%20Genera%2043441ba432854ab2918956557c0acbaa/edged.png)

   - Find contours and filter them:
     ```python
     keypoints = cv.findContours(edged.copy(), cv.RETR_TREE, cv.CHAIN_APPROX_SIMPLE)
     contours = imutils.grab_contours(keypoints)
     contours = sorted(contours, key=cv.contourArea, reverse=True)[:10]
     ```
   - Identify and draw the contour of the license plate:
     ```python
     location = None
     for contour in contours:
         approx = cv.approxPolyDP(contour, 10, True)
         if len(approx) == 4:
             location = approx
             break
     mask = np.zeros(gray.shape, np.uint8)
     new_image = cv.drawContours(mask, [location], 0, 255, -1)
     ```

   ![draw_contours.png](PS%2013%20Vehicle%20Movement%20Analysis%20and%20Insight%20Genera%2043441ba432854ab2918956557c0acbaa/draw_contours.png)

   - Apply the mask to isolate the license plate:
     ```python
     new_image = cv.bitwise_and(corrected_plate, corrected_plate, mask=mask)
     ```

   ![bitwise_and.png](PS%2013%20Vehicle%20Movement%20Analysis%20and%20Insight%20Genera%2043441ba432854ab2918956557c0acbaa/bitwise_and.png)

   - Sharpen the license plate image for better OCR:
     ```python
     kernel = np.array([[0, -1, 0], [-1, 5, -1], [0, -1, 0]])
     sharpened_image = cv.filter2D(cropped_image, -1, kernel)
     ```

   ![sharpened.png](PS%2013%20Vehicle%20Movement%20Analysis%20and%20Insight%20Genera%2043441ba432854ab2918956557c0acbaa/sharpened.png)

5. **Perform OCR using EasyOCR**
   - Convert the sharpened image to grayscale:
     [https://drive.google.com/uc?export=view&id=18XeOZ4FoLamlXNllZL7ZrQ4UBOyVfWum](https://drive.google.com/uc?export=view&id=18XeOZ4FoLamlXNllZL7ZrQ4UBOyVfWum)
   - Convert the grayscale image to black and white:
     [https://drive.google.com/uc?export=view&id=1sRRyPCAkRsy11ciK4yff6OZuMTt5Eip5](https://drive.google.com/uc?export=view&id=1sRRyPCAkRsy11ciK4yff6OZuMTt5Eip5)
   - Feed the processed image to EasyOCR for character recognition.
6. **Filter and Validate License Plate Reading**
   - Filter out non-alphanumeric characters and smaller strings not matching license plate dimensions.
   - Validate if the recognized plate follows the Indian license plate format.
   - Handle OCR errors by mapping characters like 'O' to '0', 'I' to '1', 'A' to '4', etc.
7. **Store Data**

   - If the license plate is new (not in the CSV file), log entry/exit time and trigger alert for unknown vehicles.
   - If the license plate is recognized, update its entry/exit time in the CSV file.

   ### Sample output

   [https://youtu.be/B0dHeate1kU](https://youtu.be/B0dHeate1kU)

   ***

# Data Post-Processing

## License plate readings CSV

1. Among all the readings of the license plate from different frames, choose the one with highest confidence score.
2. Interpolate missing frames in the reading.
   Example :- We have readings for car A at frame 98 and 100, so we will fill same data for frame 99. So, when we will show the readings in the video, it will be smooth.

---

# Team members and contribution

## 1. AYUSH RAJ

Worked on Vehicle Matching through license plate detection.

---

# Conclusion

## Vehicle matching

### Use-cases

**Security and Access Control:**

- **Unauthorized Vehicle Detection:** Alert security personnel or automatically deny access to vehicles not registered in the system.
- **Visitor Management:** Track and manage visitor vehicle entry and exit times, enhancing campus or facility security.

**Law Enforcement:**

- **Wanted Vehicle Identification:** Aid law enforcement agencies in identifying vehicles associated with criminal activities or suspects.
- **Stolen Vehicle Recovery:** Quickly identify stolen vehicles through automated checks against databases.

**Logistics and Fleet Management:**

- **Fleet Tracking:** Monitor the movement of company vehicles for logistics planning and optimization.
- **Delivery Verification:** Confirm deliveries by matching license plates against expected schedules and routes.

**Border Control and Immigration:**

- **Border Security:** Monitor vehicles crossing borders and verify identities against watchlists or databases.
- **Immigration Control:** Track and manage vehicle entry and exit at border checkpoints.

**Public Safety and Emergency Response:**

- **Emergency Response Coordination:** Quickly identify and respond to emergencies by identifying vehicles involved or witnesses.
- **Public Event Security:** Monitor vehicles entering event venues for security and safety purposes.

## Show-stoppers

1. In India, people often use custom license plates with varying fonts making it difficult for EasyOCR to determine correct readings.
2. Any damage or discoloration on the license plate may lead to incorrect reading.
3. Lack of infrastructure for deployment.
