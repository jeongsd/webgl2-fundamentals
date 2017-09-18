Title: WebGL - 3D 직교 투영
Description: 직교 투영을 하는 WebGL에서 3D를 그리는 방법.

이 글은 몇 가지 WebGL 글에서 이어지는 글입니다.
첫번째는 [기초로 시작하기](webgl-fundamentals.html)이며 이 전 글은 [2D 행렬에 대하여](webgl-2d-matrices.html)입니다. 
만약 아직 이 글들을 읽지 않았다면 먼저 읽는 것을 권장합니다.

지난번 글에서 2D 행렬이 어떻게 작동하는지 살펴 보았습니다. 
우리는 한개의 행렬과 마법같은 행렬 수학으로 이동, 회전, 크기 그리고 픽셀에서 클립공간으로 투영하는 방법에 대하여 알아보았습니다. 
3D를 하기 위해서는 여기서 조금만 더 나아가면 됩니다.

이전 2D 예제에서 우리는 3x3 매트릭스로 곱한 2D 포인트 (x, y)를 다루었습니다.
3D에서는 점 (x, y, z)과 4x4 행렬이 필요합니다.

마지막 예제를 3D로 변경해 봅시다. F를 다시 사용하지만 이번에는 3D 'F'를 사용합니다.

첫 번쨰 할일은 버텍스 쉐이더가 3D를 처리하도록 변경하는 것입니다.
여기에 예전 버텍스 쉐이더가 있습니다.

```
#version 300 es

// 버텍스 쉐이더로 입력되는 attribute입니다.
// 버퍼로부터 데이터를 받습니다.
in vec2 a_position;

// 위치를 변환하는 행렬
uniform mat3 u_matrix;

// 모든 쉐이더는 main함수를 가지고 있습니다.
void main() {
  // 행렬에 위치를 곱합니다.
  gl_Position = vec4((u_matrix * vec3(a_position, 1)).xy, 0, 1);
}
```

여기에 새로운 쉐이더가 있습니다.

```
// 버텍스 쉐이더로 입력되는 attribute입니다.
// 버퍼로부터 데이터를 받습니다.
in vec2 a_position;

// 위치를 변환하는 행렬
uniform mat3 u_matrix;

// 모든 쉐이더는 main함수를 가지고 있습니다.
void main() {
  // 행렬에 위치를 곱합니다.
*  gl_Position = u_matrix * a_position;
}
```

더 간단 해졌습니다! `x`와`y`를 제공하고`z`를 1로 설정했던 2 차원과 마찬가지로, 
3d에서는 `x`,`y`와`z`를 제공하고`w`가 1이 되어야합니다. 
`w` 속성의 기본값은 1이라는 사실을 이용할 수 있습니다.

그 다음 3D 데이터를 제공해야합니다.

```
  ...

  // positionBuffer (ARRAY_BUFFER)에서 데이터를 가져 오는 방법을 attribute에 알립니다.
*  var size = 3;          // 반복마다 3개 구성요소
  var type = gl.FLOAT;   // 데이터는 32비트 부동소수점
  var normalize = false; // 정규화를 하지 않음
  var stride = 0;        // 0 = 반복마다 size & sizeof(type) 만큼 다음으로 이동
  var offset = 0;        // 버퍼의 시작 지점
  gl.vertexAttribPointer(
      positionAttributeLocation, size, type, normalize, stride, offset);

  ...

  // 현재 ARRAY_BUFFER 버퍼를 'F' 글자를 정의하는 값들로 채웁니다.
  function setGeometry(gl) {
    gl.bufferData(
        gl.ARRAY_BUFFER,
        new Float32Array([
            // 왼쪽 열
              0,   0,  0,
             30,   0,  0,
              0, 150,  0,
              0, 150,  0,
             30,   0,  0,
             30, 150,  0,

            // top rung
             30,   0,  0,
            100,   0,  0,
             30,  30,  0,
             30,  30,  0,
            100,   0,  0,
            100,  30,  0,

            // middle rung
             30,  60,  0,
             67,  60,  0,
             30,  90,  0,
             30,  90,  0,
             67,  60,  0,
             67,  90,  0]),
        gl.STATIC_DRAW);
  }
```

다음은 모든 행렬 함수를 2D에서 3D로 변경하는 것이 필요합니다.

여기에 2D(이전) 버전 m3.translation, m3.rotation 그리고 m3.scaling 함수가 있습니다.

```
var m3 = {
  translation: function translation(tx, ty) {
    return [
      1, 0, 0,
      0, 1, 0,
      tx, ty, 1
    ];
  },

  rotation: function rotation(angleInRadians) {
    var c = Math.cos(angleInRadians);
    var s = Math.sin(angleInRadians);
    return [
      c,-s, 0,
      s, c, 0,
      0, 0, 1
    ];
  },

  scaling: function scaling(sx, sy) {
    return [
      sx, 0, 0,
      0, sy, 0,
      0, 0, 1
    ];
  },
};
```

그리고 여기에 업데이트된 3D 버전이 있습니다.

```
var m4 = {
  translation: function(tx, ty, tz) {
    return [
       1,  0,  0,  0,
       0,  1,  0,  0,
       0,  0,  1,  0,
       tx, ty, tz, 1,
    ];
  },

  xRotation: function(angleInRadians) {
    var c = Math.cos(angleInRadians);
    var s = Math.sin(angleInRadians);

    return [
      1, 0, 0, 0,
      0, c, s, 0,
      0, -s, c, 0,
      0, 0, 0, 1,
    ];
  },

  yRotation: function(angleInRadians) {
    var c = Math.cos(angleInRadians);
    var s = Math.sin(angleInRadians);

    return [
      c, 0, -s, 0,
      0, 1, 0, 0,
      s, 0, c, 0,
      0, 0, 0, 1,
    ];
  },

  zRotation: function(angleInRadians) {
    var c = Math.cos(angleInRadians);
    var s = Math.sin(angleInRadians);

    return [
       c, s, 0, 0,
      -s, c, 0, 0,
       0, 0, 1, 0,
       0, 0, 0, 1,
    ];
  },

  scaling: function(sx, sy, sz) {
    return [
      sx, 0,  0,  0,
      0, sy,  0,  0,
      0,  0, sz,  0,
      0,  0,  0,  1,
    ];
  },
};
```

여기서 주목할것은 3개의 회전 함수가 있다는 것 입니다. 2D에서는 Z축을 기준으로 하는 한개의 회전만 있엇습니다. 이제 3D에서는 X축 또는 Y축 회전 또한 존재합니다. 모두 비슷한 모양을 가지고 있다는것을 볼 수 있습니다. 어떻게 작동하는지 알아 본다면 이전처럼 간단하게 보일 것입니다.

Z축 회전

<div class="webgl_center">
<div>newX = x *  c + y * s;</div>
<div>newY = x * -s + y * c;</div>
</div>

Y축 회전

<div class="webgl_center">
<div>newX = x *  c + z * s;</div>
<div>newZ = x * -s + z * c;</div>
</div>

X축 회전

<div class="webgl_center">
<div>newY = y *  c + z * s;</div>
<div>newZ = y * -s + z * c;</div>
</div>

이는 다음과 같은 회전을 제공합니다.

<iframe class="external_diagram" src="resources/axis-diagram.html" style="width: 540px; height: 240px;"></iframe>

마찬가지로 단순화 된 함수를 만들 것입니다.

```
  translate: function(m, tx, ty, tz) {
    return m4.multiply(m, m4.translation(tx, ty, tz));
  },

  xRotate: function(m, angleInRadians) {
    return m4.multiply(m, m4.xRotation(angleInRadians));
  },

  yRotate: function(m, angleInRadians) {
    return m4.multiply(m, m4.yRotation(angleInRadians));
  },

  zRotate: function(m, angleInRadians) {
    return m4.multiply(m, m4.zRotation(angleInRadians));
  },

  scale: function(m, sx, sy, sz) {
    return m4.multiply(m, m4.scaling(sx, sy, sz));
  },
```

투영 함수도 업데이트를 해야합니다. 여기에 예전 버전이 있습니다.

```
  projection: function (width, height) {
    // Note: 이 행렬은 Y축을 뒤집어서 0이 맨위가 되도록 합니다.
    return [
      2 / width, 0, 0,
      0, -2 / height, 0,
      -1, 1, 1
    ];
  },
}
```

이 함수는 픽셀에서 클립공간으로 변환합니다. 3D로 확장하기 위한 첫번쨰 시도를 해봅시다.

```
  projection: function(width, height, depth) {
    // Note: 이 행렬은 Y축을 뒤집어서 0이 맨위가 되도록 합니다.
    return [
       2 / width, 0, 0, 0,
       0, -2 / height, 0, 0,
       0, 0, 2 / depth, 0,
      -1, 1, 0, 1,
    ];
  },
```

픽셀에서 클립 공간으로 X와 Y를 변환해야하는 것처럼 Z에도 똑같은 작업을 수행해야합니다. 이 경우에는 Z축 픽셀 단위를 만들어야 합니다. `depth`에 `width`와 비슷 한 값을 전달할 것입니다. 그래서 공간은 0에서 `width` 픽셀이 너비가 되고, 0에서 `height` 픽셀은 높이가 되며 `depth`는 `-depth / 2 '에서 `+ depth / 2'로 됩니다.

Just like we needed to convert from pixels to clip space for X and Y, for
Z we need to do the same thing.  In this case I'm making the Z axis pixel
units as well.  I'll pass in some value similar to `width` for the `depth`
so our space will be 0 to `width` pixels wide, 0 to `height` pixels tall, but
for `depth` it will be `-depth / 2` to `+depth / 2`.

마지막으로 행렬을 계산하는 코드를 업데이트 해야 합니다.

```
  // 행렬 계산
*  var matrix = m4.projection(gl.canvas.clientWidth, gl.canvas.clientHeight, 400);
*  matrix = m4.translate(matrix, translation[0], translation[1], translation[2]);
*  matrix = m4.xRotate(matrix, rotation[0]);
*  matrix = m4.yRotate(matrix, rotation[1]);
*  matrix = m4.zRotate(matrix, rotation[2]);
*  matrix = m4.scale(matrix, scale[0], scale[1], scale[2]);

  // 행렬 설정
*  gl.uniformMatrix4fv(matrixLocation, false, matrix);
```

여기에 예제가 있습니다.

{{{example url="../webgl-3d-step1.html" }}}

여기에 있는 첫번쨰 문제는 지오메트리가 평면 F라서 3D로 보이기 어렵다는 것입니다. 
이를 고치기 위해 지오메트리를 3D로 확장해 보겠습니다. 현재 F는 각각 2개의 삼각형인 3개의 직사각형으로 구성되어 있습니다. 
3D로 만들기 위해서는 16개의 직사각형이 필요합니다. 3개의 직사각형은 앞면, 3개는 뒷면, 왼쪽면에 1개, 오른쪽면에 4개 윗면에 2개, 아랫면에 3개가 필요합니다.

<img class="webgl_center" width="300" src="resources/3df.svg" />

여기에 적어 놓은 것은 꽤나 적지만 만약 모든 예제 소스를 보기를 원한다면 3개의 버텍스인 2개의 삼각형으로 이루워진 16개의 직사각형은 96개의 점을 가집니다. 

더 많은 버텍스들을 그려야 합니다.

```
    // 지오메트리 그리기
    var primitiveType = gl.TRIANGLES;
    var offset = 0;
*    var count = 16 * 6;
    gl.drawArrays(primitiveType, offset, count);
```

여기에 새로운 버전이 있습니다.

{{{example url="../webgl-3d-step2.html" }}}

이제 슬라이더를 움직여 보면 3D라고 말할 수 있습니다. 각 사각형에 다른 색갈을 칠해 봅시다. 이를 하기 위해서 버텍스 쉐이더에 다른 attribute를 추가를 하고 varying을 버텍스 쉐이더에서 프래그먼트 쉐이더로 전달 해야합니다.

여기에 새로운 버텍스 쉐이더가 있습니다.

```
#version 300 es

// attribute는 버텍스 쉐이더에 입력입니다.
// 버퍼로 부터 데이터를 받습니다.
in vec4 a_position;
+in vec4 a_color;

// 위치를 변환하는 행렬
uniform mat4 u_matrix;

+// 프래그먼트 쉐이더로 전달할 색상 varying
+out vec4 v_color;

// 모든 쉐이더는 main함수를 가지고 있습니다.
void main() {
  // 위치에 행렬을 곱합니다.
  gl_Position = u_matrix * a_position;

+  // 프래그먼트 쉐이더로 색상을 전달 합니다.
+  v_color = a_color;
}
```

그런 다음 프래그먼트 쉐이더에서 색상을 사용해야 합니다.

```
#version 300 es

precision mediump float;

+// 버텍스 쉐이더에서 전달된 다양한 색상들
+in vec4 v_color;

// 프래그먼트 쉐이더에서 출력할 값을 선언해야 합니다.
out vec4 outColor;

void main() {
*  outColor = v_color;
}
```

색상을 제공 하기 위해서 attribute의 위치를 찾은 다음 다른 버퍼 설정하고 attribute에 색상을 제공하면 됩니다.

```
  ...
  var colorAttributeLocation = gl.getAttribLocation(program, "a_color");

  ...

  // 색상 버퍼를 생성하고 ARRAY_BUFFER로 만든 다음 색상값을 복사합니다.
  var colorBuffer = gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER, colorBuffer);
  setColors(gl);

  // attribute 켜기
  gl.enableVertexAttribArray(colorAttributeLocation);

  // attribute에 colorBuffer (ARRAY_BUFFER)부터 데이터를 가져오는 방법을 전달합니다.
  var size = 3;          // 반복당 3개의 구성요소
  var type = gl.UNSIGNED_BYTE;   // 데이터는 8 비트 부호없는 바이트입니다.
  var normalize = true;  // 0-255에서 0.0-1.0로 변환
  var stride = 0;        // 0 = 반복마다 size * sizeof(type)만큼 이동해서 다음 색상을 얻습니다.
  var offset = 0;        // 버퍼의 시작부분에서 시작
  gl.vertexAttribPointer(
      colorAttributeLocation, size, type, normalize, stride, offset);

  ...

// 버퍼에 'F' 색상을 채웁니다.

function setColors(gl) {
  gl.bufferData(
      gl.ARRAY_BUFFER,
      new Uint8Array([
          // 왼쪽 앞 칸
        200,  70, 120,
        200,  70, 120,
        200,  70, 120,
        200,  70, 120,
        200,  70, 120,
        200,  70, 120,

          // top rung front
        200,  70, 120,
        200,  70, 120,
        ...
        ...
      gl.STATIC_DRAW);
}
```

그런 다음 다음을 얻을수 있습니다.

{{{example url="../webgl-3d-step3.html" }}}

음 무엇이 이상할까요? 음, 3D 'F'의 앞면, 뒷면, 측면 등의 다양한 부분이 우리의 지오메트리 데이터에 나타나는 순서대로 그려집니다. 이는 뒷면이 앞면이 그려진 다음에 그려질수 있기 떄문에 우리가 원하는 결과를 얻지 못할수도 있습니다.

<img class="webgl_center" width="163" height="190" src="resources/polygon-drawing-order.gif" />

 <span style="background: rgb(200, 70, 120); color: white; padding: 0.25em">붉게된 부분는</span> **앞면**이지만 데이터의 첫번째 부분이기 때문에 첫번쨰로 그려지기 때문에 다른 삼각형에 가려지게 됩니다. 예를 들어 <span style="background: rgb(80, 70, 200); color: white; padding: 0.25em">보라색 부분</span>은 실제로 'F'의 뒷면 입니다. 데이터 순서가 2번째 이기 떄문에 2번째에 그려집니다.

Triangles in WebGL have the concept of front facing and back facing.  A
front facing triangle has its vertices go in a clockwise direction.  A
back facing triangle has its vertices go in a counter clockwise direction

<img src="resources/triangle-winding.svg" class="webgl_center" width="400" />

WebGL has the ability to draw only forward facing or back facing
triangles.  We can turn that feature on with

```
  gl.enable(gl.CULL_FACE);
```

Well put that in our `drawScene` function. With that
feature turned on, WebGL defaults to "culling" back facing triangles.
"Culling" in this case is a fancy word for "not drawing".

Note that as far as WebGL is concerned, whether or not a triangle is
considered to be going clockwise or counter clockwise depends on the
vertices of that triangle in clip space.  In other words, WebGL figures out
whether a triangle is front or back AFTER you've applied math to the
vertices in the vertex shader.  That means for example a clockwise
triangle that is scaled in X by -1 becomes a counter clockwise triangle or
a clockwise triangle rotated 180 degrees becomes a couter clockwise
triangle.  Because we had CULL_FACE disabled we can see both
clockwise(front) and counter clockwise(back) triangles.  Now that we've
turned it on, any time a front facing triangle flips around either because
of scaling or rotation or for whatever reason, WebGL won't draw it.
That's a good thing since as your turn something around in 3D you
generally want whichever triangles are facing you to be considered front
facing.

With CULL_FACE turned on this is what we get

{{{example url="../webgl-3d-step4.html" }}}

Hey!  Where did all the triangles go?  It turns out, many of them are
facing the wrong way.  Rotate it and you'll see them appear when you look
at the other side.  Fortunately it's easy to fix.  We just look at which
ones are backward and exchange 2 of their vertices.  For example if one
backward triangle is

```
           1,   2,   3,
          40,  50,  60,
         700, 800, 900,
```

we just swap the last 2 vertices to make it forward.

```
           1,   2,   3,
*         700, 800, 900,
*          40,  50,  60,
```

Going through and fixing all the backward triangles gets us to this

{{{example url="../webgl-3d-step5.html" }}}

That's closer but there's still one more problem.  Even with all the
triangles facing in the correct direction and with the back facing ones
being culled we still have places where triangles that should be in the back
are being drawn over triangles that should be in front.

Enter the DEPTH BUFFER.

A depth buffer, sometimes called a Z-Buffer, is a rectangle of *depth*
pixels, one depth pixel for each color pixel used to make the image.  As
WebGL draws each color pixel it can also draw a depth pixel.  It does this
based on the values we return from the vertex shader for Z.  Just like we
had to convert to clip space for X and Y, Z is also in clip space or (-1
to +1).  That value is then converted into a depth space value (0 to +1).
Before WebGL draws a color pixel it will check the corresponding depth
pixel.  If the depth value for the pixel it's about to draw is greater
than the value of the corresponding depth pixel then WebGL does not draw
the new color pixel.  Otherwise it draws both the new color pixel with the
color from your fragment shader AND it draws the depth pixel with the new
depth value.  This means, pixels that are behind other pixels won't get
drawn.

We can turn on this feature nearly as simply as we turned on culling with

```
  gl.enable(gl.DEPTH_TEST);
```


We also need to clear the depth buffer back to 1.0 before we start drawing.

```
  // Draw the scene.
  function drawScene() {

    ...

    // Clear the canvas AND the depth buffer.
    gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);

    ...
```

And now we get

{{{example url="../webgl-3d-step6.html" }}}

which is 3D!

One minor thing. In most 3d math libraries there is no `projection` function to
do our conversions from clip space to pixel space. Rather there's usually a function
called `ortho` or `orthographic` that looks like this

    var m4 = {
      orthographic: function(left, right, bottom, top, near, far) {
        return [
          2 / (right - left), 0, 0, 0,
          0, 2 / (top - bottom), 0, 0,
          0, 0, 2 / (near - far), 0,

          (left + right) / (left - right),
          (bottom + top) / (bottom - top),
          (near + far) / (near - far),
          1,
        ];
      }

Unlike our simplified `projection` function above which only had width, height, and depth
parameters this more common othrographic projection function we can pass in left, right,
bottom, top, near, and far which gives as more flexability. To use it the same as
our original projection function we'd call it with

    var left = 0;
    var right = gl.canvas.clientWidth;
    var bottom = gl.canvas.clientHeight;
    var top = 0;
    var near = -400;
    var far = 400;
    m4.orthographic(left, right, bottom, top, near, far);

In the next post I'll go over [how to make it have perspective](webgl-3d-perspective.html).

<div class="webgl_bottombar">
<h3>Why is the attribute vec4 but gl.vertexAttribPointer size 3</h3>
<p>
For those of you who are detail oriented you might have noticed we defined our 2 attributes as
</p>
<pre class="prettyprint showlinemods">
in vec4 a_position;
in vec4 a_color;
</pre>
<p>both of which are 'vec4' but when we tell WebGL how to take data out of our buffers we used</p>
<pre class="prettyprint showlinemods">
// Tell the attribute how to get data out of positionBuffer (ARRAY_BUFFER)
var size = 3;          // 3 components per iteration
var type = gl.FLOAT;   // the data is 32bit floats
var normalize = false; // don't normalize the data
var stride = 0;        // 0 = move forward size * sizeof(type) each
                       // iteration to get the next position
var offset = 0;        // start at the beginning of the buffer
gl.vertexAttribPointer(
    positionAttributeLocation, size, type, normalize, stride, offset);

...
// Tell the attribute how to get data out of colorBuffer (ARRAY_BUFFER)
var size = 3;          // 3 components per iteration
var type = gl.UNSIGNED_BYTE;   // the data is 8bit unsigned bytes
var normalize = true;  // convert from 0-255 to 0.0-1.0
var stride = 0;        // 0 = move forward size * sizeof(type) each
                       // iteration to get the next color
var offset = 0;        // start at the beginning of the buffer
gl.vertexAttribPointer(
    colorAttributeLocation, size, type, normalize, stride, offset);
</pre>
<p>
That '3' in each of those says only to pull 3 values out of the buffer per attribute
per iteration of the vertex shader.
This works because in the vertex shader WebGL provides defaults for those
values you don't supply.  The defaults are 0, 0, 0, 1 where x = 0, y = 0, z = 0
and w = 1.  This is why in our old 2D vertex shader we had to explicitly
supply the 1.  We were passing in x and y and we needed a 1 for z but
because the default for z is 0 we had to explicitly supply a 1.  For 3D
though, even though we don't supply a 'w' it defaults to 1 which is what
we need for the matrix math to work.
</p>
</div>
