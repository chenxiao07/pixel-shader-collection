
### 栅栏状的图形

![img](file:///Users/JP20186/Documents/shader/strip.png)


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

![img](file:///Users/JP20186/Documents/shader/checkboard.png)


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


