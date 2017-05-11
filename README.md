# OpenGL

- In the old days, Immediate mode was used for OpenGL, it was easy to use but it was inefficient
- OpenGL now forces us to use modern practices: OpenGL's core profile

#### State Machine
- OpenGL is by itself a large state machine
- A collection of variables that define OpenGL should currently operate
- The state of OpenGL is commonly referred as context
- We often change its state by setting some option, manipulating some buffers and then render using the context
- If we want to draw lines instead of triangles, we change the state of OpenGL by changing some context variable that sets how OpenGL should draw

#### Objects
- OpenGL libraries are written in C
- An object is a collection of options that represents a subset of OpenGL's state
- It is advised to use primitive types defined by OpenGL

#### GLFW
- Library written in C
- Provides necessities required for rendering goodies on the screen

```
  int main()
  {
      glfwInit();
      glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
      glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
      glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
      glfwWindowHint(GLFW_RESIZABLE, GL_FALSE);

      return 0;
  }
```

- **glfwInit()** initializes GLFW
- **glwfWindowHint(int option, int value)** configures option
- The code sets the GLFW version, explicitly uses the core-profile and make the window not resizable by the user

- - - -

```
  GLFWwindow* window = glfwCreateWindow(800, 600, "LearnOpenGL", nullptr, nullptr);
  if (window == nullptr)
  {
      std::cout << "Failed to create GLFW window" << std::endl;
      glfwTerminate();
      return -1;
  }
  glfwMakeContextCurrent(window);
```

- **glfwCreateWindow** sets its height, width and title (last two parameters can be ignored)
- We tell GLFW to make the context of our window the main context on the current thread

#### GLEW

GLEW manages function pointers for OpenGL so we want to initialize GLEW before we can any OpenGL functions

```
  glewExperimental = GL_TRUE;
  if (glewInit() != GLEW_OK)
  {
      std::cout << "Failed to initialize GLEW" << std::endl;
      return -1;
  }
```

- **glewExperimental** is set to GL_TRUE to ensure GLEW uses the modern technique for managing OpenGL functionality
- Setting to GL_FALSE would cause some issues
- Do not forget to define pre-processor variable if static linking is used

```
  #define GLEW_STATIC
  #include <GL/glew.h>
```

#### Viewport

We have to tell OpenGL the size of the rendering window so that it knows how we want to display the data and coordinates with respect to the window

```
  int width, height;
  glfwGetFramebufferSize(window, &width, &height);

  glViewport(0, 0, width, height);
```

- **glfwGetFramebufferSize** retrieves the size of the frame buffer of the window and store it in width and height
- **glViewport** sets the location of the lower left corner of the window
- The 3rd and 4th parameters set the width and height of the rendering window in pixels which is retrieved from the GLFW itself.


#### Ready the engines

We don't want the application to draw an image then immediately quit and close the window. So, we need a game loop for the application to keep drawing images and handling user inputs until it is asked to quit.

```
  while(!glfwWindowShouldClose(window))
  {
      glfwPollEvents();
      glfwSwapBuffers(window);
  }
```

- **glfwWindowShouldClose** checks if GLFW has been instructed to close (returns GL_TRUE/GL_FALSE)
- **glfwPollEvents** checks if any events are triggered (keyboard input and mouse movement events) and calls the corresponding functions (set via callbacks)
- **glfwSwapBuffers** will swap the color buffer that has been used to draw in during this iteration and show it as output to the screen
- When an application draw in a single buffer the resulting image might display flickering issues. It is due to the resulting image is not drawn instantly. To fix this issue, double buffer is used for rendering. The front buffer contains the final output image that is shown at the screen while all the rending commands draw to the back buffer. As soon as all the rendering commands are finished, we swap the back buffer to the front buffer so the image is instantly displayed to the user.

#### One last thing

As soon as we exit the game, we should clean/delete all the resources that were allocated.

```
  glfwTerminate();
  return 0;
```

- ** glfwTerminate** would clean all the allocated resources and it's called at the end of the main function

#### Input

We want to have some form of input control in GLFW and we can achieve this using callback functions. A callback function is a function pointer that you can set that GLFW can call at an appropriate time

```
  void key_callback(GLFWwindow* window, int key, int scancode, int action, int mode);
```

- The key input function takes GLFWwindow as its first argument, an integer that specifies the key pressed, an action that specifies if the key is pressed or released and an integer representing some bit flags to tell you if shift, control, alt or super keys have been pressed

```
  void key_callback(GLFWwindow* window, int key, int scancode, int action, int mode)
  {
      // When a user presses the escape key, we set the WindowShouldClose property to true,
      // closing the application
      if(key == GLFW_KEY_ESCAPE && action == GLFW_PRESS)
      	glfwSetWindowShouldClose(window, GL_TRUE);
  }  
```

- The function checks if the key pressed is the escape key and if it was pressed (not release)

The last thing to do is to register the callback function

```
  glfwSetKeyCallback(window, key_callback);  
```

#### Rendering

We want to place all the rendering commands in the game loop, since we want to execute all the rendering commands each iteration of the loop.

```
// Program loop
  while(!glfwWindowShouldClose(window))
  {
      // Check and call events
      glfwPollEvents();

      // Rendering commands here
      ...

      // Swap the buffers
      glfwSwapBuffers(window);
  }
```

Here is the command to change the color of the screen

```
  glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
  glClear(GL_COLOR_BUFFER_BIT);
```

- **glClearColor(int x, int y, int z, int a)** sets the color that OpenGL uses to clear the colorbuffer.
- **glClear** clears the entire buffer of the current framebuffer. When GL_COLOR_BUFFER_BIT is specified, the buffer is cleared to the color specified in **glClearColor**

### Hello Triangle

- In OpenGL, everything is in 3D space, but the screen and window are a 2D array of pixels, so OpenGL transforms all 3D coordinates to 2D to fit in the screen
- The process of transforming 3D coordinates to 2D coordinates is managed by the graphics pipeline in OpenGL
- The graphics pipeline can be divided into two large parts:
  - Transform 3D coordinates to 2D coordinates
  - Transform 2D coordinates into colored pixels
- The graphics pipeline can be divided into several steps where each step requires the output of the previous step as its input
- All the steps can easily be executed in parallel
- The GPU of today have thousands of small processing cores to quickly process the data within the graphics pipeline by running small programs for each step
- Small programs are called shaders
- Some of these shaders can be configured by the developers to replace the default ones
- Shaders are written in the OpenGL Shading Language (GLSL)

Here is an abstract representation of all stages  of the graphics pipeline. The blue sections are configurable by the developers

![Alt](https://learnopengl.com/img/getting-started/pipeline.png)

- As an input to the graphics pipeline, we pass a list of three 3D coordinates that represent the coordinates of a triangle in an array called **Vertex Data**
- The first part of the pipeline is the vertex shader that takes as input a single vertex
- The main purpose of the vertex shader is to transform 3D coordinates into different 3D coordinates and the vertex shader allows us to do some basic processing on the vertex attributes
- The primitive assembly takes all the vertices (or vertex if GL_POINTS is chosen) as an input from the vertex shader and assembles all the point(s) in the primitive shape given
- The output of the primitive assembly stage is passed to the geometry shader
- The geometry shader takes as input a collection of vertices that form a primitive and has the ability to generate other shapes by emitting new vertices to form other primitive(s)
- The output of the geometry shader is then passed on the rasterization stage where it maps the resulting primitive(s) to the corresponding pixels on the final screen, resulting in fragment shader to use.
- Before the fragment shaders runs, clipping is performed. Clipping discards all fragments that are outside your view, increasing performance.
- A fragment in OpenGL is all the data required for OpenGL to render a single pixel
- The main purpose of the fragment shader is to calculate the final color of a pixel and it is usually the stage where all the advanced OpenGL effects occur
- Usually, the fragment shader contains data about the 3D scene that it can use to calculate the final pixel color (lights, shadows, color of the light and so on)
- Once all the corresponding color values have been determined, the final object is passed to the last stage called alpha test and blending stage
- The last stage checks the depth (and stencil) of the fragment and uses those to check if the resulting fragment is in front or behind other objects and should be discarded accordingly
- In modern OpenGL we are required to define at least the vertx and fragment shader on our own. The geometry shader is optional and usually left to its default shader

#### Vertex Input

- All coordinate we specify in OpenGL are in 3D
- OpenGL only processes 3D coordinates when they're in specific range between -1.0 to 1.0 on all 3 axes (x, y and z)
- Because OpenGL works in 3D, we render a 2D triangle with each vertex having a z coordinate of 0.0. This way the depth of the triangle remains the same making it
look like 2D

```
  GLfloat vertices[] = {
      -0.5f, -0.5f, 0.0f,
       0.5f, -0.5f, 0.0f,
       0.0f,  0.5f, 0.0f
  };  
```

- With the vertex data defined, we'd like to send it as input to the vertex shader. This is done by creating memory on the GPU where to store the vertex data configure how OpenGL should interpret the memory and specify how to send data to the graphic card
- The vertex shader then processes as much vertices as we tell it to from its memory
- We manage memory via vertex buffer object (VBO) that can store a large number of vertices in the GPU's memory
- The advantage using VBO is that we can send large batches of data all at once to the graphics card without having to send data a vertex at a time
- Sending data to the graphics card from the CPU is slow, so we try to send as much data possible at once
- Once the data is in the graphics card's memory, the vertex shader has almost instant access to the vertices

```
  GLuint VBO;
  glGenBuffers(1, &VBO);
```
 - **glGenBuffers(GLsizei size, GLuint* buffers)** generates one/mutliple buffer objects and store each object's ID to the buffers' array

OpenGL allows us to bind several buffers at once as long as they have a different buffer type

 ```
 glBindBuffer(GL_ARRAY_BUFFER, VBO);  
 ```

From that point on any buffer call  we make on GL_ARRAY_BUFFER will be used to configure the currently bound buffer, which is VBO

```
  glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
```

- **glBufferData** is a function to copy user-defined data into the currently bound buffer
- Its first argument is the type of the buffer we want to copy data into
- The second argument specifies the size of the data (in bytes) we want to pass to the buffer
- The third parameter is the actual dat we want to send
- The fourth parameter specifies  how we want the graphics card to manage the given data:
  - GL_STATIC_DRAW: the data will most likely not change at all or very rarely.
  - GL_DYNAMIC_DRAW: the data is likely to change a lot.
  - GL_STREAM_DRAW: the data will change every time it is drawn.
- The position data of the triangle does not change and stays the same for every render call so its usage type should best be GL_STATIC_DRAW. - If one would have a buffer with data that is likely to change frequently, a usage type of GL_DYNAMIC_DRAW or GL_STREAM_DRAW ensures the graphics card will place the data in memory that allows for faster writes.

#### Vertex Shader

Modern OpenGL requires us to at least set up the vertex and fragment shader if we want to do some rendering. The shader language GLSL (OpenGL Shading Language) is used to write the vertex shader.

```
  #version 330 core

  layout (location = 0) in vec3 position;

  void main()
  {
    gl_Position = vec4(position.x, position.y, position.z, 1.0);
  }
```

- **vec3** position is an input variable of 3D coordinates and the location of the variable is set to 0
- To set the output of the vertex shader we have to assign the position data to the predefined **gl_Position**
