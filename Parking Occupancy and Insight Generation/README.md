# PS 13: Vehicle Movement Analysis and Insight Generation in a College Campus using Edge AI

# Unique Idea Brief
### Occupancy Monitoring:

1. In any frame of the parking-lot video, the area of interest’s (AOI) i.e. parking areas’ coordinates are specified with respect to any frame size (640 * 360).
2. Then with the help of YOLOv8 pre-trained model we will detect the objects in the frame.
3. If in any frame the detected object is a car and its bounding box’s centre lie inside any of the specified parking area, we will mark the area as occupied and decrease the free space number.

### Vehicle movement analysis:

1. While monitoring the cars occupancy, we are also storing the time stamp and number of cars at that moment in a csv file.
2. The number of cars is resampled for a specific time duration (20s / 30min/ 1hr) and its mean is taken.
3. The resampled data’s mean and standard deviation is used to calculate a range within which parking lot is occupied for most of the time.
4. The calculated mean and standard deviations are used to calculate the time when least and most cars are present.

---

# Datasets

### Occupancy monitoring

used parking-lot video link:

https://mega.nz/file/4w0UCRyJ#TV55n4q-1j4jLCHtT_jmDZjMGdvny0-4Nlj8hAImd7Y

# Process Flow

---

### Process Flow for Occupancy monitoring and Vehicle movement analysis:

![flowChart.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/689c7499-36d0-4d4c-88de-8cbb5a80796a/6bd931c7-fbc0-4bc2-854e-00c96b7e9342/flowChart.png)

1. **Area of Interest (AOI):**
    - Specify the area of interest for the frame of the same size as the screen in which video is going to play.
    
    ![Screenshot 2024-07-07 213535.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/689c7499-36d0-4d4c-88de-8cbb5a80796a/cc352968-af97-404e-a564-b9f6f6e833b0/ab2bff60-58a7-42cf-850c-dab0d7952c6a.png)
    
2. **Object detection using YOLOv8:**
    - Whenever a car is detected in a frame check the position of the coordinate of the centre of the bounding box of the car.
    - Use “pointPolygonTest” to check if the centre is inside or outside the AOI (i.e. parking space).
    - If the point is inside the AOI, then mark the parking-space as occupied.
    
    ![free space count is specified on the top-left corner](https://prod-files-secure.s3.us-west-2.amazonaws.com/689c7499-36d0-4d4c-88de-8cbb5a80796a/25360f19-9f35-4f38-b377-f5d1cf6b2dc3/Screenshot_2024-07-07_213743.png)
    
    Free space count is specified on the top-left corner
    
3. **Store Data**
    - While doing it in each frame store the number of cars present at that moment and the timestamp in a csv file.
    - So that the data from that csv file can be used later for analysis purposes.
4. **Analysis**
    - The number of cars is resampled for a specific time duration (20s / 30min/ 1hr) and its mean is taken. Resampled data’s mean and standard deviation are found.
    - Mean and standard deviation are used to calculate the range of [minCarCount, maxCarCount]. Values above and below these values are considered as least crowded and most crowded time.

![Screenshot 2024-07-06 075537.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/689c7499-36d0-4d4c-88de-8cbb5a80796a/31c716e5-e62b-47bf-8653-8b10918477f2/Screenshot_2024-07-06_075537.png)

---

# Team members and contribution

Worked on parking lot occupancy and vehicle movement analysis.

---

# Conclusion

## Use-cases

### Occupancy and movement analysis

**Provide real time occupancy**:

- Provide real time data on the number of available spaces.
- Reduces the time used by drivers to find parking space.

**Parking guidance system:**

- This system can guide drivers to available spaces using mobile apps.
- Increases the user experience.

**Use cases in smart city:**

- It decreases the time spent searching for parking space, which leads to a decrease in carbon dioxide emission.

**Dynamic pricing system:**

- Adjust parking space price based on demand and occupancy of the parking space.

## Show-stoppers

### Occupancy and movement analysis

1. High initial-cost: Significant investment required for infrastructure and technology integration.
2. Data privacy concern: Collecting and storing data on vehicle movements and parking raises privacy issues.
