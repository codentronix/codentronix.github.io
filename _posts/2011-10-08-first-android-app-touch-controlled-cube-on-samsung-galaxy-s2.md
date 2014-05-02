---
layout: post
title: ! 'First Android App: Touch Controlled Cube on Samsung Galaxy S2'
categories:
- Android
tags:
- 3d
- android
- samsung galaxy s2
status: publish
type: post
published: true
meta:
  _oembed_3fc9442f0934458338429cccf8767b45: <iframe width="640" height="360" src="http://www.youtube.com/embed/0URpA8LPotk?feature=oembed"
    frameborder="0" allowfullscreen></iframe>
  og_video: http://www.youtube.com/v/0URpA8LPotk
  og_description: A couple of weeks ago I started to tinker with android programming.
    After reading some articles in android developers guide I developed my first application.
    It is a simple application that draws a cube in a 2D canvas. The cube can be controlled
    with touch input. Below you may see the video of the application in action. I
    have tested the application on a Samsung Galaxy S2.
  _oembed_83e387d927734653ad009893413fea04: <object width="400" height="225"><param
    name="movie" value="http://www.youtube.com/v/0URpA8LPotk?version=3"></param><param
    name="allowFullScreen" value="true"></param><param name="allowscriptaccess" value="always"></param><embed
    src="http://www.youtube.com/v/0URpA8LPotk?version=3" type="application/x-shockwave-flash"
    width="400" height="225" allowscriptaccess="always" allowfullscreen="true"></embed></object>
  views: '8785'
author:
  login: admin
  email: leonelmachava@gmail.com
  display_name: lefam
  first_name: Leonel
  last_name: Machava
---
A couple of weeks ago I started to tinker with android programming. After reading some articles in <a href="http://developer.android.com/guide/index.html" rel="nofollow">android developers guide</a> I developed my first application. It is a simple application that draws a cube in a 2D canvas. The cube can be controlled with touch input. Below you may see the video of the application in action. I have tested the application on a Samsung Galaxy S2.

<object width="400" height="225"><param
    name="movie" value="http://www.youtube.com/v/0URpA8LPotk?version=3"></param><param
    name="allowFullScreen" value="true"></param><param name="allowscriptaccess" value="always"></param><embed
    src="http://www.youtube.com/v/0URpA8LPotk?version=3" type="application/x-shockwave-flash"
    width="400" height="225" allowscriptaccess="always" allowfullscreen="true"></embed></object>

<!--more-->

##The Idea

I have created a custom view by extending the *View* class. In the custom view I overridden the *onDraw()* method to draw the cube. I also overridden the *onTouchEvent() *method to capture touch events.

##The Code

**CubeView.java**:
The *CubeView* class is the custom view responsible for drawing the cube.

{% highlight java %}
package com.codentronix.android;
import android.content.Context;
import android.util.AttributeSet;
import android.view.MotionEvent;
import android.view.View;
import android.graphics.Canvas;
import android.graphics.Paint;
import android.graphics.Color;
import android.graphics.Paint.Style;
import android.graphics.Path;
public class CubeView extends View {
	/* Vertices of the cube. */
	protected Vector3D vertices[];
	/* Define the indices to the vertices of each face of the cube. */
	protected int faces[][];
	/* Colors of each face of the cube. */
	protected int colors[];
	/* Orientation of the cube around X axis. */
	protected float ax;
	/* Orientation of the cube around Y axis. */
	protected float ay;
	/* Orientation of the cube around Z axis. */
	protected float az;
	protected float lastTouchX;
	protected float lastTouchY;
	/* This constructor is used when the view is created from code. */
	public CubeView(Context context) {
		super(context);
	}
	/* This constructor is used when the view is inflated from a XML file. */
	public CubeView(Context context, AttributeSet attrs) {
		super(context, attrs);
	}
	/**
	 * Initializes the cube geometry.
	 */
	public void initialize() {
		vertices = new Vector3D[] {
	        new Vector3D(-1, 1, -1),
	        new Vector3D(1, 1, -1),
	        new Vector3D(1, -1, -1),
	        new Vector3D(-1, -1, -1),
	        new Vector3D(-1, 1, 1),
	        new Vector3D(1, 1, 1),
	        new Vector3D(1, -1, 1),
	        new Vector3D(-1, -1, 1)
		};
		// Define the 6 faces of the cube. We specify the indices to the 4 vertices of each face.
  {% raw %}
		faces = new int[][] {{0, 1, 2, 3}, {1, 5, 6, 2}, {5, 4, 7, 6}, {4, 0, 3, 7}, {0, 4, 5, 1}, {3, 2, 6, 7}};
  {% endraw %}
		// Define the color of each face.
		colors = new int[] {Color.BLUE, Color.RED, Color.YELLOW, Color.LTGRAY, Color.CYAN, Color.MAGENTA};
	}
	@Override
	protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
		super.onLayout(changed, left, top, right, bottom);
		// Initialize the cube geometry.
		initialize();
		// Allow the view to receive touch input.
		setFocusableInTouchMode(true);
	}
	@Override
	public boolean onTouchEvent(MotionEvent event) {
		if( event.getAction() == MotionEvent.ACTION_DOWN ) {
			lastTouchX = event.getX();
			lastTouchY = event.getY();
		}else if( event.getAction() == MotionEvent.ACTION_MOVE ) {
			float dx = (event.getX() - lastTouchX) / 30.0f;
			float dy = (event.getY() - lastTouchY) / 30.0f;
			ax += dy;
			ay -= dx;
			postInvalidate();
		}
		return true;
	}
	@Override
	protected void onDraw(Canvas canvas) {
		Vector3D t[]  = new Vector3D[8];
		double avgZ[] = new double[6];
		int order[] = new int[6];
		for( int i = 0; i < 8; i++ ) {
			// Rotate the vertex around X, next around Y, and then around Z.
			t[i] = vertices[i].rotateX(ax).rotateY(ay).rotateZ(az);
			// Finally, map the vertex from 3D to 2D.
			t[i] = t[i].project(getWidth(), getHeight(), 256, 4);
		}
		// Compute the average Z value of each face.
		for( int i = 0; i < 6; i++ ) {
			avgZ[i] = (t[faces[i][0]].z + t[faces[i][1]].z + t[faces[i][2]].z + t[faces[i][3]].z) / 4;
			order[i] = i;
		}
        // Next we sort the faces in descending order based on the Z value.
        // The objective is to draw distant faces first. This is called
        // the PAINTERS ALGORITHM. So, the visible faces will hide the invisible ones.
        // The sorting algorithm used is the SELECTION SORT.
		for( int i = 0; i             int iMax = i;
    		for( int j = i + 1; j  avgZ[iMax] ) {
                    iMax = j;
                }
    		}
            if( iMax != i ) {
	            double dTmp = avgZ[i];
	            avgZ[i] = avgZ[iMax];
	            avgZ[iMax] = dTmp;
	            int iTmp = order[i];
	            order[i] = order[iMax];
	            order[iMax] = iTmp;
            }
		}
		canvas.drawColor(Color.BLACK);
		Paint paint = new Paint();
		paint.setColor(Color.WHITE);
		paint.setStyle(Style.FILL);
		paint.setTextSize(24);
		canvas.drawText('3D Cube',10,40,paint);
		paint.setTextSize(12);
		canvas.drawText('Drag the cube to change its orientation', 10, 80, paint);
		canvas.drawText('http://codentronix.com', 10, getHeight() - 10, paint);
		for( int i = 0; i < 6; i++ ) {
			int index = order[i];
			Path p = new Path();
			p.moveTo((float)t[faces[index][0]].x, (float)t[faces[index][0]].y);
			p.lineTo((float)t[faces[index][1]].x, (float)t[faces[index][1]].y);
			p.lineTo((float)t[faces[index][2]].x, (float)t[faces[index][2]].y);
			p.lineTo((float)t[faces[index][3]].x, (float)t[faces[index][3]].y);
			p.close();
			paint.setColor(colors[index]);
			canvas.drawPath(p, paint);
		}
	}
}
{% endhighlight %}

**Vector3D.java**:
The *Vector3D* class is be used to define each vertex of the cube in 3D space.

{% highlight java %}
package com.codentronix.android;
public class Vector3D {
	protected double x;
	protected double y;
	protected double z;
	public Vector3D() {
		x = y = z = 0;
	}
	public Vector3D(double x, double y, double z) {
		this.x = x;
		this.y = y;
		this.z = z;
	}
	public Vector3D rotateX(double angle) {
        double rad, cosa, sina, yn, zn;
        rad = angle * Math.PI / 180;
        cosa = Math.cos(rad);
        sina = Math.sin(rad);
        yn = this.y * cosa - this.z * sina;
        zn = this.y * sina + this.z * cosa;
        return new Vector3D(this.x, yn, zn);
	}
	public Vector3D rotateY(double angle) {
        double rad, cosa, sina, xn, zn;
        rad = angle * Math.PI / 180;
        cosa = Math.cos(rad);
        sina = Math.sin(rad);
        zn = this.z * cosa - this.x * sina;
        xn = this.z * sina + this.x * cosa;
        return new Vector3D(xn, this.y, zn);
	}
	public Vector3D rotateZ(double angle) {
        double rad, cosa, sina, xn, yn;
        rad = angle * Math.PI / 180;
        cosa = Math.cos(rad);
        sina = Math.sin(rad);
        xn = this.x * cosa - this.y * sina;
        yn = this.x * sina + this.y * cosa;
        return new Vector3D(xn, yn, this.z);
	}
	public Vector3D project(int viewWidth, int viewHeight, float fov, float viewDistance) {
        double factor, xn, yn;
        factor = fov / (viewDistance + this.z);
        xn = this.x * factor + viewWidth / 2;
        yn = this.y * factor + viewHeight / 2;
        return new Vector3D(xn, yn, this.z);
	}
}
{% endhighlight %}

The methods *rotateX()*, *rotateY()*, and *rotateZ()* rotate the vertex around X, Y, and Z axis respectively. The method *project()* maps the vertex from 3D space to 2D space.

**CubeIn2DCanvasActivity.java:**

This file defines the main activity of the application. The activity loads the layout and adds a menu item. The layout is composed by the custom view we defined above.

{% highlight java %}
package com.codentronix.android;
import android.app.Activity;
import android.app.AlertDialog;
import android.app.AlertDialog.Builder;
import android.app.Dialog;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuInflater;
import android.view.MenuItem;
import android.view.Window;
public class CubeIn2DCanvasActivity extends Activity {
    /** Called when the activity is first created. */
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // Remove the title bar.
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        setContentView(R.layout.main);
    }
	@Override
	public boolean onCreateOptionsMenu(Menu menu) {
		MenuInflater inflater = getMenuInflater();
		inflater.inflate(R.menu.main, menu);
		return true;
	}
	@Override
	protected Dialog onCreateDialog(int id) {
		AlertDialog.Builder builder = new AlertDialog.Builder(this);
		if( id == R.id.about ) {
			builder.setMessage('Developed by Leonel Machava ')
				.setPositiveButton('OK', null);
		}
		return builder.create();
	}
	@Override
	public boolean onOptionsItemSelected(MenuItem item) {
		if( item.getItemId() == R.id.about ) {
			showDialog(R.id.about);
		}
		return true;
	}
}
{% endhighlight %}

##Resource Files

**Layout: main.xml**

{% highlight xml %}
<?xml version='1.0' encoding='utf-8'?>
<LinearLayout xmlns:android='http://schemas.android.com/apk/res/android'
    android:orientation='vertical'
    android:layout_width='fill_parent'
    android:layout_height='fill_parent'
    >
<com.codentronix.android.CubeView
    android:layout_width='fill_parent'
    android:layout_height='fill_parent'
    />
</LinearLayout>
{% endhighlight %}

**Menu file: menu.xml**
{% highlight xml %}
<?xml version='1.0' encoding='utf-8'?>
<menu
  xmlns:android='http://schemas.android.com/apk/res/android'>
	<item   android:id='@+id/about'
			android:title='Sobre' />
</menu>
{% endhighlight %}