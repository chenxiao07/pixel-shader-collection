
### 最基础的raymarching

![img](https://github.com/chenxiao07/pixel-shader-collection/blob/master/shader/raymarching_sphere.png)

glsl代码

```glsl

// raymarching 最大迭代次数
#define ITE_MAX      45

// 收敛边界条件
#define DIST_MIN     0.01

// 用距离函数构成的空间图形
// 这个例子里是一个半径为0.5，球心为（0，0，0）的球体
float map(vec3 p) {
  return length(p) - 0.5;
}

void main( void ) {

  vec2 uv = position * 2.0 - 1.0;
  
  // 从视点发出的光线
  vec3  dir = normalize(vec3(uv, 1.0));
  
  // 视点坐标
  vec3 pos = vec3(0.0, 0.0, -1.0);
  
  float t = 0.0;

  for(int i = 0 ; i < ITE_MAX; i++) {
    float ttemp = map(t * dir + pos);
    if(ttemp < DIST_MIN) break;

    t += ttemp;
  }
  
  // 收敛时的坐标，没有收敛时是一个相当大的数值
  vec3 ip = pos + dir * t;
  
  // 这里直接将深度转化为颜色
  vec3 color = vec3(t);
  
  gl_FragColor = vec4(color, 1.0);
}

```


### 两个球

![img](https://github.com/chenxiao07/pixel-shader-collection/blob/master/shader/two_sphere.png)

glsl代码

```glsl

#define ITE_MAX      45
#define DIST_MIN     0.01

// 半径为0.3， 球心分别在（0.3，0，0）和（－0.3，0，0）的两个球体
float map(vec3 p) {
  float sphere1 = length(p - vec3(0.3, 0, 0)) - 0.3;
  float sphere2 = length(p - vec3(-0.3, 0, 0)) - 0.3;
  return min(sphere1, sphere2);
}

void main( void ) {

  vec2 uv = position * 2.0 - 1.0;
  vec3 dir = normalize(vec3(uv, 1.0));
  vec3 pos = vec3(0.0, 0.0, -1.0);
  
  float t = 0.0;

  for(int i = 0 ; i < ITE_MAX; i++) {
    float ttemp = map(t * dir + pos);
    if(ttemp < DIST_MIN) break;

    t += ttemp;
  }
  
  vec3 color = vec3(t);
  
  gl_FragColor = vec4(color, 1.0);
}

```


### 迭代次数球

![img](https://github.com/chenxiao07/pixel-shader-collection/blob/master/shader/ite_count.png)

glsl代码

```glsl

#define ITE_MAX      45
#define DIST_MIN     0.01

float map(vec3 p) {
  return length(p) - 0.5;
}

void main( void ) {

  vec2 uv = position * 2.0 - 1.0;
  vec3 dir = normalize(vec3(uv, 1.0));
  vec3 pos = vec3(0.0, 0.0, -1.0);
  
  float t = 0.0;
  int iteCount = 0;

  for(int i=0; i < ITE_MAX; i++) {
    float ttemp = map(t * dir + pos);
    if(ttemp < DIST_MIN) {
      iteCount = i;
      break;
    }

    t += ttemp * 0.5;
  }
  
  vec3 ip = pos + dir * t;
  float color = float(iteCount)/float(ITE_MAX);
  
  gl_FragColor = vec4( color, 0.0, 0.0, 1.0);
}

```


### 法线方向颜色

![img](https://github.com/chenxiao07/pixel-shader-collection/blob/master/shader/normal_direction.png)

glsl代码


```glsl

#define ITE_MAX      45
#define DIST_MIN     0.01

float map(vec3 p) {
  return length(p) - 0.5;
}

vec3 getNormal(vec3 p){
    float d = 0.0001;
    return normalize(vec3(
        map(p + vec3(  d, 0.0, 0.0)) - map(p + vec3( -d, 0.0, 0.0)),
        map(p + vec3(0.0,   d, 0.0)) - map(p + vec3(0.0,  -d, 0.0)),
        map(p + vec3(0.0, 0.0,   d)) - map(p + vec3(0.0, 0.0,  -d))
    ));
}

void main( void ) {

  vec2 uv = position * 2.0 - 1.0;
  vec3 dir = normalize(vec3(uv, 1.0));
  vec3 pos = vec3(0.0, 0.0, -1.0);
  
  float t = 0.0;
  int iteCount = 0;

  for(int i=0; i < ITE_MAX; i++) {
    float ttemp = map(t * dir + pos);
    if(ttemp < DIST_MIN) {
      iteCount = i;
      break;
    }

    t += ttemp * 0.5;
  }
  
  vec3 ip = pos + dir * t;

  if (iteCount == 0) {
    gl_FragColor = vec4(0.0, 0.0, 0.0, 1.0);
  } else {
    vec3 color = getNormal(ip);
    gl_FragColor = vec4( abs(color), 1.0);
  }
}

```glsl


### 基本光照

![img](https://github.com/chenxiao07/pixel-shader-collection/blob/master/shader/basic_lighting.png)

glsl代码

```

#define ITE_MAX      45
#define DIST_MIN     0.01

float map(vec3 p) {
  return length(p) - 0.5;
}

vec3 getNormal(vec3 p){
    float d = 0.0001;
    return normalize(vec3(
        map(p + vec3(  d, 0.0, 0.0)) - map(p + vec3( -d, 0.0, 0.0)),
        map(p + vec3(0.0,   d, 0.0)) - map(p + vec3(0.0,  -d, 0.0)),
        map(p + vec3(0.0, 0.0,   d)) - map(p + vec3(0.0, 0.0,  -d))
    ));
}

float getLightIndensity (vec3 p) {
  vec3 normal = getNormal(p);

  // 点光源位置在 (1.0, 1.0, -1.0)
  vec3 lightDir = normalize(vec3(1.0, 1.0, -1.0) - p);

  return dot(lightDir, normal);
}

void main( void ) {

  vec2 uv = position * 2.0 - 1.0;
  vec3 dir = normalize(vec3(uv, 1.0));
  vec3 pos = vec3(0.0, 0.0, -1.0);
  
  float t = 0.0;
  int iteCount = 0;

  for(int i=0; i < ITE_MAX; i++) {
    float ttemp = map(t * dir + pos);
    if(ttemp < DIST_MIN) {
      iteCount = i;
      break;
    }

    t += ttemp * 0.5;
  }
  
  vec3 ip = pos + dir * t;

  if (iteCount == 0) {
    gl_FragColor = vec4(0.0, 0.0, 0.0, 1.0);
  } else {
    float color = getLightIndensity(ip);
    gl_FragColor = vec4( color, color, color, 1.0);
  }
}

```



### 平面和球

![img](https://github.com/chenxiao07/pixel-shader-collection/blob/master/shader/sphere_plane.png)

glsl代码

```glsl

#define ITE_MAX      80
#define DIST_MIN     0.01

float map(vec3 p) {
  float plane = p.y;
  float sphere = length(p - vec3(0.0, 0.5, 0.0)) - 0.5;
  return min(plane, sphere);
}

vec3 getNormal(vec3 p){
    float d = 0.0001;
    return normalize(vec3(
        map(p + vec3(  d, 0.0, 0.0)) - map(p + vec3( -d, 0.0, 0.0)),
        map(p + vec3(0.0,   d, 0.0)) - map(p + vec3(0.0,  -d, 0.0)),
        map(p + vec3(0.0, 0.0,   d)) - map(p + vec3(0.0, 0.0,  -d))
    ));
}

float getLightIndensity (vec3 p) {
  vec3 normal = getNormal(p);

  // 点光源位置在 (1.0, 1.0, -1.0)
  vec3 lightDir = normalize(vec3(1.0, 1.0, -1.0) - p);

  return dot(lightDir, normal);
}

void main( void ) {

  vec2 uv = position * 2.0 - 1.0;
  vec3 dir = normalize(vec3(uv, 1.0));
  vec3 pos = vec3(0.0, 0.5, -1.0);
  
  float t = 0.0;
  int iteCount = 0;

  for(int i=0; i < ITE_MAX; i++) {
    float ttemp = map(t * dir + pos);
    if(ttemp < DIST_MIN) {
      iteCount = i;
      break;
    }

    t += ttemp;
  }
  
  vec3 ip = pos + dir * t;

  if (iteCount == 0) {
    gl_FragColor = vec4(0.0, 0.0, 0.0, 1.0);
  } else {
    float color = getLightIndensity(ip);
    gl_FragColor = vec4( color, color, color, 1.0);
  }
}

```




### FOG效果

![img](https://github.com/chenxiao07/pixel-shader-collection/blob/master/shader/fog.png)

glsl代码

```glsl

precision mediump float;

varying vec2 position;

#define ITE_MAX      80
#define DIST_MIN     0.01

float map(vec3 p) {
  float plane = p.y;
  float sphere = length(p - vec3(0.0, 0.5, 0.0)) - 0.5;
  return min(plane, sphere);
}

vec3 getNormal(vec3 p){
    float d = 0.0001;
    return normalize(vec3(
        map(p + vec3(  d, 0.0, 0.0)) - map(p + vec3( -d, 0.0, 0.0)),
        map(p + vec3(0.0,   d, 0.0)) - map(p + vec3(0.0,  -d, 0.0)),
        map(p + vec3(0.0, 0.0,   d)) - map(p + vec3(0.0, 0.0,  -d))
    ));
}

float getLightIndensity (vec3 p) {
  vec3 normal = getNormal(p);

  vec3 lightDir = normalize(vec3(1.0, 1.0, -1.0) - p);

  return dot(lightDir, normal);
}

void main( void ) {

  vec2 uv = position * 2.0 - 1.0;
  vec3 dir = normalize(vec3(uv, 1.0));
  vec3 pos = vec3(0.0, 0.5, -1.0);
  
  float t = 0.0;
  int iteCount = 0;

  for(int i=0; i < ITE_MAX; i++) {
    float ttemp = map(t * dir + pos);
    if(ttemp < DIST_MIN) {
      iteCount = i;
      break;
    }

    t += ttemp;
  }
  
  vec3 ip = pos + dir * t;

  float distance = length(ip);
  float step = smoothstep(1.0, 10.0, distance);

  float color = getLightIndensity(ip);

  gl_FragColor = vec4(mix(vec3(color, color, color), vec3(0.0, 0.0, 0.6), step), 1.0);
}

```
