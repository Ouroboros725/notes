# How to automate gestures

**Gestures**  play an important role in how your app is being used. With lot of unique gesture supported on mobile devices, automation has its own challenge. Some of the common gestures are:

```
single tap
double tap
flick (left or right)
pull down to refresh
long press

```

Appium handles these gestures using  **TouchActions**  api they have created. It’s more like Actions class in Selenium. Apart from that they have also enabled JSON wire protocol server extensions.

We can pass in coordinates as parameters and specify the action we want. Sample code would look like as:

```
JavascriptExecutor js = (JavascriptExecutor) driver;
HashMap<String, Double> swipeObject = new HashMap<String, Double>();
swipeObject.put(“startX”, 0.01);
swipeObject.put(“startY”, 0.5);
swipeObject.put(“endX”, 0.9);
swipeObject.put(“endY”, 0.6);
swipeObject.put(“duration”, 3.0);
js.executeScript(“mobile: swipe”, swipeObject);

```

When we enter the coordinates in decimal, it actually specifies the percentage. So in above example it means, 1% from x and 50% from y coordinates.  _Duration_  basically specifies how long it will tap and is in seconds.

Some of the mobile methods to be used are:

```
mobile: tap
mobile: flick
mobile: swipe
mobile: scroll
mobile: shake

```

Prefix “_mobile:_” allows us to route these requests to the appropriate endpoint.

You might also want to explore  _TouchActions_  class provided by Selenium. It implements actions for touch devices and basically built upon the Actions class.

We have added some test in the git project to show how to use the gestures in automation.

The project would help you learn how to automate android app using Appium.

Appium let's you do a scroll to the element text but sometimes it might not work depending on how app is and CSS structure are. You can write your own parallel code to perform scroll and here is the code snippet for the same.

```
JavascriptExecutor js = (JavascriptExecutor) driver;
    HashMap<String, String> scrollObject = new HashMap<String, String>();
    scrollObject.put("direction", "down");
    js.executeScript("mobile: scroll", scrollObject);
```

https://nishantverma.gitbooks.io/appium-for-android/content/automating_gestures/index.html