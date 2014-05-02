---
layout: post
title: Rotating Wireframe Cube using VB.NET and GDI+
categories:
- Computer Graphics
- VB.NET
tags:
- 3d
- GDI+
- simulation
- vb.net
- visual basic
status: publish
type: post
published: true
meta:
  og_video: http://www.youtube.com/v/K7hwscgxz40
  views: '7022'
  _oembed_2ccc3a04862d9d5962d797b285f63438: <object width="640" height="505"><param
    name="movie" value="http://www.youtube.com/v/K7hwscgxz40?version=3"></param><param
    name="allowFullScreen" value="true"></param><param name="allowscriptaccess" value="always"></param><embed
    src="http://www.youtube.com/v/K7hwscgxz40?version=3" type="application/x-shockwave-flash"
    width="640" height="505" allowscriptaccess="always" allowfullscreen="true"></embed></object>
  _oembed_f81b82e556113fa1f18fb6b649298a44: <object width="400" height="300"><param
    name="movie" value="http://www.youtube.com/v/K7hwscgxz40?version=3"></param><param
    name="allowFullScreen" value="true"></param><param name="allowscriptaccess" value="always"></param><embed
    src="http://www.youtube.com/v/K7hwscgxz40?version=3" type="application/x-shockwave-flash"
    width="400" height="300" allowscriptaccess="always" allowfullscreen="true"></embed></object>
  _oembed_904239d1755fdc2895ff831d7984a539: <iframe width="640" height="480" src="http://www.youtube.com/embed/K7hwscgxz40?feature=oembed"
    frameborder="0" allowfullscreen></iframe>
author:
  login: admin
  email: leonelmachava@gmail.com
  display_name: lefam
  first_name: Leonel
  last_name: Machava
---

Today I will show you how to simulate the rotation of a wireframe cube using VB.NET and GDI+. I assume the reader has at least a basic understanding of Visual Basic. See below a video of the wireframe cube in action.


http://www.youtube.com/watch?v=K7hwscgxz40

<iframe width="640" height="480" src="http://www.youtube.com/embed/K7hwscgxz40?feature=oembed"
    frameborder="0" allowfullscreen></iframe>

So, what is *GDI+*? GDI+ is an API that consists of a set of classes that enable applications to draw graphics on video displays and printers. GDI+ is defined in the **System.Drawing** namespace.

I have created this simulation using Visual Studio Express 2010 which is freely downloadable from Microsoft. However, you can use any IDE for .NET of your choice.

THE FULL SOURCE CODE IS <a href="http://goo.gl/mkaSP">HERE</a>.

##The Code

First of all, create the **Point3D** class. This class will represent points in 3D space. Below is the code of  for the class. For better organization, save it in a file named Point3D.vb.

{% highlight vb.net %}
'
' Defines the Point3D class that represents points in 3D space.
' Developed by leonelmachava <leonelmachava@gmail.com>
' http://codentronix.com
'
' Copyright (c) 2011 Leonel Machava
'
Public Class Point3D
    Protected m_x As Double, m_y As Double, m_z As Double
    Public Sub New(ByVal x As Double, ByVal y As Double, ByVal z As Double)
        Me.X = x
        Me.Y = y
        Me.Z = z
    End Sub
    Public Property X() As Double
        Get
            Return m_x
        End Get
        Set(ByVal value As Double)
            m_x = value
        End Set
    End Property
    Public Property Y() As Double
        Get
            Return m_y
        End Get
        Set(ByVal value As Double)
            m_y = value
        End Set
    End Property
    Public Property Z() As Double
        Get
            Return m_z
        End Get
        Set(ByVal value As Double)
            m_z = value
        End Set
    End Property
    Public Function RotateX(ByVal angle As Integer) As Point3D
        Dim rad As Double, cosa As Double, sina As Double, yn As Double, zn As Double
        rad = angle * Math.PI / 180
        cosa = Math.Cos(rad)
        sina = Math.Sin(rad)
        yn = Me.Y * cosa - Me.Z * sina
        zn = Me.Y * sina + Me.Z * cosa
        Return New Point3D(Me.X, yn, zn)
    End Function
    Public Function RotateY(ByVal angle As Integer) As Point3D
        Dim rad As Double, cosa As Double, sina As Double, Xn As Double, Zn As Double
        rad = angle * Math.PI / 180
        cosa = Math.Cos(rad)
        sina = Math.Sin(rad)
        Zn = Me.Z * cosa - Me.X * sina
        Xn = Me.Z * sina + Me.X * cosa
        Return New Point3D(Xn, Me.Y, Zn)
    End Function
    Public Function RotateZ(ByVal angle As Integer) As Point3D
        Dim rad As Double, cosa As Double, sina As Double, Xn As Double, Yn As Double
        rad = angle * Math.PI / 180
        cosa = Math.Cos(rad)
        sina = Math.Sin(rad)
        Xn = Me.X * cosa - Me.Y * sina
        Yn = Me.X * sina + Me.Y * cosa
        Return New Point3D(Xn, Yn, Me.Z)
    End Function
    Public Function Project(ByVal viewWidth As Integer, ByVal viewHeight As Integer, ByVal fov As Integer, ByVal viewDistance As Integer)
        Dim factor As Double, Xn As Double, Yn As Double
        factor = fov / (viewDistance + Me.Z)
        Xn = Me.X * factor + viewWidth / 2
        Yn = Me.Y * factor + viewHeight / 2
        Return New Point3D(Xn, Yn, Me.Z)
    End Function
End Class
{% endhighlight %}

The methods **RotateX()**, **RotateY()**, and **RotateZ()**, rotate a point, respectively, around X, Y and Z axis. The method **Project(), **transforms a point from 3D space to 2D space. This is fundamental in order to draw a 3D point in our 2D screens.

You should not bother with the details of the implementation of these methods because they are just based on formulas that can be easily found on the internet.

Next, create a Windows Form, and name it Main. After that, copy the code below to the form.

{% highlight vb.net %}
'
' Simulation of a Wireframe Cube using GDI+
' Developed by leonelmachava <leonelmachava@gmail.com>
' http://codentronix.com
'
' Copyright (c) 2011 Leonel Machava
'
Imports System.Drawing.Graphics
Imports System.Drawing.Pen
Imports System.Drawing.Color
Imports System.Drawing.Brush
Imports System.Drawing.Point
Public Class Main
    Protected m_pen As Pen
    Protected m_timer As Timer
    Protected m_vertices(8) As Point3D
    Protected m_faces(6, 4) As Integer
    Protected m_angle As Integer
    Private Sub Main_Load(ByVal sender As System.Object, ByVal e As System.EventArgs) Handles MyBase.Load
        ' Create a GDI+ Pen. This will be used to draw lines.
        m_pen = New Pen(Color.Red)
        InitCube()
        ' Create the timer.
        m_timer = New Timer()
        ' Set the timer interval to 33 milliseconds. This will give us 1000/34 ~ 30 frames per second.
        m_timer.Interval = 33
        ' Set the callback for the timer.
        AddHandler m_timer.Tick, AddressOf AnimationLoop
        ' Start the timer.
        m_timer.Start()
    End Sub
    Private Sub InitCube()
        ' Create an array with 8 points.
        m_vertices = New Point3D() {
                     New Point3D(-1, 1, -1),
                     New Point3D(1, 1, -1),
                     New Point3D(1, -1, -1),
                     New Point3D(-1, -1, -1),
                     New Point3D(-1, 1, 1),
                     New Point3D(1, 1, 1),
                     New Point3D(1, -1, 1),
                     New Point3D(-1, -1, 1)}
        ' Create an array representing the 6 faces of a cube. Each face is composed by indices to the vertex array
        ' above.
        {% raw %}
        m_faces = New Integer(,) {{0, 1, 2, 3}, {1, 5, 6, 2}, {5, 4, 7, 6}, {4, 0, 3, 7}, {0, 4, 5, 1}, {3, 2, 6, 7}}
        {% endraw %}
    End Sub
    Private Sub AnimationLoop()
        ' Forces the Paint event to be called.
        Me.Invalidate()
        ' Update the variable after each frame.
        m_angle += 1
    End Sub
    Private Sub Main_Paint(ByVal sender As Object, ByVal e As System.Windows.Forms.PaintEventArgs) Handles Me.Paint
        Dim t(8) As Point3D
        Dim f(4) As Integer
        Dim v As Point3D
        ' Clear the window
        e.Graphics.Clear(Color.LightBlue)
        ' Transform all the points and store them on the "t" array.
        For i = 0 To 7
            v = m_vertices(i)
            t(i) = v.RotateX(m_angle).RotateY(m_angle).RotateZ(Me.m_angle)
            t(i) = t(i).Project(Me.ClientSize.Width, Me.ClientSize.Height, 256, 4)
        Next
        ' Draw the wireframe cube. Uses the "m_faces" array to find the vertices that compose each face.
        For i = 0 To 5
            e.Graphics.DrawLine(m_pen, CInt(t(m_faces(i, 0)).X), CInt(t(m_faces(i, 0)).Y), CInt(t(m_faces(i, 1)).X), CInt(t(m_faces(i, 1)).Y))
            e.Graphics.DrawLine(m_pen, CInt(t(m_faces(i, 1)).X), CInt(t(m_faces(i, 1)).Y), CInt(t(m_faces(i, 2)).X), CInt(t(m_faces(i, 2)).Y))
            e.Graphics.DrawLine(m_pen, CInt(t(m_faces(i, 2)).X), CInt(t(m_faces(i, 2)).Y), CInt(t(m_faces(i, 3)).X), CInt(t(m_faces(i, 3)).Y))
            e.Graphics.DrawLine(m_pen, CInt(t(m_faces(i, 3)).X), CInt(t(m_faces(i, 3)).Y), CInt(t(m_faces(i, 0)).X), CInt(t(m_faces(i, 0)).Y))
        Next
    End Sub
End Class
{% endhighlight %}

When the form is loaded, we first create a Pen that is later used to draw the lines of the cube. Next, we call *InitCube()* to initialize data about the cube. Finally, we create and setup a timer to drive the animation at 30 frames per second.

The *AnimationLoop()* method handles the timer event, triggering a Paint event each time it is invoked. *Main_Paint()* rotates and draws the wireframe cube while handling the Paint event.

THE FULL SOURCE CODE IS <a href="http://goo.gl/mkaSP">HERE</a>.

##Conclusion

Today, we have seen how to simulate a rotating wireframe cube. In the next tutorial, I will show you how to adapt this code to simulate a rotating solid cube.
