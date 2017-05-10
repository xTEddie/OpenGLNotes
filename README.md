# OpenGL

- In the old days, Immediate mode was used for OpenGL, it was easy to use but it was inefficient
- OpenGL now forces us to use modern practices: OpenGL's core profile

###### State Machine
- OpenGL is by itself a large state machine
- A collection of variables that define OpenGL should currently operate
- The state of OpenGL is commonly referred as context
- We often change its state by setting some option, manipulating some buffers and then render using the context
- If we want to draw lines instead of triangles, we change the state of OpenGL by changing some context variable that sets how OpenGL should draw

###### Objects
- OpenGL libraries are written in C
- An object is a collection of options that represents a subset of OpenGL's state
- It is advised to use primitive types defined by OpenGL

##### GLFW
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

##### GLEW

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

##### Viewport

We have to tell OpenGL the size of the rendering window so that it knows how we want to display the data and coordinates with respect to the window

```
  int width, height;
  glfwGetFramebufferSize(window, &width, &height);

  glViewport(0, 0, width, height);
```

- **glfwGetFramebufferSize** retrieves the size of the frame buffer of the window and store it in width and height
- **glViewport** sets the location of the lower left corner of the window
- The 3rd and 4th parameters set the width and height of the rendering window in pixels which is retrieved from the GLFW itself.


##### Ready the engines

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

###### One last thing

As soon as we exit the game, we should clean/delete all the resources that were allocated.

```
  glfwTerminate();
  return 0;
```

- ** glfwTerminate** would clean all the allocated resources and it's called at the end of the main function

##### Input

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

- The function checks if the key pressed is the escape key and if it was pressed

The last thing to do is to register the callback function

```
  glfwSetKeyCallback(window, key_callback);  
```

##### Rendering
