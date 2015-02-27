
### 栅栏状的图形

![img](https://github.com/chenxiao07/pixel-shader-collection/blob/master/shader/strip.png)


glsl代码

```
float stripes(vec2 p, float steps) {
  return fract(p.x*steps);
}

void main() {
  vec2 p = position - vec2(0.5, 0.5);
  
  float color = stripes(p, 10.0);
  
  gl_FragColor.rgb = vec3(color);
  gl_FragColor.a = 1.;
}
```

### 棋盘格

![img](https://github.com/chenxiao07/pixel-shader-collection/blob/master/shader/checkboard.png)


glsl代码

```
float checkboard(vec2 p, float steps) {
  vec2 temp = fract(p * steps);
  temp = step(temp, vec2(0.5, 0.5));
  return abs(temp.x + temp.y - 1.0);
}

void main() {
  vec2 p = position - vec2(0.5, 0.5);
  
  float color = checkboard(p, 10.0);
  
  gl_FragColor.rgb = vec3(color);
  gl_FragColor.a = 1.;
}
```

### 白噪音

![img](https://github.com/chenxiao07/pixel-shader-collection/blob/master/shader/noise1.png)


glsl代码

```
float rand(vec2 co){
    return fract(sin(dot(co.xy ,vec2(12.9898,78.233))) * 43758.5453);
}

void main() {
  vec2 p = position - vec2(0.5, 0.5);
  
  float color = rand(p);
  
  gl_FragColor.rgb = vec3(color);
  gl_FragColor.a = 1.;
}
```

### 正弦曲线

![img](https://github.com/chenxiao07/pixel-shader-collection/blob/master/shader/sin1.png)


glsl代码

```
void main() {
  vec2 p = position - vec2(0.5, 0.5);
  p = p*8.0;
  
  if (abs(sin(p.x) - p.y) < 0.05) {
    gl_FragColor.rgb = vec3(1.0, 0.0, 0.0);
  } else {
    gl_FragColor.rgb = vec3(0.0, 0.0, 0.0);
  }
  gl_FragColor.a = 1.0;
}
```


### 正弦曲线AA

![img](https://github.com/chenxiao07/pixel-shader-collection/blob/master/shader/sin2.png)


glsl代码

```
void main() {
  vec2 p = position - vec2(0.5, 0.5);
  p = p*8.0;
  
  float value = abs(sin(p.x) - p.y);
  
  gl_FragColor.rgb = vec3(smoothstep(0.05, 0.0, value), 0.0, 0.0);

  gl_FragColor.a = 1.0;
}
```

