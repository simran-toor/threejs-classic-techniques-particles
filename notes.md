# 18 PARTICLES 
* can be used to create **stars**, **smoke**, **rain**, **dust**, **fire**, etc. 
* can have thousands with a reasonable frame rate
* each particle is composed of a plane (2 triangles) always facing the camera
* creating particles is like creating a `Mesh`:
    - a **geometry** (`BufferGeometry`)
    - a **material** (`PointsMaterial`)
    - a `Points` instance (instead of `Mesh`)
* remove test cube
## FIRST PARTICLES
### GEOMETRY
* instantiate any `SphereGeometry`
* each vertex of the geometry will become and particle
    ```
    const particleGeometry = new THREE.SphereGeometry(1, 32, 32)
    ```
### POINTS MATERIAL
* instantiate a `PointsMaterial`
* change the `size` property to control all particle size and the `sizeAttenuation` to specify if distant particles should be smaller than the close particles
    ```
    const particlesMaterial = new THREE.PointsMaterial({
        size: 0.02,
        sizeAttenuation: true
    })
    ```
### POINTS
* instantiate and `Points` class and add to scene
    ```
    const particles = new THREE.Points(particlesGeometry, particlesMaterial)
    scene.add(particles)
    ```
## CUSTOM GEOMETRY
* create a `BufferGeometry` and add a `position` attribute
    ```
    const particlesGeometry = new THREE.BufferGeometry()
    const count = 500

    const positions = new Float32Array(count * 3)

    for(let i = 0; i < count; i++){
        positions[i] = (Math.random() - 0.5) * 10
    }

    particlesGeometry.setAttribute(
        'position',
        new THREE.BufferAttribute(positions, 3) 
    )
    ```
    * `positions[i] = (Math.random() - 0.5)` the **- 0.5** centers the particles 
    * `new THREE.BufferAttributes(positions, 3)` groups into 3s 
* test the limits of computer with more particles like **500**, **5000**, **50000**
* stay at **5000** and change size to **0.1**
## COLOR, MAP AND ALPHA MAP
* can change the colour of all the particles with the `color` property on the `PointsMaterial`:
    ```
    const particlesMaterial = new THREE.PointsMaterial({
        color: '#ff88cc',
        size: 0.1,
        sizeAttenuation: true
    })
    ```
* can use the `map` property to put a texture on the particles
* load one of the textures located in folder:
    ```
    const textureLoader = new THREE.TextureLoader()
    const particleTextures = new textureLoader.load('/textures/particles/2.png')

    // ...

    const particlesMaterial = new THREE.PointsMaterial({
        color: '#ff88cc',
        size: 0.1,
        sizeAttenuation: true,
        map: particleTexture
    })
    ```
    * textures provided are from `https://kenney.nl/assets/particle-pack`
    * can also create your own 
* looking closely you can see that the front particles are hiding the back particles 
* activate the transparency and se the texture on the `alphaMap` property instead of the `map`:
    ```
    const particlesMaterial = new THREE.PointsMaterial({
        color: '#ff88cc',
        size: 0.1,
        sizeAttenuation: true,
        transparent: true,
        alphaMap: particleTexture
    })
    ```
* can still see edges of the particles
* this is b/c the particles are drawn in the same order as they are created, and WebGL doesn't know which one is in front of the other
* multiple ways of fixing this
### USING ALPHA TEST
* `alphaTest` is a value between **0** and **1**
* it enables WebGl to know when not to render the pixel according to that pixels transparency 
* deult value is **0** meaning that the pixel will be rendered anyway 
* use **0.001**:
    ```
    const particlesMaterial = new THREE.PointsMaterial({
        color: '#ff88cc',
        size: 0.1,
        sizeAttenuation: true,
        transparent: true,
        alphaMap: particleTexture,
        alphaTest: 0.001
    })
    ```
* when drawing, WebGL tests if what's being drawn is closer than what's already drawn
* this is called **depth testing** and can be deactivated with `alphaTest`:
    ```
    const particlesMaterial = new THREE.PointsMaterial({
        color: '#ff88cc',
        size: 0.1,
        sizeAttenuation: true,
        transparent: true,
        alphaMap: particleTexture,
        //alphaTest: 0.001,
        depthTest: false
    })
    ```
* deactivating depth testing might create bugs if you have other objects in your scene or particles with different colours
* add cube to the scene to see this:
    ```
    // Cube
    const cube = new THREE.Mesh(
        new THREE.BoxGeometry(),
        new THREE.MeshBasicMaterial()
    )
    scene.add(cube)
    ```
* particles that are supposed to be behind the cube can be seen 
* makes a cool effect 
### USING DEPTH WRITE 
* the depth of what's being drawn is stored in a depth buffer
* instead of not testing if the particle is closer than what's in this depth buffer, we can tell WebGL not to write particles in that depth buffer with `depthTest`:
    ```
    const particlesMaterial = new THREE.PointsMaterial({
        color: '#ff88cc',
        size: 0.1,
        sizeAttenuation: true,
        transparent: true,
        alphaMap: particleTexture,
        // alphaTest: 0.001,
        // depthTest: false,
        depthWrite: false
    })
    ```
* many techniques, usually have to adapt and find best combination according to the project
* none of them have much impact on performance 
## BLENDING
* WebGL currently draws pixels one on top of the other
* with the `blending` property, you can tell the WebGL to add the colour of the pixel to the colour of the pixel already drawn 
* change the `blending` property to `THREE.AdditiveBlending:
    ```
    const particlesMaterial = new THREE.PointsMaterial({
        color: '#ff88cc',
        size: 0.1,
        sizeAttenuation: true,
        transparent: true,
        alphaMap: particleTexture,
        // alphaTest: 0.001,
        // depthTest: false,
        depthWrite: false,
        blending: THREE.AdditiveBlending
    })
    ```
    * the effect will impact the performance 
* change the count to `20000` and remove the `cube`
## DIFFERENT COLOURS
* can have a different colour for each particle
* add a `color` attribute with 3 values (red, green, blue):
    ```
    const positions = new Float32Array(count * 3)
    const colors = new Float32Array(count * 3)

    // add to for loop

    for(let i = 0; i < count; i++){
        positions[i] = (Math.random() - 0.5) * 10
        colors[i] = Math.random()
    }

    // add to particleMaterials
    vertexColors: true
    ```
* the main colour of the material still affects these vertex colours, comment it:
    ```
    // color: '#ff88cc',
    ```
## ANIMATE
* There are multiple ways of animating particles
### BY USING THE POINT AS AN OBJECT 
* `Points` class inherits from the `Object3D` so we can move, rotate and scale the points
* rotate the particles in the `tick` function:
    ```
    particles.rotation.y = elapsedTime * 0.2
    ```
### BY CHANGING THE ATTRIBUTES 
* can update each vertex seperatly in `particlesGeometry.attributes.position.arrray`
* b/c this array contains the particles positions
* needs to be **3 by 3** 
* make particles move up and down like waves 
* to create this use sinus:
    ```
    // in tick function
    for(let i = 0; i < count; i++){
        const i3 = i * 3
        particlesGeometry.attributes.position.array[i3 + 1] = Math.sin(elapsedTime)
    }

    particlesGeometry.attributes.position.needsUpdate = true
    ```
* apply an offset to the **sinus** to get wave shape using x coordinate:
    ```
    particlesGeometry.attributes.position.array[i3 + 1] = Math.sin(elapsedTime + x)
    ```
* avoid using this technique because its updating the whole attribute on each frame which is v bad for perfomance 
* ok do do if using small number of particles 0 - 100
### USING A CUSTOM SHADER
* best way to animate particles is to create own shaders 