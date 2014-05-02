---
layout: post
title: ! 'HTML5 Experiment: Rotating Cube with Flat Shading'
categories:
- Computer Graphics
- HTML5
- JavaScript
tags:
- 3d
- experiment
- html5 demo
- javascript
- simulation
status: draft
type: post
published: false
meta:
  og_image: http://codentronix.com/projects/html5/html5-logo.png
  og_description: This articles shows an HTML5 experiment that draws a rotating cube
    in a 2D canvas using "flat shading".
  iframe: <iframe src="http://codentronix.com/projects/html5/demos/cube_flat_shading.html"
    width="420" height="340" frameborder="0"></iframe>
author:
  login: admin
  email: leonelmachava@gmail.com
  display_name: lefam
  first_name: Leonel
  last_name: Machava
---
A few months ago I posted an HTML5 experiment that consisted of a <a title="HTML5 Experiment: A Rotating Solid Cube" href="http://codentronix.com/2011/05/10/html5-experiment-a-rotating-solid-cube/">rotating solid cube in a 2D canvas</a>. Building from the code from that experiment I adapted it to draw the rotating cube using **flat shading**.

<iframe src="http://codentronix.com/projects/html5/demos/cube_flat_shading.html"
    width="420" height="340" frameborder="0"></iframe>
<!--more-->

## Source Code

{% highlight javascript %}
<!DOCTYPE html>
<html>
<head>
    <title>Cubo 3D usando HTML5 e Canvas: Parte 3</title>
    <script type='text/javascript'>
        window.onload = startDemo;
        function Point3D(x,y,z) {
            this.x = x;
            this.y = y;
            this.z = z;
            this.scale = function(k) {
                return new Point3D(x * k,y * k,z * k)
            }
            this.translate = function(tx,ty,tz) {
                return new Point3D(x + tx,y + ty,z + tz)
            }
            this.rotateX = function(angle) {
                var rad, cosa, sina, y, z
                rad = angle * Math.PI / 180
                cosa = Math.cos(rad)
                sina = Math.sin(rad)
                y = this.y * cosa - this.z * sina
                z = this.y * sina + this.z * cosa
                return new Point3D(this.x, y, z)
            }
            this.rotateY = function(angle) {
                var rad, cosa, sina, x, z
                rad = angle * Math.PI / 180
                cosa = Math.cos(rad)
                sina = Math.sin(rad)
                z = this.z * cosa - this.x * sina
                x = this.z * sina + this.x * cosa
                return new Point3D(x,this.y, z)
            }
            this.rotateZ = function(angle) {
                var rad, cosa, sina, x, y
                rad = angle * Math.PI / 180
                cosa = Math.cos(rad)
                sina = Math.sin(rad)
                x = this.x * cosa - this.y * sina
                y = this.x * sina + this.y * cosa
                return new Point3D(x, y, this.z)
            }
            this.project = function(viewWidth, viewHeight, fov) {
                var factor, x, y
                factor = fov / this.z
                x = this.x * factor + viewWidth / 2
                y = -this.y * factor + viewHeight / 2
                return new Point3D(x, y, this.z)
            }
            this.dotProduct = function(v) {
              return this.x * v.x + this.y * v.y + this.z * v.z
            }
            this.length = function() {
              return Math.sqrt(this.x * this.x + this.y * this.y + this.z * this.z)
            }
            this.normalize = function() {
              var len = this.length()
              this.x /= len
              this.y /= len
              this.z /= len
            }
        }
        function Face(i1,i2,i3,i4,normal) {
          this.indices = [i1,i2,i3,i4]
          this.normal  = normal
          // This property will store the normal transformed.
          this.tnormal = normal
        }
        var vertices = [
            new Point3D(-1,1,-1),
            new Point3D(1,1,-1),
            new Point3D(1,-1,-1),
            new Point3D(-1,-1,-1),
            new Point3D(-1,1,1),
            new Point3D(1,1,1),
            new Point3D(1,-1,1),
            new Point3D(-1,-1,1)
        ];
        var faces = [
          new Face(0,1,2,3,new Point3D(0,0,-1)),
          new Face(1,5,6,2,new Point3D(1,0,0)),
          new Face(5,4,7,6,new Point3D(0,0,1)),
          new Face(4,0,3,7,new Point3D(-1,0,0)),
          new Face(0,4,5,1,new Point3D(0,1,0)),
          new Face(3,2,6,7,new Point3D(0,-1,0))
        ];
        // Define the colors for each face.
        var colors = [[255,0,0],[0,255,0],[0,0,255],[255,255,0],[0,255,255],[255,0,255]];
        // Define the view direction
        var viewDir = new Point3D(0,0,1)
        // Define the inverse of the view direction. This will be used to perform
        // backface culling and calculate light intensity.
        var invViewDir = new Point3D(0,0,-1)
        var angle = 0;
        function startDemo() {
            canvas = document.getElementById('tutorial');
            if( canvas &amp;&amp; canvas.getContext ) {
                ctx = canvas.getContext('2d');
                gradient = ctx.createLinearGradient(0,0,0,canvas.height)
                gradient.addColorStop(0,'#009999')
                gradient.addColorStop(1,'#00cccc')
                setInterval(loop,33);
            }
        }
        /* Constructs a CSS RGB value from an array of 3 elements. */
        function arrayToRGB(arr) {
            if( arr.length == 3 ) {
                return 'rgb(' + arr[0] + ',' + arr[1] + ',' + arr[2] + ')';
            }
            return 'rgb(0,0,0)';
        }
        function loop() {
            var t = new Array();
            ctx.fillStyle = gradient;
            ctx.fillRect(0,0,canvas.width,canvas.height);
            for( var i = 0; i < vertices.length; i++ ) {
                var v = vertices[i];
                var r = v.rotateX(angle).rotateY(angle).rotateZ(angle).translate(0,0,4);
                var p = r.project(canvas.width,canvas.height,256);
                t.push(p)
            }
            var avg_z = new Array();
            for( var i = 0; i < faces.length; i++ ) {
                faces[i].tnormal = faces[i].normal.rotateX(angle).rotateY(angle).rotateZ(angle)
                var f = faces[i].indices;
                avg_z[i] = {'index':i, 'z':(t[f[0]].z + t[f[1]].z + t[f[2]].z + t[f[3]].z) / 4.0};
            }
            avg_z.sort(function(a,b) {
                return b.z - a.z;
            });
            for( var i = 0; i < 6; i++ ) {
                var face = faces[avg_z[i].index]
                f = face.indices
                face.tnormal.normalize()
                var dotProd = face.tnormal.dotProduct(invViewDir)
                if( dotProd > 0 ) {
                  var color = new Array(3)
                  color[0] = Math.floor(colors[avg_z[i].index][0] * dotProd)
                  color[1] = Math.floor(colors[avg_z[i].index][1] * dotProd)
                  color[2] = Math.floor(colors[avg_z[i].index][2] * dotProd)
                  ctx.fillStyle = arrayToRGB(color);
                  ctx.beginPath()
                  ctx.moveTo(t[f[0]].x,t[f[0]].y)
                  ctx.lineTo(t[f[1]].x,t[f[1]].y)
                  ctx.lineTo(t[f[2]].x,t[f[2]].y)
                  ctx.lineTo(t[f[3]].x,t[f[3]].y)
                  ctx.closePath()
                  ctx.fill()
                }
            }
            angle += 1
        }
    </script>
</head>
<body>
    <canvas id='tutorial' width='400' height='300'>
      Your browser does not support HTML5 canvas.
      Please upgrade your browser.
    </canvas>
</body>
</html>
[/javascript]
If you like this experiment.
