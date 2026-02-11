To move an OBJ model based on camera movements, you'll need to use a computer vision library to track motion. Here are the main approaches:

## 1. **Using OpenCV for Processing (Video Library)**

This tracks your body/face position and maps it to the model's position:

```java
import processing.video.*;
import gab.opencv.*;

Capture video;
OpenCV opencv;
PShape model;

void setup() {
  size(1280, 720, P3D);
  
  video = new Capture(this, 640, 480);
  opencv = new OpenCV(this, 640, 480);
  opencv.loadCascade(OpenCV.CASCADE_FRONTALFACE);
  
  model = loadShape("mymodel.obj");
  video.start();
}

void draw() {
  background(0);
  
  // Update camera
  if (video.available()) {
    video.read();
  }
  
  // Detect face
  opencv.loadImage(video);
  Rectangle[] faces = opencv.detect();
  
  // Draw 3D scene
  pushMatrix();
  translate(width/2, height/2, 0);
  
  if (faces.length > 0) {
    // Map face position to model position
    float x = map(faces[0].x, 0, video.width, -300, 300);
    float y = map(faces[0].y, 0, video.height, -300, 300);
    translate(x, y, 0);
  }
  
  rotateY(frameCount * 0.01);
  shape(model);
  popMatrix();
  
  // Optional: show camera feed
  image(video, 0, 0, 320, 240);
}
```

## 2. **Body Tracking with PoseNet (ml5.js in p5.js)**

For full body tracking (note: this uses p5.js, not Java Processing):

```javascript
let video;
let poseNet;
let poses = [];
let model;

function preload() {
  model = loadModel('model.obj');
}

function setup() {
  createCanvas(640, 480, WEBGL);
  video = createCapture(VIDEO);
  video.size(640, 480);
  
  poseNet = ml5.poseNet(video, modelReady);
  poseNet.on('pose', function(results) {
    poses = results;
  });
  video.hide();
}

function draw() {
  background(0);
  
  if (poses.length > 0) {
    let pose = poses[0].pose;
    
    // Use nose position to control model
    let nose = pose.nose;
    let x = map(nose.x, 0, width, -200, 200);
    let y = map(nose.y, 0, height, -200, 200);
    
    translate(x, y, 0);
  }
  
  rotateY(frameCount * 0.01);
  model(model);
}

function modelReady() {
  console.log('PoseNet Ready');
}
```

## 3. **Hand Tracking with OpenCV**

Track hand movements to control the model:

```java
import processing.video.*;
import gab.opencv.*;

Capture video;
OpenCV opencv;
PShape model;
PVector handPos = new PVector();

void setup() {
  size(1280, 720, P3D);
  
  video = new Capture(this, 640, 480);
  opencv = new OpenCV(this, 640, 480);
  
  model = loadShape("mymodel.obj");
  video.start();
}

void draw() {
  background(0);
  
  if (video.available()) {
    video.read();
  }
  
  opencv.loadImage(video);
  opencv.gray();
  opencv.threshold(70); // Adjust for your lighting
  
  // Find contours (hand silhouette)
  ArrayList<Contour> contours = opencv.findContours();
  
  if (contours.size() > 0) {
    Contour biggestContour = contours.get(0);
    Rectangle rect = biggestContour.getBoundingBox();
    
    // Update hand position
    handPos.x = map(rect.x + rect.width/2, 0, video.width, -width/2, width/2);
    handPos.y = map(rect.y + rect.height/2, 0, video.height, -height/2, height/2);
  }
  
  // Draw 3D model at hand position
  pushMatrix();
  translate(width/2 + handPos.x, height/2 + handPos.y, 0);
  rotateY(frameCount * 0.02);
  shape(model);
  popMatrix();
}
```

## 4. **Depth Camera (Kinect)**

For more precise 3D tracking:

```java
import org.openkinect.processing.*;

Kinect kinect;
PShape model;

void setup() {
  size(1280, 720, P3D);
  kinect = new Kinect(this);
  kinect.initDepth();
  kinect.initVideo();
  kinect.enableIR(true);
  
  model = loadShape("mymodel.obj");
}

void draw() {
  background(0);
  
  PVector user = kinect.getDepthMapRealWorld()[kinect.width/2 + kinect.height/2 * kinect.width];
  
  pushMatrix();
  translate(width/2, height/2, 0);
  
  // Map Kinect tracking to model position
  translate(user.x * 100, user.y * 100, user.z);
  
  rotateY(frameCount * 0.01);
  shape(model);
  popMatrix();
}
```

## Installation Requirements

**For OpenCV approach:**
1. Install OpenCV for Processing: Sketch → Import Library → Add Library → Search "OpenCV"
2. Install Video library (usually included)

**For PoseNet (p5.js):**
1. Use p5.js web editor
2. Add ml5.js library

**For Kinect:**
1. Install Kinect library for Processing
2. Connect Kinect hardware

## Simple Motion Detection Example

If you just want basic motion tracking without external libraries:

```java
import processing.video.*;

Capture video;
PImage prevFrame;
PShape model;
PVector motionCenter = new PVector();

void setup() {
  size(1280, 720, P3D);
  video = new Capture(this, 640, 480);
  model = loadShape("mymodel.obj");
  video.start();
}

void draw() {
  background(0);
  
  if (video.available()) {
    prevFrame = video.copy();
    video.read();
  }
  
  if (prevFrame != null) {
    // Detect motion
    video.loadPixels();
    prevFrame.loadPixels();
    
    float avgX = 0, avgY = 0, count = 0;
    
    for (int i = 0; i < video.pixels.length; i++) {
      color current = video.pixels[i];
      color previous = prevFrame.pixels[i];
      
      float diff = abs(brightness(current) - brightness(previous));
      
      if (diff > 50) { // Motion threshold
        int x = i % video.width;
        int y = i / video.width;
        avgX += x;
        avgY += y;
        count++;
      }
    }
    
    if (count > 0) {
      motionCenter.x = avgX / count;
      motionCenter.y = avgY / count;
    }
  }
  
  // Move model based on motion
  pushMatrix();
  translate(width/2, height/2, 0);
  translate(
    map(motionCenter.x, 0, video.width, -300, 300),
    map(motionCenter.y, 0, video.height, -300, 300),
    0
  );
  rotateY(frameCount * 0.01);
  shape(model);
  popMatrix();
}
```

Which tracking method interests you most? I can provide more detailed code for your specific use case!
