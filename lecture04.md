
### FAKE AO

![img](https://github.com/chenxiao07/pixel-shader-collection/blob/master/shader/fake_ao.png)

glsl代码

```glsl

precision mediump float;

varying vec2 position;

#define ITE_MAX      50
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

float ambientOcclusion(vec3 p, vec3 n)
{
  float stepSize = 0.01;
  float t = stepSize;
  float oc = 0.0;
  for(int i = 0; i < 10; ++i)
  {
    float d = map(p + n * t);
    oc += t - d; // Actual distance to surface - distance field value
    t += stepSize;
  }

  return clamp(oc, 0.0, 1.0);
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
  
  float ao =  ambientOcclusion(ip, getNormal(ip));

  float color = getLightIndensity(ip) * (1.0 - ao);

  gl_FragColor = vec4(mix(vec3(color, color, color), vec3(0.0, 0.0, 0.6), step), 1.0);
}

```



### 添加 Global Ambient

![img](https://github.com/chenxiao07/pixel-shader-collection/blob/master/shader/global_ambient.png)

glsl代码

```glsl

precision mediump float;

varying vec2 position;

#define ITE_MAX      50
#define DIST_MIN     0.01

const vec3 g_ambient = vec3(0.15, 0.2, 0.32);

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

  float lightIndensity = getLightIndensity(ip);

  vec3 color = vec3(1.0) * lightIndensity + g_ambient * (1.0 - lightIndensity);

  gl_FragColor = vec4(mix(color, vec3(0.0, 0.0, 0.6), step), 1.0);
}

```


### Global Ambient 和 Fake AO 一起

![img](https://github.com/chenxiao07/pixel-shader-collection/blob/master/shader/AO_global_ambient.png)

glsl代码

```glsl

precision mediump float;

varying vec2 position;

#define ITE_MAX      50
#define DIST_MIN     0.01

const vec3 g_ambient = vec3(0.15, 0.2, 0.32);

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

float ambientOcclusion(vec3 p, vec3 n)
{
  float stepSize = 0.01;
  float t = stepSize;
  float oc = 0.0;
  for(int i = 0; i < 10; ++i)
  {
    float d = map(p + n * t);
    oc += t - d; // Actual distance to surface - distance field value
    t += stepSize;
  }

  return clamp(oc, 0.0, 1.0);
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
  
  float ao =  ambientOcclusion(ip, getNormal(ip));

  float lightIndensity = getLightIndensity(ip);

  vec3 color = vec3(1.0) * lightIndensity + g_ambient * (1.0 - lightIndensity);

  color = color * (1.0 - ao);

  gl_FragColor = vec4(mix(color, vec3(0.0, 0.0, 0.6), step), 1.0);
}

```

### Shadow

![img](https://github.com/chenxiao07/pixel-shader-collection/blob/master/shader/shadow.png)

glsl代码

```glsl

precision mediump float;

varying vec2 position;

#define ITE_MAX      50
#define DIST_MIN     0.01

const vec3 g_ambient = vec3(0.15, 0.2, 0.32);

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

float ambientOcclusion(vec3 p, vec3 n)
{
  float stepSize = 0.01;
  float t = stepSize;
  float oc = 0.0;
  for(int i = 0; i < 10; ++i)
  {
    float d = map(p + n * t);
    oc += t - d; // Actual distance to surface - distance field value
    t += stepSize;
  }

  return clamp(oc, 0.0, 1.0);
}

float getShadow(vec3 p0, vec3 p1, float k)
{
  vec3 rd = normalize(p1 - p0);
  float t = 10.0 * DIST_MIN; // Start a bit away from the surface
  float maxt = length(p1 - p0);
  float f = 1.0;
  for(int i = 0; i < ITE_MAX; ++i)
  {
    float d = map(p0 + rd * t);

    // A surface was hit before we reached p1
    if(d < DIST_MIN)
      return 0.0;

    f = min(f, k * d / t);

    t += d;

    // We reached p1
    if(t >= maxt)
      break;
  }

  return f;
}

float getLightIndensity (vec3 p) {
  vec3 normal = getNormal(p);

  vec3 lightPos = vec3(1.0, 1.0, -1.0);

  float lightIntensity = 0.0;

  float shadow = getShadow(p, lightPos, 16.0);
  if(shadow > 0.0) // If we are at all visible
  {
    vec3 lightDirection = normalize(lightPos - p);
    lightIntensity = shadow * clamp(dot(normal, lightDirection), 0.0, 1.0);
  }

  return lightIntensity;
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
  
  float ao =  ambientOcclusion(ip, getNormal(ip));

  float lightIndensity = getLightIndensity(ip);

  vec3 color = vec3(1.0) * lightIndensity + g_ambient * (1.0 - lightIndensity);

  color = color * (1.0 - ao);

  gl_FragColor = vec4(mix(color, vec3(0.0, 0.0, 0.6), step), 1.0);
}

```



