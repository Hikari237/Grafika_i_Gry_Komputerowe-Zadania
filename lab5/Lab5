#include <GL/glew.h>
#include <GLFW/glfw3.h>
#include <glm/glm.hpp>
#include <glm/gtc/matrix_transform.hpp>
#include <glm/gtc/type_ptr.hpp>
#include <iostream>
#include <vector>
#include <cmath>
#include <stack>

#pragma comment(lib, "opengl32.lib")

const float PI = 3.14159265359f;

int objectNumber = 1;
int colorIndex = 0;
const int N = 13;

float rotX = 15.0f, rotY = 0.0f, rotZ = 0.0f;

using MatStack = std::stack<glm::mat4>;
void pushM(MatStack& s) { s.push(s.top()); }
void popM(MatStack& s) { s.pop(); }

const char* vertexSrc = R"(
    #version 330 core
    layout(location = 0) in vec3 aPos;
    layout(location = 1) in float aPS;
    
    uniform mat4 MVP;
    
    void main() {
        gl_Position = MVP * vec4(aPos, 1.0);
        gl_PointSize = aPS;
    }
)";

const char* fragmentSrc = R"(
    #version 330 core
    uniform vec3 uColor;
    uniform int isPoint; // NASZ PSTRYCZEK
    out vec4 fragColor;
    
    void main() {
        // Wycinaj kółka TYLKO dla korkociągu
        if (isPoint == 1) {
            vec2 c = gl_PointCoord - vec2(0.5);
            if (dot(c, c) > 0.25) discard; 
        }
        
        fragColor = vec4(uColor, 1.0);
    }
)";

GLuint compileShader(GLenum type, const char* src) {
    GLuint shader = glCreateShader(type);
    glShaderSource(shader, 1, &src, nullptr);
    glCompileShader(shader);
    int ok;
    glGetShaderiv(shader, GL_COMPILE_STATUS, &ok);
    if (!ok) {
        char log[512];
        glGetShaderInfoLog(shader, 512, nullptr, log);
        std::cerr << log << "\n";
    }
    return shader;
}

GLuint createProgram() {
    GLuint vs = compileShader(GL_VERTEX_SHADER, vertexSrc);
    GLuint fs = compileShader(GL_FRAGMENT_SHADER, fragmentSrc);
    GLuint prog = glCreateProgram();
    glAttachShader(prog, vs);
    glAttachShader(prog, fs);
    glLinkProgram(prog);
    glDeleteShader(vs);
    glDeleteShader(fs);
    return prog;
}

void keyCallback(GLFWwindow* window, int key, int scancode, int action, int mods) {
    if (action == GLFW_PRESS || action == GLFW_REPEAT) {
        const float step = 5.0f;
        if (key == GLFW_KEY_ESCAPE) glfwSetWindowShouldClose(window, true);
        if (key == GLFW_KEY_1) objectNumber = 1;
        if (key == GLFW_KEY_2) objectNumber = 2;
        if (key == GLFW_KEY_C && action == GLFW_PRESS) colorIndex = (colorIndex + 1) % 3;
        if (key == GLFW_KEY_UP) rotX -= step;
        if (key == GLFW_KEY_DOWN) rotX += step;
        if (key == GLFW_KEY_LEFT) rotY -= step;
        if (key == GLFW_KEY_RIGHT) rotY += step;
        if (key == GLFW_KEY_PAGE_UP) rotZ += step;
        if (key == GLFW_KEY_PAGE_DOWN) rotZ -= step;
        if (key == GLFW_KEY_HOME) { rotX = 15.0f; rotY = 0.0f; rotZ = 0.0f; }
    }
}

int main() {
    glfwInit();
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

    GLFWwindow* window = glfwCreateWindow(800, 600, "Grafika 3D", nullptr, nullptr);
    glfwMakeContextCurrent(window);
    glfwSetKeyCallback(window, keyCallback);

    glewExperimental = GL_TRUE;
    glewInit();

    glEnable(GL_DEPTH_TEST);
    glEnable(GL_PROGRAM_POINT_SIZE);

    GLuint program = createProgram();
    GLint uMVP = glGetUniformLocation(program, "MVP");
    GLint uColor = glGetUniformLocation(program, "uColor");
    GLint uIsPoint = glGetUniformLocation(program, "isPoint"); 

    std::vector<float> corkscrewData;
    int turns = 13;
    int pointsPerTurn = 60;
    int corkscrewCount = turns * pointsPerTurn;
    float r = 0.6f;

    for (int i = 0; i < corkscrewCount; i++) {
        float angle = i * 0.15f;
        float y = ((float)i / corkscrewCount) * 2.0f - 1.0f;
        float x = cos(angle) * r;
        float z = sin(angle) * r;
        float pointSize = 2.0f + ((float)i / corkscrewCount) * 15.0f;

        corkscrewData.push_back(x);
        corkscrewData.push_back(y);
        corkscrewData.push_back(z);
        corkscrewData.push_back(pointSize);
    }

    GLuint corkscrewVAO, corkscrewVBO;
    glGenVertexArrays(1, &corkscrewVAO);
    glGenBuffers(1, &corkscrewVBO);
    glBindVertexArray(corkscrewVAO);
    glBindBuffer(GL_ARRAY_BUFFER, corkscrewVBO);
    glBufferData(GL_ARRAY_BUFFER, corkscrewData.size() * sizeof(float), corkscrewData.data(), GL_STATIC_DRAW);
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 4 * sizeof(float), (void*)0);
    glEnableVertexAttribArray(0);
    glVertexAttribPointer(1, 1, GL_FLOAT, GL_FALSE, 4 * sizeof(float), (void*)(3 * sizeof(float)));
    glEnableVertexAttribArray(1);

    std::vector<float> baseData;
    baseData.push_back(0.0f); baseData.push_back(-0.5f); baseData.push_back(0.0f);
    for (int i = 0; i <= N; i++) {
        float angle = i * 2.0f * PI / N;
        baseData.push_back(cos(angle) * 0.8f);
        baseData.push_back(-0.5f);
        baseData.push_back(sin(angle) * 0.8f);
    }
    int pyramidBaseCount = baseData.size() / 3;

    GLuint baseVAO, baseVBO;
    glGenVertexArrays(1, &baseVAO);
    glGenBuffers(1, &baseVBO);
    glBindVertexArray(baseVAO);
    glBindBuffer(GL_ARRAY_BUFFER, baseVBO);
    glBufferData(GL_ARRAY_BUFFER, baseData.size() * sizeof(float), baseData.data(), GL_STATIC_DRAW);
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
    glEnableVertexAttribArray(0);

    float angle1 = 0.0f;
    float angle2 = 2.0f * PI / N;
    float sideData[] = {
        0.0f, 0.8f, 0.0f,
        cos(angle1) * 0.8f, -0.5f, sin(angle1) * 0.8f,
        cos(angle2) * 0.8f, -0.5f, sin(angle2) * 0.8f
    };

    GLuint sideVAO, sideVBO;
    glGenVertexArrays(1, &sideVAO);
    glGenBuffers(1, &sideVBO);
    glBindVertexArray(sideVAO);
    glBindBuffer(GL_ARRAY_BUFFER, sideVBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(sideData), sideData, GL_STATIC_DRAW);
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
    glEnableVertexAttribArray(0);

    while (!glfwWindowShouldClose(window)) {
        glClearColor(0.1f, 0.1f, 0.12f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

        glUseProgram(program);

        int width, height;
        glfwGetWindowSize(window, &width, &height);
        float aspect = (height == 0) ? 1.0f : (float)width / (float)height;

        glm::mat4 proj = glm::perspective(glm::radians(45.0f), aspect, 0.1f, 50.0f);
        glm::mat4 view = glm::lookAt(glm::vec3(0.0f, 1.0f, 4.5f), glm::vec3(0.0f, 0.0f, 0.0f), glm::vec3(0.0f, 1.0f, 0.0f));

        glm::mat4 model = glm::mat4(1.0f);
        model = glm::rotate(model, glm::radians(rotX), glm::vec3(1.0f, 0.0f, 0.0f));
        model = glm::rotate(model, glm::radians(rotY), glm::vec3(0.0f, 1.0f, 0.0f));
        model = glm::rotate(model, glm::radians(rotZ), glm::vec3(0.0f, 0.0f, 1.0f));

        glm::mat4 mvp = proj * view * model;

        if (objectNumber == 1) {
            glUniform1i(uIsPoint, 1);

            if (colorIndex == 0) glUniform3f(uColor, 0.0f, 1.0f, 0.0f);
            if (colorIndex == 1) glUniform3f(uColor, 0.0f, 0.0f, 1.0f);
            if (colorIndex == 2) glUniform3f(uColor, 0.6f, 0.3f, 0.1f);

            glUniformMatrix4fv(uMVP, 1, GL_FALSE, glm::value_ptr(mvp));
            glBindVertexArray(corkscrewVAO);
            glDrawArrays(GL_POINTS, 0, corkscrewCount);

        }
        else if (objectNumber == 2) {
            glUniform1i(uIsPoint, 0); 

            MatStack ms;
            ms.push(mvp);

            glUniform3f(uColor, 0.5f, 0.5f, 0.5f);
            glUniformMatrix4fv(uMVP, 1, GL_FALSE, glm::value_ptr(ms.top()));
            glBindVertexArray(baseVAO);
            glDrawArrays(GL_TRIANGLE_FAN, 0, pyramidBaseCount);

            for (int i = 0; i < N; i++) {
                pushM(ms);

                ms.top() = glm::rotate(ms.top(), i * 2.0f * PI / N, glm::vec3(0.0f, 1.0f, 0.0f));
                glUniformMatrix4fv(uMVP, 1, GL_FALSE, glm::value_ptr(ms.top()));

                if (i % 2 == 0) glUniform3f(uColor, 0.8f, 0.3f, 0.3f);
                else glUniform3f(uColor, 0.6f, 0.1f, 0.1f);

                glBindVertexArray(sideVAO);
                glDrawArrays(GL_TRIANGLE_FAN, 0, 3);

                popM(ms);
            }
        }

        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    glfwTerminate();
    return 0;
}
