<div align="center"><img src="/screens/cover_2.gif" width="600"/></div>

# VectorMaster
This library introduces dynamic control over vector drawables. Each and every aspect of a vector drawable can be controlled dynamically(via Java instances), using this library. 

Features :

- <b>Control</b> : Control every attribute related to `path`, `group`, `vector` and `clip-path` like **color**, **alpha**, **strokeWdith**, **translation**, **scale**, **rotation** etc.  
- <b>Clip Paths</b> : The library supports clip paths.
- <b>Trimming</b> : The library allows trimming of path by using `trimEnd`, `trimStart` and `trimOffset` parameters.

# Usage
Just add the following dependency in your app's `build.gradle`
```groovy
dependencies {
      compile 'com.sdsmdg.harjot:vectormaster:1.0.1'
}
```

# Background and Working
`VectorDrawables` are really helpful for removing scaling problem but they lack control. Most of the changes to vector drawables are only possible by creating an `AnimatedVectorDrawable` and also defining animations. All of this is good but lacks control that may be required during runtime. 

For example, if we need to change vector's properties *(Say, color)* based on a user action *(Say, if user is choosing the theme of app)*. We can achieve this using `AnimatedVectorDrawable` but only to an extent, this approach can't be used if user action leads to an infinite number of cases *(Say, if user picks up a random color for theme)* and we need to change property of the vector for each case. Thus we need a mechanism that can be used to change a vector's properties at runtime using basic methods like `setColor`, `setScale`, `setTranslation` etc. This is where this library comes in.

The working of the library is as follows :
- First the `vector.xml`*(the `VectorDrawable` that we wish to control)*, is parsed using `XmlPullParser` and the attributes are stored in Models corresponding to the tag.
- `vector` attributes are stored in `VectorModel`, `group` attributes in `GroupModel`, `path` atrributes in `PathModel` and `clip-path` attributes in `ClipPathModel`. The hierarchy is as follows :<br><br><div align="center"><img src="/screens/Hierarchy.png" width="600"/></div><br>
- The `pathData` in `PathModel` is then parsed using `PathParser.java`; It parses the string data and converts it into a `Path` object.
- All the transformations, scaling etc are done using **Matrices** after the `Path` object is built. All this is done prior to the first draw on canvas.
- At first draw we have the same output as we should have got if we used inbuilt methods to draw the `vector.xml` using `srcCompat`.
- Now, all Models are accessible via `getModelByName(...)` public methods that can be directly called via the instance of `VectorMasterView` that we get using `findViewById(...)`.
- If we wish to change any value, we just need to call `model.setParamter(...)`. `model` is of type `VectorModel`, `GroupModel`, `PathModel` or `ClipPathModel`. `parameter` can be anything like **color**, **scale**, **rotation** etc. depending on the model we are using.
- After setting a paramter the necesarry `paints` and `paths` are rebuilt, scaled, transformed etc.
- A call to `update` method repaints the canvas with the required changes.

# Examples

#### ic_heart.xml (This is the original vector that has been used in all examples)
<img src="/screens/ic_heart_original_resized.png" align="right" width="200">

```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
        android:width="24dp"
        android:height="24dp"
        android:viewportWidth="24.0"
        android:viewportHeight="24.0">
    <path
        android:name="outline"
        android:pathData="M20.84,4.61a5.5,5.5 0,0 0,-7.78 0L12,5.67l-1.06,-1.06a5.5,5.5 0,0 0,-7.78 7.78l1.06,1.06L12,21.23l7.78,-7.78 1.06,-1.06a5.5,5.5 0,0 0,0 -7.78z"
        android:strokeLineCap="round"
        android:strokeColor="#5D5D5D"
        android:fillColor="#00000000"
        android:strokeWidth="2"
        android:strokeLineJoin="round"/>
</vector>
```

## Example 1 (Simple Color change)
#### XML
```xml
<com.sdsmdg.harjot.vectormaster.VectorMasterView
        android:id="@+id/heart_vector"
        android:layout_width="150dp"
        android:layout_height="150dp"
        app:vector_src="@drawable/ic_heart" />
```

#### Java
```java
VectorMasterView heartVector = (VectorMasterView) findViewById(R.id.heart_vector);

// find the correct path using name
PathModel outline = heartVector.getPathModelByName("outline");

// set the stroke color
outline.setStrokeColor(Color.parseColor("#ED4337"));

// set the fill color (if fill color is not set or is TRANSPARENT, then no fill is drawn)
outline.setFillColor(Color.parseColor("#ED4337"));
```

#### Result
<div align="center"><img src="/screens/result_1.png" width="500"/></div>

## Example 2 (Trim paths)
#### XML
```xml
<com.sdsmdg.harjot.vectormaster.VectorMasterView
        android:id="@+id/heart_vector"
        android:layout_width="150dp"
        android:layout_height="150dp"
        app:vector_src="@drawable/ic_heart" />
```

#### Java
```java
VectorMasterView heartVector = (VectorMasterView) findViewById(R.id.heart_vector);

// find the correct path using name
PathModel outline = heartVector.getPathModelByName("outline");

// set trim path start (values are given in fraction of length)
outline.setTrimPathStart(0.0f);

// set trim path end (values are given in fraction of length)
outline.setTrimPathEnd(0.65f);
```

#### Result
<div align="center"><img src="/screens/result_2.png" width="500"/></div>

## Example 3 (Simple color animation using ValueAnimator)
#### XML
```xml
<com.sdsmdg.harjot.vectormaster.VectorMasterView
        android:id="@+id/heart_vector"
        android:layout_width="150dp"
        android:layout_height="150dp"
        app:vector_src="@drawable/ic_heart" />
```

#### Java
```java
VectorMasterView heartVector = (VectorMasterView) findViewById(R.id.heart_vector);

// find the correct path using name
PathModel outline = heartVector.getPathModelByName("outline");

outline.setStrokeColor(Color.parseColor("#ED4337"));

heartVector.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {

    	// initialize valueAnimator and pass start and end color values
        ValueAnimator valueAnimator = ValueAnimator.ofObject(new ArgbEvaluator(), Color.WHITE, Color.parseColor("#ED4337"));
        valueAnimator.setDuration(1000);

        valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator valueAnimator) {

            	// set fill color and update view
                outline.setFillColor((Integer) valueAnimator.getAnimatedValue());
                heartVector.update();
            }
        });
        valueAnimator.start();
    }
});
```

#### Result
<div align="center"><img src="/screens/result_3.gif" width="300"/></div>

## Example 4 (Simple trim animation using ValueAnimator)
#### XML
```xml
<com.sdsmdg.harjot.vectormaster.VectorMasterView
        android:id="@+id/heart_vector"
        android:layout_width="150dp"
        android:layout_height="150dp"
        app:vector_src="@drawable/ic_heart" />
```

#### Java
```java
VectorMasterView heartVector = (VectorMasterView) findViewById(R.id.heart_vector);

// find the correct path using name
PathModel outline = heartVector.getPathModelByName("outline");

outline.setStrokeColor(Color.parseColor("#ED4337"));
outline.setTrimPathEnd(0.0f);

heartVector.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {

    	// initialise valueAnimator and pass start and end float values
        ValueAnimator valueAnimator = ValueAnimator.ofFloat(0.0f, 1.0f);
        valueAnimator.setDuration(1000);

        valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator valueAnimator) {

            	// set trim end value and update view
                outline.setTrimPathEnd((Float) valueAnimator.getAnimatedValue());
                heartVector.update();
            }
        });
        valueAnimator.start();
    }
});
```

#### Result
<div align="center"><img src="/screens/result_4.gif" width="300"/></div>

# Complex animations
The above examples are just the basic use cases and are meant to serve as a quick start to using the library. For more complex animations and use cases involving **clip-paths** and **groups**, head to [Here](abc)<br>
<div align="center"><img src="/screens/more_animations.gif" width="500"/></div>