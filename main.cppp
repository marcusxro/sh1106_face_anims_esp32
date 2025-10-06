#include <U8g2lib.h>
#include <Wire.h>

U8G2_SH1106_128X64_NONAME_F_HW_I2C display(U8G2_R0, U8X8_PIN_NONE);


int leftEyeX = 40;
int rightEyeX = 88;
int eyeY = 28;
int eyeWidth = 22;
int eyeHeight = 30;


float leftPupilX = 0;
float leftPupilY = 0;
float rightPupilX = 0;
float rightPupilY = 0;
float targetPupilX = 0;
float targetPupilY = 0;


float blinkAmount = 0;
float blinkTarget = 0;
bool isBlinking = false;


bool isWakingUp = true;
unsigned long wakeUpStartTime = 0;
int wakeUpPhase = 0;

// states
enum Emotion {
  NEUTRAL,
  SUPER_HAPPY,
  BORED,
  ANGRY
};

Emotion currentEmotion = NEUTRAL;
Emotion nextEmotion = NEUTRAL;
unsigned long lastEmotionChange = 0;
unsigned long emotionDuration = 4000;
unsigned long animationTimer = 0;

float transitionProgress = 1.0;
bool isTransitioning = false;

void setup() {
  Serial.begin(115200);
  display.begin();
  display.setContrast(255);
  randomSeed(analogRead(0));
  
  wakeUpStartTime = millis();
  blinkAmount = eyeHeight; 
  
  Serial.println("Cat Waking Up!");
}

void loop() {
  unsigned long currentTime = millis();

  if (isWakingUp) {
    handleWakeUp(currentTime);
  } else {
    if (currentTime - lastEmotionChange > emotionDuration && !isTransitioning) {
      lastEmotionChange = currentTime;
      startTransition();
    }
  }

  if (isTransitioning) {
    transitionProgress += 0.08;
    if (transitionProgress >= 1.0) {
      transitionProgress = 1.0;
      isTransitioning = false;
      currentEmotion = nextEmotion;
    }
  }

  leftPupilX += (targetPupilX - leftPupilX) * 0.18;
  leftPupilY += (targetPupilY - leftPupilY) * 0.18;
  rightPupilX = leftPupilX;
  rightPupilY = leftPupilY;

  blinkAmount += (blinkTarget - blinkAmount) * 0.3;

  if (!isWakingUp && random(100) < 3 && !isBlinking && currentEmotion != SUPER_HAPPY) {
    isBlinking = true;
    blinkTarget = eyeHeight;
  }

  if (isBlinking && blinkAmount >= eyeHeight - 2) {
    if (random(100) < 40) {
      isBlinking = false;
      blinkTarget = 0;
    }
  }

  display.clearBuffer();
  
  if (isWakingUp) {
    drawWakeUpAnimation();
  } else {
    drawCurrentEmotion();
  }

  display.sendBuffer();
  delay(40);
}

// -------------------- WAKE-UP ANIMATION --------------------

void handleWakeUp(unsigned long currentTime) {
  unsigned long elapsed = currentTime - wakeUpStartTime;
  
  if (elapsed < 1000) {
    wakeUpPhase = 0;
    blinkTarget = eyeHeight;
  } else if (elapsed < 2000) {
    wakeUpPhase = 1;
    float openProgress = (elapsed - 1000) / 1000.0;
    blinkTarget = eyeHeight * (1.0 - openProgress);
  } else if (elapsed < 3500) {
    wakeUpPhase = 2;
    blinkTarget = 0;
    targetPupilX = 0;
    targetPupilY = -3;
    animationTimer = currentTime;
  } else if (elapsed < 4500) {
    wakeUpPhase = 3;
    float mouthProgress = (elapsed - 3500) / 1000.0;
    targetPupilY = -3 + (3 * mouthProgress);
  } else {
    isWakingUp = false;
    currentEmotion = NEUTRAL;
    targetPupilX = 0;
    targetPupilY = 0;
    lastEmotionChange = currentTime;
    Serial.println("Wake up complete! Normal mode.");
  }
}

void drawWakeUpAnimation() {
  if (wakeUpPhase == 0 || wakeUpPhase == 1) {
    drawCatEye(leftEyeX, eyeY, leftPupilX, leftPupilY);
    drawCatEye(rightEyeX, eyeY, rightPupilX, rightPupilY);
    drawScreamingMouth(0); 
  } else if (wakeUpPhase == 2) {
    drawCatEye(leftEyeX, eyeY, leftPupilX, leftPupilY);
    drawCatEye(rightEyeX, eyeY, rightPupilX, rightPupilY);
    drawScreamingMouth(1);
  } else if (wakeUpPhase == 3) {
    float progress = (millis() - (wakeUpStartTime + 3500)) / 1000.0;
    drawCatEye(leftEyeX, eyeY, leftPupilX, leftPupilY);
    drawCatEye(rightEyeX, eyeY, rightPupilX, rightPupilY);
    drawScreamingMouth(1 - progress); 
  }
}

// -------------------- MOUTH ANIMATION --------------------

void drawScreamingMouth(float openAmount) {
  int mouthCenterX = 64;
  int mouthY = 52;
  int mouthHeight = 10 * openAmount; 
  int mouthWidth = 18;

  for (int x = -mouthWidth; x <= mouthWidth; x++) {
    int y = mouthY + (x*x) / 60;
    for (int dy = 0; dy < mouthHeight; dy++) {
      display.drawPixel(mouthCenterX + x, y + dy);
    }
  }
}

// -------------------- EMOTIONS --------------------

void startTransition() {
  isTransitioning = true;
  transitionProgress = 0.0;
  nextEmotion = getRandomEmotion();
  Serial.print("Transitioning to: ");
  Serial.println(nextEmotion);
}

Emotion getRandomEmotion() {
  int rand = random(100);
  if (rand < 30) {
    emotionDuration = random(3000, 5000);
    targetPupilX = random(-4, 5);
    targetPupilY = random(-2, 3);
    return NEUTRAL;
  } else if (rand < 50) {
    emotionDuration = random(3000, 4500);
    targetPupilX = 0;
    targetPupilY = -3;
    animationTimer = millis();
    return SUPER_HAPPY;
  } else if (rand < 75) {
    emotionDuration = random(4000, 6000);
    targetPupilX = random(-4, 5);
    targetPupilY = 4;
    return BORED;
  } else {
    emotionDuration = random(2500, 4000);
    targetPupilX = 0;
    targetPupilY = 0;
    return ANGRY;
  }
}

void drawCurrentEmotion() {
  if (isTransitioning) {
    if (transitionProgress < 0.3) {
      blinkTarget = eyeHeight * (transitionProgress / 0.3);
    } else if (transitionProgress < 0.7) {
      blinkTarget = eyeHeight;
    } else {
      blinkTarget = eyeHeight * (1.0 - (transitionProgress - 0.7) / 0.3);
    }
    if (transitionProgress > 0.5) drawEmotion(nextEmotion);
    else drawEmotion(currentEmotion);
  } else {
    drawEmotion(currentEmotion);
  }
}

void drawEmotion(Emotion emotion) {
  switch (emotion) {
    case NEUTRAL: drawNeutralFace(); break;
    case SUPER_HAPPY: drawSuperHappyFace(); break;
    case BORED: drawBoredFace(); break;
    case ANGRY: drawAngryFace(); break;
  }
}

// -------------------- FACE DRAWING --------------------

void drawNeutralFace() {
  unsigned long t = millis();
  float breathe = sin(t / 1000.0) * 0.5;
  targetPupilX += sin(t / 2000.0) * 0.1;
  targetPupilY += cos(t / 2500.0) * 0.1;
  drawCatEye(leftEyeX, eyeY, leftPupilX, leftPupilY);
  drawCatEye(rightEyeX, eyeY, rightPupilX, rightPupilY);
  
  // Gentle smile
  int mouthY = 54;
  for (int x = -12; x <= 12; x++) {
    int y = mouthY + (x * x) / 30 + breathe;
    display.drawPixel(64 + x, y);
    display.drawPixel(64 + x, y + 1);
  }
}

void drawSuperHappyFace() {
  for (int i = 0; i < 4; i++) {
    for (int x = -eyeWidth/2; x <= eyeWidth/2; x++) {
      int y = eyeY + 5 - abs(x) / 3;
      display.drawPixel(leftEyeX + x, y + i);
      display.drawPixel(rightEyeX + x, y + i);
    }
  }
  
  int mouthY = 52;
  for (int x = -16; x <= 16; x++) {
    int y = mouthY + (x * x) / 25;
    display.drawPixel(64 + x, y);
    display.drawPixel(64 + x, y + 1);
    display.drawPixel(64 + x, y + 2);
  }

  unsigned long elapsed = millis() - animationTimer;
  drawFloatingHeart(15, elapsed, 40, 4);
  drawFloatingHeart(110, elapsed, 45, 4);
  drawFloatingHeart(8, elapsed, 50, 3);
  drawFloatingHeart(118, elapsed, 43, 3);
  drawFloatingHeart(64, elapsed, 48, 3);
  drawFloatingHeart(25, elapsed, 55, 3);
  drawFloatingHeart(100, elapsed, 52, 3);
  drawFloatingHeart(50, elapsed, 38, 2);
  drawFloatingHeart(78, elapsed, 42, 2);
}

void drawFloatingHeart(int baseX, unsigned long elapsed, int speed, int size) {
  int heartY = 64 - ((elapsed / speed) % 70);
  if (heartY < 64 && heartY > -10) drawBigHeart(baseX, heartY, size);
}

void drawBoredFace() {
  unsigned long t = millis();
  float drift = sin(t / 3000.0) * 2;
  drawCatEye(leftEyeX, eyeY + 4, leftPupilX + drift, leftPupilY);
  drawCatEye(rightEyeX, eyeY + 4, rightPupilX + drift, rightPupilY);
  
  display.setDrawColor(0);
  display.drawBox(leftEyeX - eyeWidth/2 - 2, eyeY - eyeHeight/2, eyeWidth + 4, eyeHeight/2 + 4);
  display.drawBox(rightEyeX - eyeWidth/2 - 2, eyeY - eyeHeight/2, eyeWidth + 4, eyeHeight/2 + 4);
  display.setDrawColor(1);
  
  for (int i = 0; i < 3; i++) {
    display.drawLine(leftEyeX - eyeWidth/2, eyeY + 4 + i, leftEyeX + eyeWidth/2, eyeY + 4 + i);
    display.drawLine(rightEyeX - eyeWidth/2, eyeY + 4 + i, rightEyeX + eyeWidth/2, eyeY + 4 + i);
  }
  
  display.drawLine(52, 54, 76, 54);
  display.drawLine(52, 55, 76, 55);
  display.drawLine(52, 56, 76, 56);
  
  int sweatY = 20 + (t / 100) % 15;
  display.drawCircle(12, sweatY, 5);
  display.drawCircle(12, sweatY, 4);
  if (sweatY < 30) {
    display.drawLine(12, sweatY + 5, 10, sweatY + 10);
    display.drawLine(12, sweatY + 5, 14, sweatY + 10);
  }
}

void drawAngryFace() {
  unsigned long t = millis();
  int shake = (t / 100) % 2 == 0 ? 1 : -1;
  drawCatEye(leftEyeX + shake, eyeY, leftPupilX, leftPupilY);
  drawCatEye(rightEyeX + shake, eyeY, rightPupilX, rightPupilY);
  
  for (int i = 0; i < 5; i++) {
    display.drawLine(leftEyeX - eyeWidth/2 - 3, eyeY - eyeHeight/2 - 8 + i, 
                     leftEyeX + eyeWidth/2 - 5, eyeY - eyeHeight/2 - 3 + i);
    display.drawLine(rightEyeX - eyeWidth/2 + 5, eyeY - eyeHeight/2 - 3 + i, 
                     rightEyeX + eyeWidth/2 + 3, eyeY - eyeHeight/2 - 8 + i);
  }

  int mouthY = 56;
  for (int x = -12; x <= 12; x++) {
    int y = mouthY - (x * x) / 35;
    display.drawPixel(64 + x + shake, y);
    display.drawPixel(64 + x + shake, y - 1);
    display.drawPixel(64 + x + shake, y - 2);
  }

  int pulse = (t / 150) % 2;
  for (int i = 0; i < 3 + pulse; i++) {
    display.drawLine(6 + i, 10, 10 + i, 14);
    display.drawLine(10 + i, 10, 6 + i, 14);
    display.drawCircle(8 + i, 12, 6);
    
    display.drawLine(112 + i, 10, 116 + i, 14);
    display.drawLine(116 + i, 10, 112 + i, 14);
    display.drawCircle(114 + i, 12, 6);
  }
}

// -------------------- EYES --------------------

void drawCatEye(int x, int y, float pupilOffsetX, float pupilOffsetY) {
  int currentHeight = eyeHeight - blinkAmount;
  
  if (currentHeight <= 3) {
    for (int i = 0; i < 4; i++) {
      display.drawLine(x - eyeWidth/2, y + i - 2, x + eyeWidth/2, y + i - 2);
    }
    return;
  }
  
  for (int dy = -currentHeight/2; dy <= currentHeight/2; dy++) {
    float widthRatio = 1.0 - (float)(dy * dy) / (float)((currentHeight/2) * (currentHeight/2));
    int width = eyeWidth * widthRatio / 2;
    display.drawPixel(x - width, y + dy);
    display.drawPixel(x + width, y + dy);
    for (int dx = -width + 1; dx < width; dx++) {
      display.drawPixel(x + dx, y + dy);
    }
  }
  
  int pupilX = x + pupilOffsetX * 2;
  int pupilY = y + pupilOffsetY * 2;
  int pupilWidth = 5;
  int pupilHeight = currentHeight * 0.6;
  for (int py = -pupilHeight/2; py <= pupilHeight/2; py++) {
    for (int px = -pupilWidth/2; px <= pupilWidth/2; px++) {
      display.setDrawColor(0);
      display.drawPixel(pupilX + px, pupilY + py);
    }
  }
  display.setDrawColor(1);
  display.drawPixel(pupilX - 2, pupilY - pupilHeight/3);
  display.drawPixel(pupilX - 2, pupilY - pupilHeight/3 + 1);
  display.drawPixel(pupilX - 3, pupilY - pupilHeight/3);
}

// -------------------- HEART --------------------

void drawBigHeart(int x, int y, int size) {
  if (y < -5 || y > 68) return;
  display.drawDisc(x - size, y - size/2, size);
  display.drawDisc(x + size, y - size/2, size);
  for (int i = 0; i <= size * 2; i++) {
    int bottomY = y + size * 2 - i/2;
    if (bottomY < 64) {
      display.drawLine(x - size * 2 + i, y, x, bottomY);
      display.drawLine(x + size * 2 - i, y, x, bottomY);
    }
  }
}
