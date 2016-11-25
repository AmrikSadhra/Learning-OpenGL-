# Open GL Notes

## Vertex Buffer Objects (VBO) - GL_ARRAY_BUFFER OpenGL buffer type

Create memory to store vertex data on GPU, config how OpenGL should interpret the memory and specify how to send data to GPU

```c
GLuint VBO;
glGenBuffers(1, &VBO);  //Generate VBO with buffer ID of 1
glBindBuffer(GL_ARRAY_BUFFER, VBO); //Bind VBO to GL_ARRAY_BUFFER
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);  //Copy vertex data into buffer
```

Types of Draw (Controls what memory GPU will place data in)

```
1\. GL_STATIC_DRAW: the data will most likely not change at all or very rarely.
2\. GL_DYNAMIC_DRAW: the data is likely to change a lot.
3\. GL_STREAM_DRAW: the data will change every time it is drawn.
```

**Vertex Data -> Vertex Shader (Normalised device coords -1.0 to 1.0) -> Transformed to screen space via glViewport config**

## GLSL

```c
#version 330 core
//Declare version, specify core profile.

    //Declare inputs
    layout (location = 0) in vec3 position;

    void main()
    {
        //gl_Position is a vec4, is output of shader
            //Need to cast out vec3 to vec4    
        gl_Position = vec4(position.x, position.y, position.z, 1.0);
    }
```

## Vertex Shader

Need a vertex shader to render geometry

```c
GLchar *vertexShaderSource = "source here\n";
GLuint vertexShader;
vertexShader = glCreateShader(GL_VERTEX_SHADER); glShaderSource(vertexShader, 1, &vertexShaderSource, NULL); glCompileShader(vertexShader);

GLint success;
GLchar infoLog[512];
glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);
if(!success) {
    glGetShaderInfoLog(vertexShader, 512, NULL, infoLog); std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
}
```

## Fragment Shader

Needed to add color to geometry. Same process as vertex shader.

```c
#version 330 core

out vec4 color;

void main()
{
    color = vec4(1.0f, 0.5f, 0.2f, 1.0f);
}
```

## Shader Program

We gotta link these shaders if we're gonna use them. Link them to a shader program object, then activate it when rendering objects. Activated shader program will be used when doing render calls.

```c
//Link the shaders to a shader program so they can be used
    GLUint shaderProgram;
    shaderProgram = glCreateProgram();

    glAttachShader(shaderProgram, vertexShader);
    glAttachShader(shaderProgram, fragmentShader);
    glLinkProgram(shaderProgram);

    //Check status of link into shaderProgram
    glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);
    if(!success) {
        glGetProgramInfoLog(shaderProgram, 512, NULL, infoLog);
        std::cout << "Shader program link fail!\n" << infoLog << std::endl;
    }

    //Use the shader program for any rendering calls
    glUseProgram(shaderProgram);
    //Delete the shaders as we no longer need them
    glDeleteShader(vertexShader);
    glDeleteShader(fragmentShader);
```

## Link Vertex Attributes

OpenGL doesn't yet know how to interpret vertex data in memory , or how it should connect the vertex data to the vertex shader's attributes. We must link the shader to the data.

| V1 | V1 | V1 | V2 | V2 | V2 | V3 | V3 | V3 |
|----|----|----|----|----|----|----|----|----|
| X  | Y  | X  | X  | Y  | Z  | X  | Y  | Z  |

- The position data is stored as 32-bit (4 byte) floating point values.
- Each position is composed of 3 of those values.
- There is no space (or other values) between each set of 3 values. The values are tightly packed in the array.
The first value in the data is at the beginning of the buffer.

```C
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(GLfloat), (GLvoid*)0);
glEnableVertexAttribArray(0);  
```

- The first parameter specifies which vertex attribute we want to configure (location = 0).
- The next argument specifies the size of the vertex attribute. The vertex attribute is a vec3 so it is composed of 3 values.
- The third argument specifies the type of the data which is GL_FLOAT.
- The next argument specifies if we want the data to be normalized. If we set this to GL_TRUE all the data that has a value not between 0 (or -1 for signed data) and 1 will be mapped to those values.
- The fifth argument is known as the stride and tells us the space between consecutive vertex attribute sets. Since the next set of position data is located exactly 3 times the size of a GLfloat away we specify that value as the stride. Note that since we know that the array is tightly packed (there is no space between the next vertex attribute value) we could've also specified the stride as 0 to let OpenGL determine the stride (this only works when values are tightly packed).
- The last parameter is of type GLvoid* and thus requires that weird cast. This is the offset of where the position data begins in the buffer. Since the position data is at the start of the data array this value is just 0.

## Drawing an Objects

Must repeat this code every time we draw an object. Binding the appropriate buffer objects and configuring all vertex attributes can be cumbersome. Reference **VAO**

```C
// 0. Copy our vertices array in a buffer for OpenGL to use
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
// 1. Then set the vertex attributes pointers
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(GLfloat), (GLvoid*)0);
glEnableVertexAttribArray(0);  
// 2. Use our shader program when we want to render an object
glUseProgram(shaderProgram);
// 3. Now draw the object
someOpenGLFunctionThatDrawsOurTriangle();   
```

## Vertex Array Object (VAO's)

Core OpenGL requires we use VAO's
