<h1 align="center">3D ГРАФИКА</h1>

---
<p align="center">Программирование GPU, основы Vulkan API и подход к трёхмерной графике как к объектно-ориентированной системе.</p>
## GPU и OpenGL
#### GPU как вычислительная система
• Видеокарта решает задачу **рендеринга**, т.е. двумерного представления трёхмерной сцены.
• Эта задача сложна и специфична. Графические процессоры всегда отличались от CPU и с ними традиционно работают через разные API.
```mermaid
flowchart TD
	Host["Host<br>program"]
	GAPI["Graphics API<br>(OpenGL, Vulkan)"]
	WAPI["Window API<br>(GLFW, SDL)"]
	Driver["Driver"]
	OS["Operating system API"]
	GPU["GPU"]

	Host --> GAPI
	Host --> WAPI
	GAPI --> Driver
	WAPI --> OS
	Driver <--> GPU
	Driver <--> OS

	classDef yellow fill:#fdfd96,stroke:#333,stroke-width:1px
	classDef green fill:#c9f2c9,stroke:#333,stroke-width:1px
	classDef blue fill:#cbf1f7,stroke:#333,stroke-width:1px

	class Host,GAPI,WAPI yellow
	class Driver,OS green
	class GPU blue
```
#### Давайте отрендерим квадрат
• OpenGL для рендеринга.
```cpp
glClear(GL_COLOR_BUFFER_BIT);
glBegin(GL_QUADS);
glColor3f(1.0, 1.0, 1.0);
for(auto Coord : Vertices)
	glVertex3fv(Coord);
glEnd();
```
• GLFW для окон и управления.
```cpp
glfwSwapBuffers(Wnd.get());
glfwPollEvents();
```
![[../../../_Meta/attachments/10.1.png]]
Пример с гита:
```cpp
//-------------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//-------------------------------------------------------------------------------
//
// Simplest example: blank square on the screen
// This example also show how to use GLFW library which is rather standard
// Note really short time to first triangle (quad in this case)
//
// cl /EHsc ogl-simplest.cc /link glfw3dll.lib opengl32.lib
//
//-------------------------------------------------------------------------------

#include <cassert>
#include <iostream>
#include <memory>
#include <stdexcept>

#include <GLFW/glfw3.h>

// initial window sizes
constexpr int SZX = 600;
constexpr int SZY = 600;

// custom error handler class
struct glfw_error : public std::runtime_error {
	glfw_error(const char* s) : std::runtime_error(s) {}
};

// throw on errors
void error_callback(int, const char* err_str) { throw glfw_error(err_str); }

// make sure the viewport matches the new window dimensions
void framebuffer_size_callback(GLFWwindow* window, int width, int height) {
	glViewport(0, 0, width, height);
}

// initialization routine
GLFWwindow* initialize_window() {
	GLFWwindow* Window;
	glfwSetErrorCallback(error_callback);
	glfwInit();
	Window = glfwCreateWindow(SZX, SZY, "Hello World", NULL, NULL);
	assert(Window); // error callback shall throw otherwise
	glfwMakeContextCurrent(Window);
	glfwSetFrameBufferSizeCallback(Window, framebuffer_size_callback);
	return Window;
}

// vertices to render
GLfloat Vertices[4][3] = {
	{-0.5f, 0.5f, 0.0f}, // top left
	{0.5f, 0.5f, 0.0f}, // top right
	{0.5f, -0.5f, 0.0f}, // bottom right
	{-0.5f, -0.5f, 0.0f}, // bottom left
};

// render routine
void do_render() {
	glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
	glClear(GL_COLOR_BUFFER_BIT);
	glBegin(GL_QUADS);
	glColor3f(1.0, 1.0, 1.0);
	for(auto Coord : Vertices)
		glVertex3fv(Coord);
	glEnd();
}

// entry point
int main() try {
	auto Cleanup = [](GLFWwindow*) { glfwTerminate(); };
	using UWnd = std::unique_ptr<GLFWwindow, decltype(Cleanup)>;
	UWnd Wnd(initialize_window(), Cleanup);
	
	while(!glfwWindowShouldClose(Wnd.get())) {
		do_render();
		glfwSwapBuffers(Wnd.get());
		glfwPollEvents();
	}
} catch(glfw_error& E) {
	std::cout << "GLFW error: " << E.what() << std::endl;
} catch(std::exception& E) {
	std::cout << "Standard error: " << E.what() << std::endl;
} catch(...) {
	std::cout << "Unknown error\n";
}
```
#### Фиксированный конвейер
• Фиксированные блоки.
• Управляют отдельными функциями
```cpp
glEnable(GL_DEPTH_TEST);
glDepthFunc(GL_LESS);
glEnable(GL_CULL_FACE);
glClear(GL_COLOR_BUFFER_BIT);
```
• Это тонна API функций и enums.
```mermaid
flowchart TD
	API["API"]
	PP["Primitive<br>processing"]
	TLCC["Transform<br>Lighting<br>Clipping<br>Culling"]
	PA["Primitive<br>assembly"]
	TP["Texture<br>processing"]
	CSAF["Color sum<br>Alpha channel<br>Fog"]
	DS["Depth<br>Stencil"]
	DB["Dithering<br>Blending"]
	FB["Frame buffer"]

	API -->|geometry| PP
	PP -->|vertices| TLCC
	TLCC --> PA
	PA -->|triangles, lines| PP
	PA -->|rasterization, interpolation| TP
	API -->|textures| TP
	TP --> CSAF
	CSAF --> DS
	DS --> DB
	DB --> FB
	FB -->|pixel read| API

	classDef pink fill:#ffb3ba,stroke:#333,stroke-width:1px
	classDef yellow fill:#fdfd96,stroke:#333,stroke-width:1px
	classDef cyan fill:#bfefff,stroke:#333,stroke-width:1px
	classDef green fill:#c9f2c9,stroke:#333,stroke-width:1px

	class API pink
	class PP,TLCC,PA yellow
	class CSAF,TP,DS,DB cyan
	class FB green
```
#### Общение с рантаймом
• Каждый раз, когда вы дёргаете API функцию, вы дёргаете рантайм, который должен в какой-то момент послать информацию драйверу.
```cpp
for(auto Coord : Vertices)
	glVertex3fv(Coord); // это вызов OpenGL runtime
```
• Проблема в том, что каждый такой API вызов предполагает накладные расходы, на которые вы идёте каждый фрейм. И которые сложно кешировать.
• Нам наоборот хочется максимум отдать в память GPU и минимально с ней взаимодействовать.
• Во многом это компенсируется тем, что в OpenGL возможны **расширения**.
#### Расширения OpenGL: буферы вершин
• Отрендерим тот же квадрат по другому: подготовим буферы вершин.
```cpp
glGenVertexArrays(1, &VAO);
glGenBuffers(1, &VBO);
glBindVertexArray(VAO);
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(Vertices)), Vertices, GL_STATIC_DRAW);
```
• В цикле рендеринга теперь всё стало куда приятней.
```cpp
glClear(GL_COLOR_BUFFER_BIT);
glBindVertexArray(VAO);
glDrawArrays(GL_QUADS, 0, 4);
```
Пример с гита:
```cpp
//-------------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//-------------------------------------------------------------------------------
//
// Extensions simplest example: blank square on the screen
// This example extends ogl_simplest
// glad required for using extensions like GL_ARRAY_BUFFER of GL_STATIC_DRAW
//
// cl /EHsc ogl-extensions.cc /link glad.lib glfw3dll.lib opengl32.lib
//
//-------------------------------------------------------------------------------

#include <cassert>
#include <iostream>
#include <memory>
#include <stdexcept>

// clang-format off
// this headers shall be in this position
#include <glad/glad.h>
// clang format on

#include <GLFW/glfw3.h>

// initial window sizes
constexpr int SZX = 600;
constexpr int SZY = 600;

// custom error handler class
struct glfw_error : public std::runtime_error {
	glfw_error(const char* s) : std::runtime_error(s) {}
};

// throw on errors
void error_callback(int, const char* err_str) { throw glfw_error(err_str); }

// make sure the viewport matches the new window dimensions
void framebuffer_size_callback(GLFWwindow* window, int width, int height) {
	glViewport(0, 0, width, height);
}

// initialization routine
GLFWwindow* initialize_window() {
	GLFWwindow* Window;
	glfwSetErrorCallback(error_callback);
	glfwInit();
	Window = glfwCreateWindow(SZX, SZY, "Hello World", NULL, NULL);
	assert(Window); // error callback shall throw otherwise
	glfwMakeContextCurrent(Window);
	glfwSetFrameBufferSizeCallback(Window, framebuffer_size_callback);
	if(!gladLoadGLLoader(reinterpret_cast<GLADLoadproc>(glfwGetProcAddress)))
		throw glfw_error("Failed to initialize GLAD");
	return Window;
}

// vertices to render
GLfloat Vertices[] = {
	// positions        // colors (not used in this example)
	-0.5f, 0.5f,  0.0f, 0.0f, 0.0f, 0.0f, // top left
	0.5f,  0.5f,  0.0f, 0.0f, 0.0f, 0.0f, // top right
	0.5f,  -0.5f, 0.0f, 0.0f, 0.0f, 0.0f, // bottom right
	-0.5f, -0.5f, 0.0f, 0.0f, 0.0f, 0.0f, // bottom left
};

// render routine
void do_render() {
	glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
	glClear(GL_COLOR_BUFFER_BIT);
	glBindVertexArray(VAO);
	glDrawArrays(GL_QUADS, 0, 4);
}

// entry point
int main() try {
	auto Cleanup = [](GLFWwindow*) { glfwTerminate(); };
	using UWnd = std::unique_ptr<GLFWwindow, decltype(Cleanup)>;
	UWnd Wnd(initialize_window(), Cleanup);
	
	GLuint VBO, VAO;
	glGenVertexArrays(1, &VAO);
	glGenBuffers(1, &VBO);
	
	glBindVertexArray(VAO);
	
	glBindBuffer(GL_ARRAY_BUFFER, VBO);
	glBufferData(GL_ARRAY_BUFFER, sizeof(Vertices), Vertices, GL_STATIC_DRAW);
	
	// position attribute
	glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(GLfloat), nullptr);
	glEnableVertexAttribArray(0);
	// color attribute
	// ...
} catch(glfw_error& E) {
	std::cout << "GLFW error: " << E.what() << std::endl;
} catch(std::exception& E) {
	std::cout << "Standard error: " << E.what() << std::endl;
} catch(...) {
	std::cout << "Unknown error\n";
}
```
#### Расширения и версии
• Буфера вершин были введены расширением ARB_vertex_array_object в OpenGL 2.1 и закреплены в стандарте OpenGL 3.0.
• Расширения предлагаются участниками консорциума и их реально десятки.
https://www.khronos.org/registry/OpenGL/extensions
• Существуют автоматизированные системы такие как glad, запрашивающие вам расширения и генерирующие хедер с доступными функциями.
• Для более тонкого контроля есть библиотека GLEW (The OpenGL Extension Wrangler Library), позволяющая проверять доступность расширений и многое другое.
#### Нефиксированный конвейер
• Первой идеей, появившейся достаточно рано была идея шейдера, т.е. небольшой программы для видеокарты, которая позволяет гибко управлять светом и тенью на каждой вершине.
• Так в 2001 году в OpenGL 2.0 появился язык GLSL.
• В программировании GPU есть своя специфика.
```mermaid
flowchart TD
	API["API"]
	PP["Primitive<br>processing"]
	VS{{"Vertex<br>shader"}}
	CC["Clipping<br>Culling"]
	PA["Primitive<br>assembly"]
	FS{{"Fragment<br>shader"}}
	DS["Depth<br>Stencil"]
	DB["Dithering<br>Blending"]
	FB["Frame buffer"]

	API -->|geometry| PP
	PP -->|vertices| VS
	VS --> CC
	CC --> PA
	PP -->|triangles, lines| PA
	PA -->|rasterization, interpolation| FS
	API -->|textures| FS
	FS --> DS
	DS --> DB
	DB --> FB
	FB -->|pixel read| API

	classDef yellow fill:#fdfd96,stroke:#333,stroke-width:1px
	classDef pink fill:#ffb3ba,stroke:#333,stroke-width:1px
	classDef cyan fill:#bfefff,stroke:#333,stroke-width:1px
	classDef green fill:#c9f2c9,stroke:#333,stroke-width:1px

	class PP,CC,PA yellow
	class VS yellow
	class API pink
	class FS,DS,DB cyan
	class FB green
```
