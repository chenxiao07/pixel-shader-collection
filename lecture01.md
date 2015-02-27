
### 纯颜色shader

![img](file:///Users/JP20186/Documents/shader/red.png)


glsl代码

```
void main() {
  gl_FragColor.r = 1.0;
  gl_FragColor.g = 0.0;
  gl_FragColor.b = 0.0;
  gl_FragColor.a = 1.0;
}
```



### 带渐变的shader

![img](file:///Users/JP20186/Documents/shader/gradient.png)


glsl代码

```
void main() {
  gl_FragColor.r = position.x;
  gl_FragColor.g = 0.0;
  gl_FragColor.b = 0.0;
  gl_FragColor.a = 1.0;
}
```

### 两个方向渐变的shader

![img](file:///Users/JP20186/Documents/shader/gradient2.png)


glsl代码

```
void main() {
  gl_FragColor.r = position.x;
  gl_FragColor.g = position.y;
  gl_FragColor.b = 0.0;
  gl_FragColor.a = 1.0;
}
```

### 径向渐变的shader

![img](file:///Users/JP20186/Documents/shader/gradient3.png)

glsl代码

```
void main() {
  float r = length(position - vec2(0.5, 0.5));
  gl_FragColor.r = r;
  gl_FragColor.g = r;
  gl_FragColor.b = r;
  gl_FragColor.a = 1.0;
}
```

### 径向渐变的shader

![img](file:///Users/JP20186/Documents/shader/gradient4.png)

glsl代码

```
void main() {
  vec2 p = position - vec2(0.5, 0.5);
  
  float radius = length(p);
  float angle = atan(p.y, p.x);
  
  gl_FragColor.r = radius;
  gl_FragColor.g = 0.0;
  gl_FragColor.b = abs(angle / 3.14159);
  gl_FragColor.a = 1.0;
}
```