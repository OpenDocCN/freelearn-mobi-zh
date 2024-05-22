# 第六章。太阳系

当我 8 岁的时候，在学校的一个科学项目中，我用电线、聚苯乙烯泡沫球和油漆制作了一个*太阳系*。今天，全世界的 8 岁孩子们都可以在 VR 中制作虚拟太阳系，特别是如果他们读了这一章！这个项目创建了一个模拟我们太阳系的 Cardboard VR 应用程序。也许不是完全科学准确，但对于一个孩子的项目来说已经足够好了，比聚苯乙烯泡沫球要好。

在本章中，您将使用`RenderBox`库创建一个新的太阳系项目，执行以下步骤：

+   设置新项目

+   创建一个`Sphere`组件和一个纯色材料

+   添加带照明的“地球”纹理材料

+   安排太阳系的几何形状

+   天体的动画

+   交互式更改相机位置

+   使用我们的新代码更新`RenderBox`库

当我们将它们放在一起时，我们将从一个球体创建行星和卫星。然而，大部分代码将用于渲染这些天体的各种材料和着色器。

### 注意

此项目的源代码可以在 Packt Publishing 网站上找到，并且在 GitHub 上也可以找到[`github.com/cardbookvr/solarsystem`](https://github.com/cardbookvr/solarsystem)（每个主题作为单独的提交）。

# 设置新项目

为了构建此项目，我们将使用我们在第五章中创建的`RenderBox`库，*RenderBox 引擎*。您可以使用您自己的库，或者从本书提供的可下载文件或我们的 GitHub 存储库中获取副本（使用标记为`after-ch5`的提交——[`github.com/cardbookvr/renderboxlib/releases/tag/after-ch5`](https://github.com/cardbookvr/renderboxlib/releases/tag/after-ch5)）。有关如何导入`RenderBox`库的更详细说明，请参阅第五章的最终*在未来项目中使用 RenderBox*部分，*RenderBox 引擎*。执行以下步骤创建新项目：

1.  在打开的 Android Studio 中创建一个新项目。让我们将其命名为`SolarSystem`，并针对**Android 4.4 KitKat (API 19)**使用**空活动**。

1.  为`renderbox`、`common`和`core`包创建新模块，使用**文件** | **新建模块** | **导入.JAR/.AAR 包**。

1.  将模块设置为应用程序的依赖项，使用**文件** | **项目结构**。

1.  按照第二章中的说明编辑`build.gradle`文件，*骨架 Cardboard 项目*，以编译 SDK 22。

1.  更新`/res/layout/activity_main.xml`和`AndroidManifest.xml`，如前几章所述。

1.  将`MainActivity`编辑为`class MainActivity extends CardboardActivity implements IRenderBox`，并实现接口方法存根（*Ctrl* + *I*）。

我们可以继续在`MainActivity`中定义`onCreate`方法。该类现在具有以下代码：

```kt
public class MainActivity extends CardboardActivity implements IRenderBox {
    private static final String TAG = "SolarSystem";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        CardboardView cardboardView = (CardboardView) findViewById(R.id.cardboard_view);
        cardboardView.setRenderer(new RenderBox(this, this));
        setCardboardView(cardboardView);
    }
    @Override
    public void setup() {

    }
    @Override
    public void preDraw() {
    }
    @Override
    public void postDraw() {
    }
}
```

在构建此项目时，我们将创建一些新的类，这些类可能是`RenderBox`库的良好扩展。我们将首先将它们作为常规类放入此项目。然后，在本章末尾，我们将帮助您将它们移入`RenderBox`库项目并重新构建该库：

1.  右键单击`solarsystem`文件夹（`com.cardbookvr.solarsystem`），选择**新建** | **包**，并命名为`RenderBoxExt`。

1.  在`RenderBoxExt`中，创建名为`components`和`materials`的包子文件夹。

没有真正的技术需要将其制作成一个单独的包，但这有助于组织我们的文件，因为`RenderBoxExt`中的文件将在本章末尾移入我们可重用的库中。

您可以将一个立方体临时添加到场景中，以确保一切设置正确。将其添加到`setup`方法中，如下所示：

```kt
    public void setup() {
        new Transform()
            .setLocalPosition(0,0,-7)
            .setLocalRotation(45,60,0)
            .addComponent(new Cube(true));
    }
```

如果您还记得，立方体是添加到变换中的一个组件。立方体定义了它的几何形状（例如，顶点）。变换定义了它在 3D 空间中的位置、旋转和缩放。

您应该能够在 Android 设备上点击**Run 'app'**，并且看到立方体和 Cardboard 分屏视图，没有编译错误。

# 创建一个球体组件

我们的太阳系将由球体构成，代表行星、卫星和太阳。让我们首先创建一个`Sphere`组件。我们将定义一个球体，它是由形成球体表面的顶点三角网格组成的（有关三角网格的更多信息，请参阅[`en.wikipedia.org/wiki/Triangle_mesh`](https://en.wikipedia.org/wiki/Triangle_mesh)）。

右键单击`RenderBoxExt/components`文件夹，选择**New** | **Java Class**，并将其命名为`Sphere`。将其定义为`public class Sphere extends RenderObject`：

```kt
public class Sphere extends RenderObject{
    private static final String TAG = "RenderBox.Sphere";
    public Sphere() {
        super();
        allocateBuffers();
    }
}
```

构造函数调用一个辅助方法`allocateBuffers`，该方法为顶点、法线、纹理和索引分配缓冲区。让我们在类的顶部声明这些变量：

```kt
    public static FloatBuffer vertexBuffer;
    public static FloatBuffer normalBuffer;
    public static FloatBuffer texCoordBuffer;
    public static ShortBuffer indexBuffer;
    public static int numIndices;
```

请注意，我们决定将缓冲区声明为`public`，以便在为对象创建任意纹理材料时具有未来的灵活性。

我们将定义一个半径为 1 的球体。它的顶点由 24 个经度部分（如同一天中的小时）和 16 个纬度部分排列，为我们的目的提供了足够的分辨率。顶部和底部盖子分开处理。这是一个很长的方法，所以我们将为您分解它。这是代码的第一部分，我们在其中声明和初始化变量，包括顶点数组。与我们的`Material`设置方法类似，我们只需要一次性分配`Sphere`缓冲区，在这种情况下，我们使用顶点缓冲区变量来跟踪这种状态。如果它不为空，则已经分配了缓冲区。否则，我们应该继续执行该函数，该函数将设置这个值：

```kt
    public static void allocateBuffers(){
        //Already allocated?
        if (vertexBuffer != null) return;
        //Generate a sphere model
        float radius = 1f;
        // Longitude |||
        int nbLong = 24;
        // Latitude ---
        int nbLat = 16;

        Vector3[] vertices = new Vector3[(nbLong+1) * nbLat + nbLong * 2];
        float _pi = MathUtils.PI;
        float _2pi = MathUtils.PI2;
```

计算顶点位置；首先是顶部和底部，然后沿纬度/经度球形网格：

```kt
        //Top and bottom vertices are duplicated
        for(int i = 0; i < nbLong; i++){
            vertices[i] = new Vector3(Vector3.up).multiply(radius);
            vertices[vertices.length - i - 1] = new Vector3(Vector3.up).multiply(-radius);
        }
        for( int lat = 0; lat < nbLat; lat++ )
        {
            float a1 = _pi * (float)(lat+1) / (nbLat+1);
            float sin1 = (float)Math.sin(a1);
            float cos1 = (float)Math.cos(a1);

            for( int lon = 0; lon <= nbLong; lon++ )
            {
                float a2 = _2pi * (float)(lon == nbLong ? 0 : lon) / nbLong;
                float sin2 = (float)Math.sin(a2);
                float cos2 = (float)Math.cos(a2);

                vertices[lon + lat * (nbLong + 1) + nbLong] = 
                    new Vector3( sin1 * cos2, cos1, sin1 * sin2 ).multiply(radius);
            }
        }
```

接下来，我们计算顶点法线，然后进行纹理映射的 UV：

```kt
        Vector3[] normals = new Vector3[vertices.length];
        for( int n = 0; n < vertices.length; n++ )
            normals[n] = new Vector3(vertices[n]).normalize();

        Vector2[] uvs = new Vector2[vertices.length];
        float uvStart = 1.0f / (nbLong * 2);
        float uvStride = 1.0f / nbLong;
        for(int i = 0; i < nbLong; i++) {
            uvs[i] = new Vector2(uvStart + i * uvStride, 1f);
            uvs[uvs.length - i - 1] = new Vector2(1 - (uvStart + i * uvStride), 0f);
        }
        for( int lat = 0; lat < nbLat; lat++ )
            for( int lon = 0; lon <= nbLong; lon++ )
                uvs[lon + lat * (nbLong + 1) + nbLong] = new Vector2( (float)lon / nbLong, 1f - (float)(lat+1) / (nbLat+1) );
```

同一`allocateBuffers`方法的下一部分生成了连接顶点的三角形索引：

```kt
        int nbFaces = (nbLong+1) * nbLat + 2;
        int nbTriangles = nbFaces * 2;
        int nbIndexes = nbTriangles * 3;
        numIndices = nbIndexes;
        short[] triangles = new short[ nbIndexes ];

        //Top Cap
        int i = 0;
        for( short lon = 0; lon < nbLong; lon++ )
        {
            triangles[i++] = lon;
            triangles[i++] = (short)(nbLong + lon+1);
            triangles[i++] = (short)(nbLong + lon);
        }

        //Middle
        for( short lat = 0; lat < nbLat - 1; lat++ )
        {
            for( short lon = 0; lon < nbLong; lon++ )
            {
                short current = (short)(lon + lat * (nbLong + 1) + nbLong);
                short next = (short)(current + nbLong + 1);

                triangles[i++] = current;
                triangles[i++] = (short)(current + 1);
                triangles[i++] = (short)(next + 1);

                triangles[i++] = current;
                triangles[i++] = (short)(next + 1);
                triangles[i++] = next;
            }
        }

        //Bottom Cap
        for( short lon = 0; lon < nbLong; lon++ )
        {
            triangles[i++] = (short)(vertices.length - lon - 1);
            triangles[i++] = (short)(vertices.length - nbLong - (lon+1) - 1);
            triangles[i++] = (short)(vertices.length - nbLong - (lon) - 1);
        }
```

最后，将这些计算值应用到相应的`vertexBuffer`、`normalBuffer`、`texCoordBuffer`和`indexBuffer`数组中，如下所示：

```kt
        //convert Vector3[] to float[]
        float[] vertexArray = new float[vertices.length * 3];
        for(i = 0; i < vertices.length; i++){
            int step = i * 3;
            vertexArray[step] = vertices[i].x;
            vertexArray[step + 1] = vertices[i].y;
            vertexArray[step + 2] = vertices[i].z;
        }
        float[] normalArray = new float[normals.length * 3];
        for(i = 0; i < normals.length; i++){
            int step = i * 3;
            normalArray[step] = normals[i].x;
            normalArray[step + 1] = normals[i].y;
            normalArray[step + 2] = normals[i].z;
        }
        float[] texCoordArray = new float[uvs.length * 2];
        for(i = 0; i < uvs.length; i++){
            int step = i * 2;
            texCoordArray[step] = uvs[i].x;
            texCoordArray[step + 1] = uvs[i].y;
        }

        vertexBuffer = allocateFloatBuffer(vertexArray);
        normalBuffer = allocateFloatBuffer(normalArray);
        texCoordBuffer = allocateFloatBuffer(texCoordArray);
        indexBuffer = allocateShortBuffer(triangles);
    }
```

这是很多代码，可能在书页上很难阅读；如果您愿意，您可以在项目 GitHub 存储库中找到一份副本。

方便的是，由于球体位于原点（0,0,0）处，每个顶点的法线向量对应于顶点位置本身（从原点辐射到顶点）。严格来说，由于我们使用了半径为 1，我们可以避免“normalize（）”步骤来生成法线数组，以进行优化。以下图显示了 24 x 16 顶点球体及其法线向量：

![创建球体组件](img/B05144_06_01.jpg)

请注意，我们的算法包括一个有趣的修复，避免了极点处的单个顶点（所有 UV 汇聚在一个点上，导致一些旋转的纹理伪影）。

我们在 UV X 上创建*nLon-1*个共同位置的顶点，偏移量为*1/(nLon*2)*，在顶部和底部绘制齿。以下图显示了球体的展平 UV 图，说明了极点的齿状：

![创建球体组件](img/B05144_06_02.jpg)

# 实色光照的球体

我们将从以实色渲染我们的球体开始，但是使用光照着色。和往常一样，我们首先编写着色器函数，其中包括定义它所使用的`Material`中的程序变量。然后，我们将定义`SolidColorLightingMaterial`类，并将其添加到`Sphere`组件中。

## 实色光照着色器

在之前的章节中，我们在顶点着色器中进行了光照计算。这更简单（也更快），但是将计算转移到片段着色器会产生更好的结果。原因是，在顶点着色器中，你只有一个法线值可以与光线方向进行比较。在片段着色器中，所有顶点属性都被插值，这意味着在两个顶点之间的给定点的法线值将是它们两个法线之间的某个点。当这种情况发生时，你会看到三角形面上的平滑渐变，而不是在每个顶点周围的局部阴影伪影。我们将创建一个新的`Material`类来在片段着色器中实现光照。

如果需要，为着色器创建一个 Android 资源目录（资源类型：`raw`），`res/raw/`。然后，创建`solid_color_lighting_vertex.shader`和`res/raw/solid_color_lighting_fragment.shader`文件，并定义如下。

文件：`res/raw/solid_color_lighting_vertex.shader`

```kt
uniform mat4 u_MVP;
uniform mat4 u_MV;

attribute vec4 a_Position;
attribute vec3 a_Normal;

varying vec3 v_Position;
varying vec3 v_Normal;

void main() {
    // vertex in eye space
    v_Position = vec3(u_MV * a_Position);

    // normal's orientation in eye space
    v_Normal = vec3(u_MV * vec4(a_Normal, 0.0));

    // point in normalized screen coordinates
    gl_Position = u_MVP * a_Position;
}
```

请注意，我们为`u_MV`和`u_MVP`分别有单独的统一变量。另外，如果你还记得，在上一章中，我们将光照模型与实际模型分开，因为我们不希望缩放影响光照计算。同样，投影矩阵只对将相机 FOV 应用于顶点位置有用，并且会干扰光照计算。

文件：`res/raw/solid_color_lighting_fragment.shader`

```kt
precision mediump float; // default medium precision in the fragment shader
uniform vec3 u_LightPos; // light position in eye space
uniform vec4 u_LightCol;
uniform vec4 u_Color;

varying vec3 v_Position;        
varying vec3 v_Normal;
varying vec2 v_TexCoordinate;   

void main() {
    // distance for attenuation.
    float distance = length(u_LightPos - v_Position);

    // lighting direction vector from the light to the vertex
    vec3 lightVector = normalize(u_LightPos - v_Position);

    // dot product of the light vector and vertex normal. // If the normal and light vector are
    // pointing in the same direction then it will get max // illumination.
    float diffuse = max(dot(v_Normal, lightVector), 0.01);

    // Add a tiny bit of ambient lighting (this is outerspace)
    diffuse = diffuse + 0.025;  

    // Multiply color by the diffuse illumination level and // texture value to get final output color
    gl_FragColor = u_Color * u_LightCol * diffuse;
}
```

## 纯色光照材质

接下来，我们为着色器定义`Material`类。在 materials 文件夹中，创建一个名为`SolidColorLightingMaterial`的新 Java 类，并定义如下：

```kt
public class SolidColorLightingMaterial extends Material {
    private static final String TAG = "solidcolorlighting";

}
```

添加颜色、程序引用和缓冲区的变量，如下面的代码所示：

```kt
    float[] color = new float[4];
    static int program = -1;
    static int positionParam;
    static int colorParam;
    static int normalParam;
    static int modelParam;
    static int MVParam;
    static int MVPParam;
    static int lightPosParam;
    static int lightColParam;

    FloatBuffer vertexBuffer;
    FloatBuffer normalBuffer;
    ShortBuffer indexBuffer;
    int numIndices;
```

现在，我们可以添加一个构造函数，它接收一个颜色（RGBA）值并设置着色器程序，如下所示：

```kt
    public SolidColorLightingMaterial(float[] c){
        super();
        setColor(c);
        setupProgram();
    }

    public void setColor(float[] c){
        color = c;
    }
```

正如我们之前所见，`setupProgram`方法创建了着色器程序并获取了对其参数的引用：

```kt
    public static void setupProgram(){
        //Already setup?
        if (program != -1) return;

        //Create shader program
        program = createProgram(R.raw.solid_color_lighting_vertex, R.raw.solid_color_lighting_fragment);

        //Get vertex attribute parameters
        positionParam = GLES20.glGetAttribLocation(program, "a_Position");
        normalParam = GLES20.glGetAttribLocation(program, "a_Normal");

        //Enable them (turns out this is kind of a big deal ;)
        GLES20.glEnableVertexAttribArray(positionParam);
        GLES20.glEnableVertexAttribArray(normalParam);

        //Shader-specific parameters
        colorParam = GLES20.glGetUniformLocation(program, "u_Color");
        MVParam = GLES20.glGetUniformLocation(program, "u_MV");
        MVPParam = GLES20.glGetUniformLocation(program, "u_MVP");
        lightPosParam = GLES20.glGetUniformLocation(program, "u_LightPos");
        lightColParam = GLES20.glGetUniformLocation(program, "u_LightCol");

        RenderBox.checkGLError("Solid Color Lighting params");
    }
```

同样，我们添加一个`setBuffers`方法，该方法由`RenderObject`组件（`Sphere`）调用：

```kt
    public void setBuffers(FloatBuffer vertexBuffer, FloatBuffer normalBuffer, ShortBuffer indexBuffer, int numIndices){
        this.vertexBuffer = vertexBuffer;
        this.normalBuffer = normalBuffer;
        this.indexBuffer = indexBuffer;
        this.numIndices = numIndices;
    }
```

最后，添加`draw`代码，该代码将从`Camera`组件中调用，以渲染在缓冲区中准备的几何图形（通过`setBuffers`）。`draw`方法如下所示：

```kt
    @Override
    public void draw(float[] view, float[] perspective) {
        GLES20.glUseProgram(program);

        GLES20.glUniform3fv(lightPosParam, 1, RenderBox.instance.mainLight.lightPosInEyeSpace, 0);
        GLES20.glUniform4fv(lightColParam, 1, RenderBox.instance.mainLight.color, 0);

        Matrix.multiplyMM(modelView, 0, view, 0, RenderObject.lightingModel, 0);
        // Set the ModelView in the shader, // used to calculate lighting
        GLES20.glUniformMatrix4fv(MVParam, 1, false, modelView, 0);
        Matrix.multiplyMM(modelView, 0, view, 0, RenderObject.model, 0);
        Matrix.multiplyMM(modelViewProjection, 0, perspective, 0, modelView, 0);
        // Set the ModelViewProjection matrix for eye position.
        GLES20.glUniformMatrix4fv(MVPParam, 1, false, modelViewProjection, 0);

        GLES20.glUniform4fv(colorParam, 1, color, 0);

        //Set vertex attributes
        GLES20.glVertexAttribPointer(positionParam, 3, GLES20.GL_FLOAT, false, 0, vertexBuffer);
        GLES20.glVertexAttribPointer(normalParam, 3, GLES20.GL_FLOAT, false, 0, normalBuffer);

        GLES20.glDrawElements(GLES20.GL_TRIANGLES, numIndices, GLES20.GL_UNSIGNED_SHORT, indexBuffer);
    }
```

现在我们有了一个纯色光照材质和着色器，我们可以将它们添加到`Sphere`类中，以在我们的项目中使用。

## 向球体添加材质

为了将这个`Material`与`Sphere`一起使用，我们将定义一个新的构造函数（`Sphere`），它调用一个辅助方法（`createSolidColorLightingMaterial`）来创建材质并设置缓冲区。代码如下：

```kt
    public Sphere(float[] color) {
        super();
        allocateBuffers();
        createSolidColorLightingMaterial(color);
    }

    public Sphere createSolidColorLightingMaterial(float[] color){
        SolidColorLightingMaterial mat = new SolidColorLightingMaterial(color);
        mat.setBuffers(vertexBuffer, normalBuffer, indexBuffer, numIndices);
        material = mat;
        return this;
    }
```

好的，现在我们可以将球体添加到我们的场景中。

## 查看球体

让我们看看这是什么样子！我们将创建一个带有球体、灯光和相机的场景。请记住，幸运的是，`RenderBox`类为我们创建了默认的`Camera`和`Light`实例。我们只需要添加`Sphere`组件。

编辑你的`MainActivity.java`文件，在`setup`中添加球体。我们将它着色为黄色，并将其定位在*x*、*y*、*z*位置（2，-2，5）：

```kt
    private Transform sphere;

    @Override
    public void setup() {
        sphere = new Transform();
        float[] color = new float[]{1, 1, 0.5f, 1};
        sphere.addComponent(new Sphere(color));
        sphere.setLocalPosition(2.0f, -2.f, -5.0f);
    }
```

这就是它应该看起来的样子，一对金色的立体对球：

查看球体

如果你看到了我看到的，你就应该得到一个奖励！

# 添加地球纹理材质

接下来，我们将通过在球体表面上渲染纹理来将我们的球体改造成地球的球体。

着色器可能会变得非常复杂，实现各种高光、反射、阴影等。一个更简单的算法，仍然利用颜色纹理和光照的是漫反射材质。这就是我们将在这里使用的。漫反射一词指的是光线在表面上的扩散，而不是反射或闪亮（镜面光照）。

纹理只是一个可以映射（投影）到几何表面上的图像文件（例如`.jpg`）。由于球体不容易被扁平化或剥离成二维地图（正如几个世纪的制图师所证明的那样），纹理图像将会看起来扭曲。以下是我们将用于地球的纹理。（本书的下载文件中提供了此文件的副本，类似的文件也可以在互联网上找到[`www.solarsystemscope.com/nexus/textures/`](http://www.solarsystemscope.com/nexus/textures/)）：

+   在我们的应用程序中，我们计划将图像资产打包到`res/drawable`文件夹中。如果需要，现在创建此文件夹。

+   将`earth_tex.png`文件添加到其中。

`earth_tex`纹理如下图所示：

![添加地球纹理材质](img/B05144_06_04.jpg)

## 加载纹理文件

现在我们需要一个函数将纹理加载到我们的应用程序中。我们可以将其添加到`MainActivity`中。或者，您可以直接将其添加到`RenderBox`库的`RenderObject`类中。（现在将其添加到`MainActivity`中，我们将在本章末将其与其他扩展一起移动到库中。）添加以下代码：

```kt
    public static int loadTexture(final int resourceId){
        final int[] textureHandle = new int[1];

        GLES20.glGenTextures(1, textureHandle, 0);

        if (textureHandle[0] != 0)
        {
            final BitmapFactory.Options options = new BitmapFactory.Options();
            options.inScaled = false;   // No pre-scaling

            // Read in the resource
            final Bitmap bitmap = BitmapFactory.decodeResource(RenderBox.instance.mainActivity.getResources(), resourceId, options);
            // Bind to the texture in OpenGL
            GLES20.glBindTexture(GLES20.GL_TEXTURE_2D, textureHandle[0]);

            // Set filtering
            GLES20.glTexParameteri(GLES20.GL_TEXTURE_2D, GLES20.GL_TEXTURE_MIN_FILTER, GLES20.GL_NEAREST);
            GLES20.glTexParameteri(GLES20.GL_TEXTURE_2D, GLES20.GL_TEXTURE_MAG_FILTER, GLES20.GL_NEAREST);

            // Load the bitmap into the bound texture.
            GLUtils.texImage2D(GLES20.GL_TEXTURE_2D, 0, bitmap, 0);

            // Recycle the bitmap, since its data has been loaded // into OpenGL.
            bitmap.recycle();
        }

        if (textureHandle[0] == 0)
        {
            throw new RuntimeException("Error loading texture.");
        }

        return textureHandle[0];
    }
```

`loadTexture`方法返回一个整数句柄，该句柄可用于引用加载的纹理数据。

## 漫反射光照着色器

现在您可能已经熟悉了，我们将创建一个新的`Material`，它使用新的着色器。我们现在将编写着色器。在`res/raw`文件夹中创建两个文件，分别命名为`diffuse_lighting_vertex.shader`和`diffuse_lighting_fragment.shader`，并定义如下。

文件：`res/raw/diffuse_lighting_vertex.shader`

```kt
uniform mat4 u_MVP;
uniform mat4 u_MV;

attribute vec4 a_Position;
attribute vec3 a_Normal;
attribute vec2 a_TexCoordinate;

varying vec3 v_Position;
varying vec3 v_Normal;
varying vec2 v_TexCoordinate;

void main() {
    // vertex in eye space
    v_Position = vec3(u_MV * a_Position);

    // pass through the texture coordinate.
    v_TexCoordinate = a_TexCoordinate;

    // normal's orientation in eye space
    v_Normal = vec3(u_MV * vec4(a_Normal, 0.0));

    // final point in normalized screen coordinates
    gl_Position = u_MVP * a_Position;
}
```

文件：`res/raw/diffuse_lighting_fragment.shader`

```kt
precision highp float; // default high precision for floating point ranges of the planets

uniform vec3 u_LightPos;        // light position in eye space
uniform vec4 u_LightCol;
uniform sampler2D u_Texture;    // the input texture

varying vec3 v_Position;
varying vec3 v_Normal;
varying vec2 v_TexCoordinate;

void main() {
    // distance for attenuation.
    float distance = length(u_LightPos - v_Position);

    // lighting direction vector from the light to the vertex
    vec3 lightVector = normalize(u_LightPos - v_Position);

    // dot product of the light vector and vertex normal. // If the normal and light vector are
    // pointing in the same direction then it will get max // illumination.
    float diffuse = max(dot(v_Normal, lightVector), 0.01);

    // Add a tiny bit of ambient lighting (this is outerspace)
    diffuse = diffuse + 0.025;

    // Multiply the color by the diffuse illumination level and // texture value to get final output color
    gl_FragColor = texture2D(u_Texture, v_TexCoordinate) * u_LightCol * diffuse;
}
```

这些着色器为光源添加属性，并利用顶点上的几何法向量来计算着色。您可能已经注意到，这与纯色着色器之间的区别在于使用了`texture2D`，这是一个**采样器函数**。另外，请注意我们将`u_Texture`声明为`sampler2D`。这种变量类型和函数利用了内置到 GPU 硬件中的纹理单元，并可以与 UV 坐标一起用于从纹理图像返回颜色值。根据图形硬件的不同，纹理单元的数量是固定的。您可以使用 OpenGL 查询纹理单元的数量。对于移动 GPU 来说，一个很好的经验法则是期望有八个纹理单元。这意味着任何着色器可以同时使用多达八个纹理。

## 漫反射光照材质

现在我们可以编写一个使用纹理和着色器的`Material`。在`materials/`文件夹中，创建一个名为`DiffuseLightingMaterial`的新 Java 类，如下所示：

```kt
public class DiffuseLightingMaterial extends Material {
    private static final String TAG = "diffuselightingmaterial";
```

添加纹理 ID、程序引用和缓冲区的变量，如下所示：

```kt
    int textureId;
    static int program = -1; //Initialize to a totally invalid value for setup state
    static int positionParam;
    static int texCoordParam;
    static int textureParam;
    static int normalParam;
    static int MVParam;    
    static int MVPParam;
    static int lightPosParam;
    static int lightColParam;

    FloatBuffer vertexBuffer;
    FloatBuffer texCoordBuffer;
    FloatBuffer normalBuffer;
    ShortBuffer indexBuffer;
    int numIndices;
```

现在我们可以添加一个构造函数，该构造函数设置着色器程序并为给定的资源 ID 加载纹理，如下所示：

```kt
    public DiffuseLightingMaterial(int resourceId){
        super();
        setupProgram();
        this.textureId = MainActivity.loadTexture(resourceId);
    }
```

正如我们之前所见，`setupProgram`方法创建着色器程序并获取其参数的引用：

```kt
    public static void setupProgram(){
        //Already setup?
        if (program != -1) return;

        //Create shader program
        program = createProgram(R.raw.diffuse_lighting_vertex, R.raw.diffuse_lighting_fragment);
        RenderBox.checkGLError("Diffuse Texture Color Lighting shader compile");

        //Get vertex attribute parameters
        positionParam = GLES20.glGetAttribLocation(program, "a_Position");
        normalParam = GLES20.glGetAttribLocation(program, "a_Normal");
        texCoordParam = GLES20.glGetAttribLocation(program, "a_TexCoordinate");

        //Enable them (turns out this is kind of a big deal ;)
        GLES20.glEnableVertexAttribArray(positionParam);
        GLES20.glEnableVertexAttribArray(normalParam);
        GLES20.glEnableVertexAttribArray(texCoordParam);

        //Shader-specific parameters
        textureParam = GLES20.glGetUniformLocation(program, "u_Texture");
        MVParam = GLES20.glGetUniformLocation(program, "u_MV");
        MVPParam = GLES20.glGetUniformLocation(program, "u_MVP");
        lightPosParam = GLES20.glGetUniformLocation(program, "u_LightPos");
        lightColParam = GLES20.glGetUniformLocation(program, "u_LightCol");

        RenderBox.checkGLError("Diffuse Texture Color Lighting params");
    }
```

同样，我们添加一个`setBuffers`方法，该方法由`RenderObject`组件（`Sphere`）调用：

```kt
    public void setBuffers(FloatBuffer vertexBuffer, FloatBuffer normalBuffer, FloatBuffer texCoordBuffer, ShortBuffer indexBuffer, int numIndices){
        //Associate VBO data with this instance of the material
        this.vertexBuffer = vertexBuffer;
        this.normalBuffer = normalBuffer;
        this.texCoordBuffer = texCoordBuffer;
        this.indexBuffer = indexBuffer;
        this.numIndices = numIndices;
    }
```

最后，添加`draw`代码，该代码将从`Camera`组件调用，以渲染通过`setBuffers`准备的几何图形。`draw`方法如下所示：

```kt
    @Override
    public void draw(float[] view, float[] perspective) {
        GLES20.glUseProgram(program);

        // Set the active texture unit to texture unit 0.
        GLES20.glActiveTexture(GLES20.GL_TEXTURE0);

        // Bind the texture to this unit.
        GLES20.glBindTexture(GLES20.GL_TEXTURE_2D, textureId);

        // Tell the texture uniform sampler to use this texture in // the shader by binding to texture unit 0.
        GLES20.glUniform1i(textureParam, 0);

        //Technically, we don't need to do this with every draw //call, but the light could move.
        //We could also add a step for shader-global parameters //which don't vary per-object
        GLES20.glUniform3fv(lightPosParam, 1, RenderBox.instance.mainLight.lightPosInEyeSpace, 0);
        GLES20.glUniform4fv(lightColParam, 1, RenderBox.instance.mainLight.color, 0);

        Matrix.multiplyMM(modelView, 0, view, 0, RenderObject.lightingModel, 0);
        // Set the ModelView in the shader, used to calculate // lighting
        GLES20.glUniformMatrix4fv(MVParam, 1, false, modelView, 0);
        Matrix.multiplyMM(modelView, 0, view, 0, RenderObject.model, 0);
        Matrix.multiplyMM(modelViewProjection, 0, perspective, 0, modelView, 0);
        // Set the ModelViewProjection matrix for eye position.
        GLES20.glUniformMatrix4fv(MVPParam, 1, false, modelViewProjection, 0);

        //Set vertex attributes
        GLES20.glVertexAttribPointer(positionParam, 3, GLES20.GL_FLOAT, false, 0, vertexBuffer);
        GLES20.glVertexAttribPointer(normalParam, 3, GLES20.GL_FLOAT, false, 0, normalBuffer);
        GLES20.glVertexAttribPointer(texCoordParam, 2, GLES20.GL_FLOAT, false, 0, texCoordBuffer);

        GLES20.glDrawElements(GLES20.GL_TRIANGLES, numIndices, GLES20.GL_UNSIGNED_SHORT, indexBuffer);

        RenderBox.checkGLError("Diffuse Texture Color Lighting draw");
    }
}
```

与我们之前定义的`SolidColorLightingMaterial`类进行比较，您会注意到它非常相似。我们用纹理 ID 替换了单一颜色，并添加了由`Sphere`组件给出的纹理坐标缓冲区（`texCoordBuffer`）的要求。另外，请注意我们将活动纹理单元设置为`GL_TEXTURE0`并绑定纹理。

## 向 Sphere 组件添加漫反射光照纹理

要将新的材质添加到`Sphere`组件中，我们将创建一个接收纹理句柄的替代构造函数。然后，它创建`DiffuseLightingMaterial`类的一个实例，并从球体设置缓冲区。

通过定义一个接受纹理 ID 并调用一个名为`createDiffuseMaterial`的新辅助方法的新构造函数(`Sphere`)，将材料添加到`Sphere`组件中，如下所示：

```kt
    public Sphere(int textureId){
        super();
        allocateBuffers();
        createDiffuseMaterial(textureId);
    }

    public Sphere createDiffuseMaterial(int textureId){
        DiffuseLightingMaterial mat = new DiffuseLightingMaterial(textureId);
        mat.setBuffers(vertexBuffer, normalBuffer, texCoordBuffer, indexBuffer, numIndices);
        material = mat;
        return this;
    }
```

现在，我们可以使用纹理材料。

## 查看地球

要将地球纹理添加到我们的球体中，修改`MainActivity`的`setup`方法，以指定纹理资源 ID 而不是颜色，如下所示：

```kt
    @Override
    public void setup() {
        sphere = new Transform();
        sphere.addComponent(new Sphere(R.drawable.earth_tex));
        sphere.setLocalPosition(2.0f, -2.f, -2.0f);
    }
```

你看，*家，甜蜜的家！*

查看地球

看起来真的很酷。哎呀，它是颠倒的！虽然在外太空中没有特定的上下，但我们的地球看起来颠倒了。让我们在`setup`方法中翻转它，以便它以正确的方向开始，而且在此过程中，让我们利用`Transform`方法返回它们自己的事实，这样我们就可以链接调用，如下所示：

```kt
    public void setup() {
        sphere = new Transform()
            .setLocalPosition(2.0f, -2.f, -2.0f)
            .rotate(0, 0, 180f)
            .addComponent(new Sphere(R.drawable.earth_tex));
    }
```

当然，地球应该旋转。让我们对其进行动画处理，使其旋转，就像我们期望地球会做的那样。将此添加到`preDraw`方法中，该方法在每个新帧之前调用。它使用`Time`类的`getDeltaTime`方法，该方法返回自上一帧以来的当前秒数变化的分数。如果我们希望它每秒旋转-10 度，我们使用*-10 * deltaTime*：

```kt
    public void preDraw() {
        float dt = Time.getDeltaTime();
        sphere.rotate( 0, -10f * dt, 0);
    }
```

我觉得这样很好！你觉得呢？

## 改变相机位置

还有一件事。我们似乎是在与光源一致地观看地球。让我们移动相机视图，这样我们就可以从侧面看到地球。这样，我们可以更好地看到受光照的阴影。

假设我们将光源位置保留在原点(0,0,0)，就好像它是太阳在太阳系的中心。地球距离太阳 1.471 亿公里。让我们将球体放在原点右侧那么多单位，并将相机放在相同的相对位置。现在，`setup`方法看起来像以下代码：

```kt
    public void setup() {
        sphere = new Transform()
            .setLocalPosition(147.1f, 0, 0)
            .rotate(0, 0, 180f)
            .addComponent(new Sphere(R.drawable.earth_tex));
        RenderBox.mainCamera.getTransform().setLocalPosition(147.1f, 2f, 2f);
    }
```

运行它，你会看到这个：

![改变相机位置](img/B05144_06_06.jpg)

看起来几乎真实，是吗？NASA 会感到自豪！

# 白天和夜晚材料

不过，地球的背面看起来异常黑暗。我的意思是，现在不是 18 世纪。现在很多东西都是 24 x 7，尤其是我们的城市。让我们用一个单独的地球夜晚纹理来表示这一点，其中有城市灯光。

我们有一个文件供您使用，名为`earth_night_tex.jpg`。将文件的副本拖入您的`res/drawable/`文件夹中。

在这本书的页面上可能有点难以辨认，但这就是纹理图像的样子：

![白天和夜晚材料](img/B05144_06_07.jpg)

## 白天/夜晚着色器

为了支持这一点，我们将创建一个新的`DayNightMaterial`类，该类接受地球纹理的两个版本。该材料还将包含相应的片段着色器，该着色器考虑了表面相对于光源方向的法线向量（使用点积，如果您熟悉矢量数学）来决定是使用白天纹理图像还是夜晚纹理图像进行渲染。

在你的`res/raw/`文件夹中，创建`day_night_vertex.shader`和`day_night_fragment.shader`文件，然后定义它们，如下所示。

文件：`day_night_vertex.shader`

```kt
uniform mat4 u_MVP;
uniform mat4 u_MV;

attribute vec4 a_Position;
attribute vec3 a_Normal;
attribute vec2 a_TexCoordinate;

varying vec3 v_Position;
varying vec3 v_Normal;
varying vec2 v_TexCoordinate;

void main() {
   // vertex to eye space
   v_Position = vec3(u_MV * a_Position);

   // pass through the texture coordinate
   v_TexCoordinate = a_TexCoordinate;

   // normal's orientation in eye space
   v_Normal = vec3(u_MV * vec4(a_Normal, 0.0));

   // final point in normalized screen coordinates
   gl_Position = u_MVP * a_Position;
}
```

除了添加`v_Texcoordinate`之外，这与我们的`SolidColorLighting`着色器完全相同。

文件：`day_night_fragment.shader`

```kt
precision highp float; //  default high precision for floating point ranges of the //  planets
uniform vec3 u_LightPos;      // light position in eye space
uniform vec4 u_LightCol;
uniform sampler2D u_Texture;  // the day texture.
uniform sampler2D u_NightTexture;    // the night texture.

varying vec3 v_Position;
varying vec3 v_Normal;
varying vec2 v_TexCoordinate;

void main() {
    // lighting direction vector from the light to the vertex
    vec3 lightVector = normalize(u_LightPos - v_Position);

    // dot product of the light vector and vertex normal. If the // normal and light vector are
    // pointing in the same direction then it will get max // illumination.
    float ambient = 0.3;
    float dotProd = dot(v_Normal, lightVector);
    float blend = min(1.0, dotProd * 2.0);
    if(dotProd < 0.0){
        //flat ambient level of 0.3
        gl_FragColor = texture2D(u_NightTexture, v_TexCoordinate) * ambient;
    } else {
        gl_FragColor = (
            texture2D(u_Texture, v_TexCoordinate) * blend
            + texture2D(u_NightTexture, v_TexCoordinate) * (1.0 - blend)
        ) * u_LightCol * min(max(dotProd * 2.0, ambient), 1.0);
    }
}
```

与往常一样，对于照明，我们计算顶点法线向量和光线方向向量的点积(`dotProd`)。当该值为负时，顶点面向远离光源（太阳），因此我们将使用夜晚纹理进行渲染。否则，我们将使用常规白天地球纹理进行渲染。

光照计算还包括混合值。这基本上是一种在计算`gl_FragColor`变量时将过渡区域压缩到终结器周围的方法。我们将点积乘以 2.0，使其遵循更陡的斜率，但仍将混合值夹在 0 和 1 之间。这有点复杂，但一旦你考虑数学，它应该有些意义。

我们使用两种纹理来绘制相同的表面。虽然这可能看起来是独特的白天/黑夜情况，但实际上这是一种非常常见的方法，称为多重纹理。你可能不相信，但在引入一次使用多个纹理的能力之前，3D 图形实际上已经取得了相当大的进展。如今，你几乎可以在任何地方看到多重纹理，实现诸如法线贴图、贴花纹理和位移/视差着色器等技术，这些技术可以在简单的网格上创建更多的细节。

## DayNightMaterial 类

现在我们可以编写`DayNightMaterial`类。它基本上类似于我们之前创建的`DiffuseLightingMaterial`类，但支持两种纹理。因此，构造函数接受两个纹理 ID。`setBuffers`方法与之前的方法相同，`draw`方法几乎相同，但增加了夜间纹理的绑定。

以下是完整的代码，突出显示与`DiffuseLightingMaterial`不同的行：

```kt
public class DayNightMaterial extends Material {
    private static final String TAG = "daynightmaterial";
```

与我们其他材料一样，声明我们需要的变量，包括白天和夜晚的纹理 ID：

```kt
    int textureId;
    int nightTextureId;

    static int program = -1; //Initialize to a totally invalid value for setup state
    static int positionParam;
    static int texCoordParam;
    static int textureParam;
    static int nightTextureParam;
    static int normalParam;
    static int MVParam;
    static int MVPParam;
    static int lightPosParam;
    static int lightColParam;

    FloatBuffer vertexBuffer;
    FloatBuffer texCoordBuffer;
    FloatBuffer normalBuffer;
    ShortBuffer indexBuffer;
    int numIndices;
```

定义接受资源 ID 和`setupProgram`辅助方法的构造函数：

```kt
    public DayNightMaterial(int resourceId, int nightResourceId){
        super();
        setupProgram();
        this.textureId = MainActivity.loadTexture(resourceId);

        this.nightTextureId = MainActivity.loadTexture(nightResourceId);
    }

    public static void setupProgram(){
        if(program != -1) return;
        //Create shader program
        program = createProgram(R.raw.day_night_vertex, R.raw.day_night_fragment);

        //Get vertex attribute parameters
        positionParam = GLES20.glGetAttribLocation(program, "a_Position");
        normalParam = GLES20.glGetAttribLocation(program, "a_Normal");
        texCoordParam = GLES20.glGetAttribLocation(program, "a_TexCoordinate");

        //Enable them (turns out this is kind of a big deal ;)
        GLES20.glEnableVertexAttribArray(positionParam);
        GLES20.glEnableVertexAttribArray(normalParam);
        GLES20.glEnableVertexAttribArray(texCoordParam);

        //Shader-specific parameters
        textureParam = GLES20.glGetUniformLocation(program, "u_Texture");
        nightTextureParam = GLES20.glGetUniformLocation(program, "u_NightTexture");
        MVParam = GLES20.glGetUniformLocation(program, "u_MV");
        MVPParam = GLES20.glGetUniformLocation(program, "u_MVP");
        lightPosParam = GLES20.glGetUniformLocation(program, "u_LightPos");
        lightColParam = GLES20.glGetUniformLocation(program, "u_LightCol");

        RenderBox.checkGLError("Day/Night params");
    }

    public void setBuffers(FloatBuffer vertexBuffer, FloatBuffer normalBuffer, FloatBuffer texCoordBuffer, ShortBuffer indexBuffer, int numIndices){
        //Associate VBO data with this instance of the material
        this.vertexBuffer = vertexBuffer;
        this.normalBuffer = normalBuffer;
        this.texCoordBuffer = texCoordBuffer;
        this.indexBuffer = indexBuffer;
        this.numIndices = numIndices;
    }
```

最后，`draw`方法将所有内容输出到屏幕上：

```kt
    @Override
    public void draw(float[] view, float[] perspective) {
        GLES20.glUseProgram(program);

        // Set the active texture unit to texture unit 0.
        GLES20.glActiveTexture(GLES20.GL_TEXTURE0);

        // Bind the texture to this unit.
        GLES20.glBindTexture(GLES20.GL_TEXTURE_2D, textureId);

        GLES20.glActiveTexture(GLES20.GL_TEXTURE1);
        GLES20.glBindTexture(GLES20.GL_TEXTURE_2D, nightTextureId);

        // Tell the texture uniform sampler to use this texture in // the shader by binding to texture unit 0.
        GLES20.glUniform1i(textureParam, 0);
        GLES20.glUniform1i(nightTextureParam, 1);

        //Technically, we don't need to do this with every draw //call, but the light could move.
        //We could also add a step for shader-global parameters //which don't vary per-object
        GLES20.glUniform3fv(lightPosParam, 1, RenderBox.instance.mainLight.lightPosInEyeSpace, 0);
        GLES20.glUniform4fv(lightColParam, 1, RenderBox.instance.mainLight.color, 0);

        Matrix.multiplyMM(modelView, 0, view, 0, RenderObject.lightingModel, 0);
        // Set the ModelView in the shader, used to calculate // lighting
        GLES20.glUniformMatrix4fv(MVParam, 1, false, modelView, 0);
        Matrix.multiplyMM(modelView, 0, view, 0, RenderObject.model, 0);
        Matrix.multiplyMM(modelViewProjection, 0, perspective, 0, modelView, 0);
        // Set the ModelViewProjection matrix for eye position.
        GLES20.glUniformMatrix4fv(MVPParam, 1, false, modelViewProjection, 0);

        //Set vertex attributes
        GLES20.glVertexAttribPointer(positionParam, 3, GLES20.GL_FLOAT, false, 0, vertexBuffer);
        GLES20.glVertexAttribPointer(normalParam, 3, GLES20.GL_FLOAT, false, 0, normalBuffer);
        GLES20.glVertexAttribPointer(texCoordParam, 2, GLES20.GL_FLOAT, false, 0, texCoordBuffer);

        GLES20.glDrawElements(GLES20.GL_TRIANGLES, numIndices, GLES20.GL_UNSIGNED_SHORT, indexBuffer);

        RenderBox.checkGLError("DayNight Texture Color Lighting draw");
    }
}
```

## 渲染白天/黑夜

现在我们准备将新材料整合到我们的`Sphere`组件中，看看效果如何。

在`Sphere.java`中，添加一个新的构造函数和`createDayNightMaterial`辅助方法，如下：

```kt
    public Sphere(int textureId, int nightTextureId){
        super();
        allocateBuffers();
        createDayNightMaterial(textureId, nightTextureId);
    }

    public Sphere createDayNightMaterial(int textureId, int nightTextureId){
        DayNightMaterial mat = new DayNightMaterial(textureId, nightTextureId);
        mat.setBuffers(vertexBuffer, normalBuffer, texCoordBuffer, indexBuffer, numIndices);
        material = mat;
        return this;
    }
```

让我们从`MainActivity`的`setup`方法中调用它，并用新的`Sphere`实例替换调用，传递两个纹理的资源 ID：

```kt
    .addComponent(new Sphere(R.drawable.earth_tex, R.drawable.earth_night_tex));
```

现在运行。看起来真酷！高雅！不幸的是，在这里粘贴屏幕截图并没有太多意义，因为城市夜间灯光不会显示得很好。你只能在自己的 Cardboard 观看器中看到它。相信我，这是值得的！

接下来，*太阳来了，我说，没问题...*

# 创建太阳

太阳将被渲染为一个带纹理的球体。然而，它不像我们的地球那样有正面和背面的阴影。我们需要以未照亮或者说未着色的方式渲染它。这意味着我们需要创建`UnlitTextureMaterial`。

我们也有太阳的纹理文件（以及所有行星）。尽管它们包含在本书的可下载文件中，但我们不会在本章中展示它们。

将`sun_tex.png`文件拖到`res/drawable/`文件夹中。

## 未照亮的纹理着色器

正如我们在本书中早些时候看到的那样，未照亮的着色器比带照明的着色器简单得多。在`res/raw/`文件夹中，创建`unlit_tex_vertex.shader`和`unlit_tex_fragment.shader`文件，并定义它们如下。

文件：`unlit_tex_vertex.shader`

```kt
uniform mat4 u_MVP;

attribute vec4 a_Position;
attribute vec2 a_TexCoordinate;

varying vec3 v_Position;
varying vec2 v_TexCoordinate;

void main() {
   // pass through the texture coordinate
   v_TexCoordinate = a_TexCoordinate;

   // final point in normalized screen coordinates
   gl_Position = u_MVP * a_Position;
}
```

文件：`unlit_tex_fragment.shader`

```kt
precision mediump float;        // default medium precision
uniform sampler2D u_Texture;    // the input texture

varying vec3 v_Position;
varying vec2 v_TexCoordinate;

void main() {
    // Send the color from the texture straight out
    gl_FragColor = texture2D(u_Texture, v_TexCoordinate);
}
```

是的，这比我们之前的着色器简单。

## 未照亮的纹理材料

现在，我们可以编写`UnlitTexMaterial`类。以下是初始代码：

```kt
public class UnlitTexMaterial extends Material {
    private static final String TAG = "unlittex";

    int textureId;

    static int program = -1; //Initialize to a totally invalid value for setup state
    static int positionParam;
    static int texCoordParam;
    static int textureParam;
    static int MVPParam;

    FloatBuffer vertexBuffer;
    FloatBuffer texCoordBuffer;
    ShortBuffer indexBuffer;
    int numIndices;
```

以下是构造函数，`setupProgram`和`setBuffers`方法：

```kt
    public UnlitTexMaterial(int resourceId){
        super();
        setupProgram();
        this.textureId = MainActivity.loadTexture(resourceId);
    }

    public static void setupProgram(){
        if(program != -1) return;
        //Create shader program
        program = createProgram(R.raw.unlit_tex_vertex, R.raw.unlit_tex_fragment);

        //Get vertex attribute parameters
        positionParam = GLES20.glGetAttribLocation(program, "a_Position");
        texCoordParam = GLES20.glGetAttribLocation(program, "a_TexCoordinate");

        //Enable them (turns out this is kind of a big deal ;)
        GLES20.glEnableVertexAttribArray(positionParam);
        GLES20.glEnableVertexAttribArray(texCoordParam);

        //Shader-specific parameters
        textureParam = GLES20.glGetUniformLocation(program, "u_Texture");
        MVPParam = GLES20.glGetUniformLocation(program, "u_MVP");

        RenderBox.checkGLError("Unlit Texture params");
    }

    public void setBuffers(FloatBuffer vertexBuffer, FloatBuffer texCoordBuffer, ShortBuffer indexBuffer, int numIndices){
        //Associate VBO data with this instance of the material
        this.vertexBuffer = vertexBuffer;
        this.texCoordBuffer = texCoordBuffer;
        this.indexBuffer = indexBuffer;
        this.numIndices = numIndices;
    }
```

在后续项目中，为纹理 ID 添加获取器和设置器方法会很方便（这里不使用）：

```kt
    public void setTexture(int textureHandle){
        textureId = textureHandle;
    }

      public int getTexture(){
          return textureId;
      }
```

最后，这是`draw`方法：

```kt
    @Override
    public void draw(float[] view, float[] perspective) {
        GLES20.glUseProgram(program);

        // Set the active texture unit to texture unit 0.
        GLES20.glActiveTexture(GLES20.GL_TEXTURE0);

        // Bind the texture to this unit.
        GLES20.glBindTexture(GLES20.GL_TEXTURE_2D, textureId);

        // Tell the texture uniform sampler to use this texture in // the shader by binding to texture unit 0.
        GLES20.glUniform1i(textureParam, 0);

        Matrix.multiplyMM(modelView, 0, view, 0, RenderObject.model, 0);
        Matrix.multiplyMM(modelViewProjection, 0, perspective, 0, modelView, 0);
        // Set the ModelViewProjection matrix in the shader.
        GLES20.glUniformMatrix4fv(MVPParam, 1, false, modelViewProjection, 0);

        // Set the vertex attributes
        GLES20.glVertexAttribPointer(positionParam, 3, GLES20.GL_FLOAT, false, 0, vertexBuffer);
        GLES20.glVertexAttribPointer(texCoordParam, 2, GLES20.GL_FLOAT, false, 0, texCoordBuffer);

        GLES20.glDrawElements(GLES20.GL_TRIANGLES, numIndices, GLES20.GL_UNSIGNED_SHORT, indexBuffer);

        RenderBox.checkGLError("Unlit Texture draw");
    }
}
```

## 使用未照亮的纹理进行渲染

我们准备将新材料整合到我们的`Sphere`类中，看看效果如何。

在`Sphere.java`中，添加一个新的构造函数，该构造函数接受一个`boolean`参数，指示纹理是否应该被照亮，并添加`createUnlitTexMaterial`辅助方法：

```kt
    public Sphere(int textureId, boolean lighting){
        super();
        allocateBuffers();
        if (lighting) {
            createDiffuseMaterial(textureId);
        } else {
            createUnlitTexMaterial(textureId);
        }
    }

    public Sphere createUnlitTexMaterial(int textureId){
        UnlitTexMaterial mat = new UnlitTexMaterial(textureId);
        mat.setBuffers(vertexBuffer, texCoordBuffer, indexBuffer, numIndices);
        material = mat;
        return this;
    }
```

请注意，我们定义构造函数的方式，您可以调用`new Sphere(texId)`或`new Sphere(texId, true)`来获得有光的渲染。但对于未照明的渲染，您必须使用第二个`Sphere(texId, false)`。还要注意，构造函数中设置整个组件并不是唯一的方法。我们之所以这样做，是因为这样可以使我们的`MainActivity`代码简洁。实际上，随着我们开始扩展对`RenderBox`及其着色器库的使用，将有必要将大部分代码放入我们的`MainActivity`类中。为每种材料创建构造函数是不可能的。最终，需要一个材料系统，以允许您创建和设置材料，而无需为每个材料创建一个新类。

## 添加太阳

现在，我们只需要将太阳球添加到`MainActivity`的`setup`方法中。让它变大，比如说，按照 6.963 的比例（记住这是以百万公里为单位）。现在这个值可能看起来是任意的，但当我们对太阳系的几何进行计算并缩放行星时，您将看到它的来源。

将以下代码添加到`MainActivity`的`setup`方法中：

```kt
    public void setup() {
        Transform origin = new Transform();

        //Sun
        Transform sun = new Transform()
            .setParent(origin, false)
            .setLocalScale(6.963f, 6.963f, 6.963f)
            .addComponent(new Sphere(R.drawable.sun_tex, false));

        //"Sun" light
        RenderBox.instance.mainLight.transform.setPosition( origin.getPosition());
        RenderBox.instance.mainLight.color = new float[]{1, 1, 0.8f, 1};

            //Earth…
```

我们首先定义一个原点变换，它将是太阳系的中心。然后，我们创建太阳，将其作为原点的父级，并给定比例。然后，添加一个具有太阳纹理的新球组件。我们还给了我们的光一个略带黄色的颜色，这将与地球的纹理颜色混合。

这是渲染后的太阳的样子，它似乎照亮了地球：

![添加太阳](img/B05144_06_08.jpg)

现在，让我们继续进行太阳系的其余部分。

# 创建一个 Planet 类

在构建我们的太阳系时，将有助于将`Planet`类抽象出来用于每个行星。

行星有许多不同的属性，除了它们的纹理资源 ID 外，还定义了它们独特的特征。行星与太阳的距离、大小（半径）和轨道速度。行星都围绕太阳作为它们的原点进行轨道运行。

+   距离将是其距离太阳的距离，以百万公里为单位进行测量。

+   半径将是行星的大小，以公里为单位（实际上是以百万公里为单位，以保持一致）。

+   旋转是行星绕其自身轴旋转的速率（其一天之一）。

+   轨道是行星绕太阳运行的速率（其一年之一）。我们将假设轨道是完全圆形的。

+   `TexId`是行星纹理图像的资源 ID。

+   `origin`是其轨道的中心。对于行星，这将是太阳的变换。对于卫星，这将是卫星的行星。

太阳系是一个非常庞大的东西。距离和半径以百万公里为单位进行测量。行星之间的距离非常遥远，相对于它们的轨道大小来说，它们的大小相对较小。旋转和轨道值是相对速率。您会注意到，我们将它们标准化为每地球日 10 秒。

根据这些属性，行星保持两个变换：一个变换用于行星本身，另一个变换描述其在轨道上的位置。通过这种方式，我们可以旋转每个行星的单独父级变换，当行星处于局部位置时，其大小等于轨道半径，会导致行星以圆形运动。然后我们可以使用其变换旋转行星本身。

对于月球，我们还将使用`Planet`类（是的，我知道，也许我们应该将其命名为`HeavenlyBody`？），但将其原点设置为地球。月球不会旋转。

在您的应用程序中（例如`app/java/com/cardbookvr/solarsystem/`），创建一个 Java 类并命名为`Planet`。添加其属性（`distance`，`radius`，`rotation`，`orbit`，`orbitTransform`和`transform`）的变量，如下所示：

```kt
public class Planet {
    protected float rotation, orbit;
    protected Transform orbitTransform, transform;

    public float distance, radius;
```

定义一个构造函数，该构造函数接受行星的属性值，初始化变量并计算初始变换：

```kt
    public Planet(float distance, float radius, float rotation, float orbit, int texId, Transform origin){
        setupPlanet(distance, radius, rotation, orbit, origin);
        transform.addComponent(new Sphere(texId));
    }

    public void setupPlanet(float distance, float radius, float rotation, float orbit, Transform origin){
        this.distance = distance;
        this.radius = radius;
        this.rotation = rotation;
        this.orbit = orbit;
        this.orbitTransform = new Transform();
        this.orbitTransform.setParent(origin, false);

        transform = new Transform()
            .setParent(orbitTransform, false)
            .setLocalPosition(distance, 0, 0)
            .setLocalRotation(180, 0, 0)
            .setLocalScale(radius, radius, radius);
    }
```

构造函数为行星生成初始变换，并添加一个带有给定纹理的`Sphere`组件。

在每一帧新的画面中，我们将更新`orbitTransform`绕太阳的旋转（年）和行星绕自身轴的旋转（日）：

```kt
    public void preDraw(float dt){
        orbitTransform.rotate(0, dt * orbit, 0);
        transform.rotate(0, dt * -rotation, 0);
    }
```

我们还可以为`Planet`类的变换提供一些访问器方法：

```kt
    public Transform getTransform() { return transform; }
    public Transform getOrbitransform() { return orbitTransform; }
```

现在，让我们来看看我们太阳系的几何形状。

# 太阳系的形成

这是我们将真正的科学投入到我们的项目中的机会。以下表格显示了每个行星的实际距离、大小、旋转和轨道值。（大部分数据来自[`www.enchantedlearning.com/subjects/astronomy/planets/`](http://www.enchantedlearning.com/subjects/astronomy/planets/)。）

| 行星 | 距离太阳（百万公里） | 半径大小（公里） | 白天长度（地球小时） | 年长度（地球年） |
| --- | --- | --- | --- | --- |
| 水星 | 57.9 | 2440 | 1408.8 | 0.24 |
| 金星 | 108.2 | 6052 | 5832 | 0.615 |
| 地球 | 147.1 | 6371 | 24 | 1.0 |
| 地球的月亮 | 0.363（来自地球） | 1737 | 0 |   |
| 火星 | 227.9 | 3390 | 24.6 | 2.379 |
| 木星 | 778.3 | 69911 | 9.84 | 11.862 |
| 土星 | 1427.0 | 58232 | 10.2 | 29.456 |
| 天王星 | 2871.0 | 25362 | 17.9 | 84.07 |
| 海王星 | 4497 | 24622 | 19.1 | 164.81 |
| 冥王星（仍然计数） | 5913 | 1186 | 6.39 | 247.7 |

我们还为每个行星准备了纹理图像。这些文件包含在本书的下载中。它们应该被添加到`res/drawable`文件夹中，命名为`mercury_tex.png`，`venus_tex.png`等等。以下表格标识了我们使用的来源以及您可以找到它们的位置： 

| 行星 | 纹理 |
| --- | --- |
| 水星 | [`laps.noaa.gov/albers/sos/mercury/mercury/mercury_rgb_cyl_www.jpg`](http://laps.noaa.gov/albers/sos/mercury/mercury/mercury_rgb_cyl_www.jpg) |
| 金星 | [`csdrive.srru.ac.th/55122420119/texture/venus.jpg`](http://csdrive.srru.ac.th/55122420119/texture/venus.jpg) |
| 地球 | [`www.solarsystemscope.com/nexus/content/tc-earth_texture/tc-earth_daymap.jpg`](http://www.solarsystemscope.com/nexus/content/tc-earth_texture/tc-earth_daymap.jpg)夜晚：[`www.solarsystemscope.com/nexus/content/tc-earth_texture/tc-earth_nightmap.jpg`](http://www.solarsystemscope.com/nexus/content/tc-earth_texture/tc-earth_nightmap.jpg) |
| 地球的月亮 | [`farm1.staticflickr.com/120/263411684_ea405ffa8f_o_d.jpg`](https://farm1.staticflickr.com/120/263411684_ea405ffa8f_o_d.jpg) |
| 火星 | [`lh5.ggpht.com/-2aLH6cYiaKs/TdOsBtnpRqI/AAAAAAAAAP4/bnMOdD9OMjk/s9000/mars%2Btexture.jpg`](http://lh5.ggpht.com/-2aLH6cYiaKs/TdOsBtnpRqI/AAAAAAAAAP4/bnMOdD9OMjk/s9000/mars%2Btexture.jpg) |
| 木星 | [`laps.noaa.gov/albers/sos/jupiter/jupiter/jupiter_rgb_cyl_www.jpg`](http://laps.noaa.gov/albers/sos/jupiter/jupiter/jupiter_rgb_cyl_www.jpg) |
| 土星 | [`www.solarsystemscope.com/nexus/content/planet_textures/texture_saturn.jpg`](http://www.solarsystemscope.com/nexus/content/planet_textures/texture_saturn.jpg) |
| 天王星 | [`www.astrosurf.com/nunes/render/maps/full/uranus.jpg`](http://www.astrosurf.com/nunes/render/maps/full/uranus.jpg) |
| 海王星 | [`www.solarsystemscope.com/nexus/content/planet_textures/texture_neptune.jpg`](http://www.solarsystemscope.com/nexus/content/planet_textures/texture_neptune.jpg) |
| 冥王星 | [`www.shatters.net/celestia/files/pluto.jpg`](http://www.shatters.net/celestia/files/pluto.jpg) |
| 太阳 | [`www.solarsystemscope.com/nexus/textures/texture_pack/assets/preview_sun.jpg`](http://www.solarsystemscope.com/nexus/textures/texture_pack/assets/preview_sun.jpg) |
| 银河系 | [`www.geckzilla.com/apod/tycho_cyl_glow.png`](http://www.geckzilla.com/apod/tycho_cyl_glow.png)（由 Judy Schmidt，[`geckzilla.com/`](http://geckzilla.com/)） |

## 在 MainActivity 中设置行星

我们将在`MainActivity`中设置所有行星，使用一个`setupPlanets`方法，该方法将从`setup`中调用。让我们开始吧。

在类的顶部，声明一个 `planets` 数组：

```kt
    Planet[] planets;
```

然后，我们声明了一些常量，我们将在下面解释：

```kt
    // tighten up the distances (millions km)
    float DISTANCE_FACTOR = 0.5f; // this is 100x relative to interplanetary distances
    float SCALE_FACTOR = 0.0001f; // animation rate for one earth rotation (seconds per rotation)
    float EDAY_RATE = 10f; // rotation scale factor e.g. to animate earth: dt * 24 * // DEG_PER_EHOUR
    float DEG_PER_EHOUR = (360f / 24f / EDAY_RATE); // animation rate for one earth rotation (seconds per orbit)//  (real is EDAY_RATE * 365.26)
    float EYEAR_RATE = 1500f; // orbit scale factorfloat DEG_PER_EYEAR = (360f / EYEAR_RATE); 
```

`setupPlanets` 方法使用我们的天体数据并相应地构建新的行星。首先，让我们定义物理数据，如下所示：

```kt
    public void setupPlanets(Transform origin) {

        float[] distances = new float[] { 57.9f, 108.2f, 149.6f, 227.9f, 778.3f, 1427f, 2871f, 4497f, 5913f };
        float[] fudged_distances = new float[] { 57.9f, 108.2f, 149.6f, 227.9f, 400f, 500f, 600f, 700f, 800f };

        float[] radii = new float[] { 2440f, 6052f, 6371f, 3390f, 69911f, 58232f, 25362f, 24622f, 1186f };

        float[] rotations = new float[] { 1408.8f * 0.05f, 5832f * 0.01f, 24f, 24.6f, 9.84f, 10.2f, 17.9f, 19.1f, 6.39f };

        float[] orbits = new float[] { 0.24f, 0.615f, 1.0f, 2.379f, 11.862f, 29.456f, 84.07f, 164.81f, 247.7f };
```

`distances` 数组包含每个行星距离太阳的距离，单位为百万公里。这实际上非常巨大，尤其是对于那些非常遥远且相对于其他行星不太可见的外行星。为了使事情更有趣，我们将调整这些行星（从木星到冥王星）的距离，因此我们将使用的值在 `fudged_distances` 数组中。

`radii` 数组包含每个行星的实际大小（以公里为单位）。

`rotations` 数组包含地球小时的长度。由于水星和金星的自转速度相对于地球非常快，我们将通过任意比例因子人为地减慢它们的速度。

`orbits` 数组包含每个行星的年份长度和绕太阳一周所需的时间。

现在，让我们为每个行星的材质设置纹理 ID：

```kt
        int[] texIds = new int[]{
                R.drawable.mercury_tex,
                R.drawable.venus_tex,
                R.drawable.earth_tex,
                R.drawable.mars_tex,
                R.drawable.jupiter_tex,
                R.drawable.saturn_tex,
                R.drawable.uranus_tex,
                R.drawable.neptune_tex,
                R.drawable.pluto_tex
        };
```

现在初始化 `planets` 数组，为每个创建一个新的 `Planet` 对象：

```kt
        planets = new Planet[distances.length + 1];
        for(int i = 0; i < distances.length; i++){
            planets[i] = new Planet(
                    fudged_distances[i] * DISTANCE_FACTOR,
                    radii[i] * SCALE_FACTOR,
                    rotations[i] * DEG_PER_EHOUR,
                    orbits[i] * DEG_PER_EYEAR * fudged_distances[i]/distances[i],
                    texIds[i],
                    origin);
        }
```

虽然我们对一些行星的实际距离进行了调整，使它们更接近内太阳系，但我们还将所有距离乘以 `DISTANCE_FACTOR` 标量，主要是为了不使我们的浮点精度计算失真。我们通过不同的 `SCALE_FACTOR` 变量来缩放所有行星的大小，使它们相对于实际尺寸更大（0.0001 实际上是 100 的因子，因为半径是以公里计算的，而距离是以百万公里计算的）。

旋转动画速率是行星的实际日长度，乘以我们希望在 VR 中动画化一天的速度。我们默认为每个地球日 10 秒。

最后，行星轨道动画有自己的比例因子。我们加快了大约 2 倍。您还可以调整距离调整因子的轨道速率（例如，冥王星绕太阳运行一次需要 247 个地球年，但我们已经将其移动得更接近，因此它需要减速）。

然后，我们添加了地球的月球。我们在这里也使用了一些艺术许可，调整了距离和半径，并加快了其轨道速率，使其在 VR 中观看更有吸引力。

```kt
        // Create the moon
        planets[distances.length] = new Planet(7.5f, 0.5f, 0, - 0.516f, R.drawable.moon_tex, planets[2].getTransform());}
```

让我们再看看另一个方法：`goToPlanet`。将摄像机方便地定位在特定行星附近。由于行星位于数据驱动的位置并且将在轨道上移动，最好将摄像机设置为行星变换的子级。这是我们将轨道变换与行星变换分开的原因之一。我们不希望摄像机随着行星旋转而旋转——您可能会晕！这是实现方式：

```kt
    void goToPlanet(int index){
        RenderBox.mainCamera.getTransform().setParent( planets[index].getOrbitransform(), false);
        RenderBox.mainCamera.getTransform().setLocalPosition( planets[index].distance, planets[index].radius * 1.5f, planets[index].radius * 2f);
    }
```

### 注意

请注意，我们最终在代码中使用的比例和距离值是从而不是实际的天体测量中得出的。要体验真正有教育价值的太阳系 VR 体验，请查看《太空巨人》（[`www.titansofspacevr.com/`](http://www.titansofspacevr.com/)）。

## 摄像机的行星视图

`gotoPlanet` 函数被调用时带有一个行星索引（例如，地球是 2），这样我们就可以将摄像机定位在指定行星附近。`Camera` 组件被作为行星的 `orbitTransform` 变量的父级，以获取行星当前的轨道旋转。然后，它被定位为距离太阳的行星距离，然后相对于行星的大小进行偏移。

在 `MainActivity` 类的设置方法中，我们已经设置了太阳和地球。我们将用一个 `setupPlanets` 辅助方法替换地球球体：

```kt
    public void setup() { 
        //Sun ...

        // Planets
 setupPlanets(origin);

 // Start looking at Earth 
 goToPlanet(2);
    }
```

如果现在构建并运行项目，您将看到地球、太阳，可能还有一些行星。但直到它们在它们的轨道上运动时，它们才会活跃起来。

## 动画天体

现在我们已经实例化了所有行星，我们可以对它们的轨道和轴旋转进行动画处理。只需在 `MainAcitvity` 类的 `preDraw` 方法中更新它们的变换即可：

```kt
    @Override
    public void preDraw() {
        float dt = Time.getDeltaTime();
        for(int i = 0; i < planets.length; i++){
            planets[i].preDraw(dt);
        }
    }
```

跑吧！哦，哇！我感觉自己像个神。嗯，不完全是，因为外面很黑。我们需要星星！

# 星空天穹

如果宇宙只是一个巨大的球，我们就在其中，那将是我们要想象的实现一个星空球形背景。

在计算机图形学中，您可以创建背景，使场景看起来比实际更大。您可以使用球形纹理或天穹，就像我们将在这里使用的一样。（许多游戏引擎中的常见替代品是由立方体的六个内部面构成的立方体天空盒。）

在我们提供的纹理集中，有`milky_way_tex.png`。如果`res/drawable/`目录中没有该文件，请将其拖入其中。

现在，我们可以将星空天穹添加到我们的场景中。将以下代码添加到`MainActivity.setup()`中：

```kt
        //Stars in the sky
        Transform stars = new Transform()
                .setParent(RenderBox.mainCamera.transform, false)
                .setLocalScale(Camera.Z_FAR * 0.99f, Camera.Z_FAR * 0.99f, Camera.Z_FAR * 0.99f)
                .addComponent(new Sphere(R.drawable.milky_way_tex, false));
```

这看起来更加神圣。

![星空天穹](img/B05144_06_09.jpg)

您可能想知道 0.99 因子是什么意思。不同的 GPU 对浮点数的处理方式不同。虽然有些可能以某种方式在绘制距离上渲染顶点，但其他可能会因为浮点精度而在几何图形“边缘”出现渲染故障。在这种情况下，我们只是通过一个任意小的因子将天空盒拉向相机。在 VR 中，天空盒尽可能远是非常重要的，这样它就不会因视差而被绘制。天空盒在左眼和右眼的确切位置相同，这就是让你的大脑误以为它是无限远的技巧。您可能会发现您需要调整这个因子以避免天空盒中的空洞。

# 微调地球

如果你是一个太空迷，你可能会想到我们可以对我们的地球模型做一些事情。首先，我们应该添加夜景纹理。（火星和其他行星不需要，因为它们的城市在夜晚关闭了所有的灯光。）此外，地球在其轴上略微倾斜。我们可以修复这个问题。

## 夜晚纹理

首先，让我们添加夜景纹理。为此，让我们将`Earth` Java 类作为`Planet`的子类。右键单击您的 Java`solarsystem`文件夹，选择**新建** | **Java 类**，并将其命名为`Earth`。然后，开始定义它如下：

```kt
public class Earth extends Planet {

    public Earth(float distance, float radius, float rotation, float orbit, int texId, int nightTexId, Transform origin) {
        super(distance, radius, rotation, orbit, origin);
        transform.addComponent(new Sphere(texId, nightTexId));
    }
}
```

这要求我们向`Planet`类添加一个新的构造函数，省略`texId`，因为地球构造函数创建了新的`Sphere`组件，这次有两个纹理，`textId`和`nightTexId`。

在`Planet.java`中，添加以下代码：

```kt
    public Planet(float distance, float radius, float rotation, float orbit, Transform origin){
        setupPlanet(distance, radius, rotation, orbit, origin);
    }
```

现在，在`MainActivity`中，让我们单独创建一个地球，而不是其他行星。在`setupPlanets`中，修改循环以处理这种情况：

```kt
        for(int i = 0; i < distances.length; i++){
 if (i == 2) {
 planets[i] = new Earth(
 fudged_distances[i] * DISTANCE_FACTOR,
 radii[i] * SCALE_FACTOR,
 rotations[i] * DEG_PER_EHOUR,
 orbits[i] * DEG_PER_EYEAR * fudged_distances[i] / distances[i],
 texIds[i],
 R.drawable.earth_night_tex,
 origin);
 } else {
                planets[i] = new Planet(
```

## 轴倾斜和摆动

在所有的伟大之中，就像所有的自然和人类一样，地球并不完美。在这种情况下，我们谈论的是倾斜和摆动。地球的旋转轴并不完全垂直于轨道平面。它也在旋转时受到轻微的摆动。我们可以在我们的虚拟模型中展示这一点。

修改`Earth`类的构造函数如下：

```kt
    Transform wobble;

    public Earth(float distance, float radius, float rotation, float orbit, int texId, int nightTexId, Transform origin) {
        super(distance, radius, rotation, orbit, origin);

        wobble = new Transform()
                .setLocalPosition(distance, 0, 0)
                .setParent(orbitTransform, false);

        Transform tilt = new Transform()
                .setLocalRotation(-23.4f,0,0)
                .setParent(wobble, false);

        transform
                .setParent(tilt, false)
                .setLocalPosition(0,0,0)
                .addComponent(new Sphere(texId, nightTexId));
    }
```

现在，地球在每一帧的旋转都是针对这个摆动变换的，所以给地球自己的`preDraw`方法，如下所示：

```kt
    public void preDraw(float dt){
        orbitTransform.rotate(0, dt * orbit, 0);
        wobble.rotate(0, dt * 5, 0);
        transform.rotate(0, dt * -rotation, 0);
    }
```

# 改变相机位置

我们太阳系的最后一个特性是使它更具互动性。我的意思是所有这些行星看起来都很酷，但你实际上无法从那么远的地方看到它们。点击 Cardboard 触发器从行星跳到行星，近距离观察一下，怎么样？

幸运的是，我们已经有一个`goToPlanet`方法，我们用它来设置我们从地球的初始视图。因为`MainActivity`扩展了`CardboardActivity`，我们可以使用 Cardboard SDK 的`onCardboardTrigger`方法（参考[`developers.google.com/cardboard/android/latest/reference/com/google/vrtoolkit/cardboard/CardboardActivity.html#onCardboardTrigger()`](https://developers.google.com/cardboard/android/latest/reference/com/google/vrtoolkit/cardboard/CardboardActivity.html#onCardboardTrigger())）。

将以下代码添加到`MainActivity`中：

```kt
    int currPlanet = 2;

    public void onCardboardTrigger(){
        if (++currPlanet >= planets.length)
            currPlanet = 0;
        goToPlanet(currPlanet);
    }
```

应用程序将从地球（索引 2）附近的摄像机开始。当用户按下硬纸板触发器（或触摸屏幕）时，它将前往火星（3）。然后是木星，依此类推，然后循环回水星（0）。

# 可能的增强

您能想到对这个项目的其他增强吗？以下是一些您可以考虑并尝试实现的增强：

+   为土星添加环。（一种廉价的实现方法可能是使用具有透明度的平面。）

+   改进`goToPlanet`，使摄像机位置在各个位置之间动画变化。

+   添加控件，允许您改变透视或在太空中自由飞行。

+   添加一个俯视选项，以获得太阳系的“传统”图片。（注意在规模上的浮点精度问题。）

+   为每个其他行星添加卫星。（这可以像我们为地球的月球所做的那样实现，以其母行星为原点。）

+   表示火星和木星之间的小行星带。

+   为其他行星添加倾斜和摆动。你知道天王星是侧着自转的吗？

+   为每个行星添加文本标签，使用行星的变换，但始终面向摄像机。在没有 3D 文本对象的情况下，标签可以是预先准备的图像。

+   添加背景音乐。

+   以准确表示给定日期上每个行星的相对位置的方式改进位置精度。

# 更新 RenderBox 库

随着太阳系项目的实施和我们的代码稳定，您可能会意识到我们构建了一些不一定特定于此应用程序的代码，可以在其他项目中重用，并且应该回到`RenderBox`库。这就是我们现在要做的。

我们建议您直接在 Android Studio 中执行此操作，从此项目的层次结构视图中选择并复制到其他项目中。执行以下步骤：

1.  将太阳系的`res/raw/`目录中的所有`.shader`文件移动到`RenderBox`库的`RenderBox`模块的`res/raw/`目录中。如果您一直在跟进，将会有八个文件，分别是`day_night`、`diffuse_lighting`、`solid_color_lighting`和`unilt_tex`的顶点和片段`.shader`文件。

1.  将太阳系的`RenderBoxExt`模块文件夹中的所有`Component`和`Material`的`.java`文件移动到`RenderBox`库的`RenderBox`模块中的相应文件夹中。删除源代码中对`MainActivity`的所有无效引用。

1.  在太阳系项目中，我们在`MainActivity`中实现了一个名为`loadTexture`的方法。它应该属于`RenderBox`库。找到太阳系的`MainActivity.java`文件中`loadTexture`的声明，并剪切代码。然后，打开`RenderBox`库中的`RenderObject.java`文件，并将定义粘贴到`RenderObject`类中。

1.  在`RenderBox`库中，用`RenderObject.loadTexture`替换（重构）所有`MainActivity.loadTexture`的实例。这些将在几个`Material` Java 文件中找到，我们在其中加载材质纹理。

1.  在`RenderBox.java`中，`reset()`方法销毁任何材料的句柄。添加我们刚刚引入的新材料的调用：

+   `DayNightMaterial.destroy()`

+   `DiffuseLightingMaterial.destroy()`

+   `SolidColorLightingMaterial.destroy()`

+   `UnlitTexMaterial.destroy()`

1.  解决任何包名称不匹配的问题，并修复任何其他编译时错误，包括删除源代码中对`solarsystem`的任何引用。

现在，您应该能够成功重建库（**构建** | **生成模块'renderbox'**），以生成更新的`renderbox[-debug].aar`库文件。

最后，太阳系项目现在可以使用新的`.aar`库。将`RenderBoxLib`项目的`renderbox/build/output`文件夹中的`renderbox[-debug].aar`文件复制到 SolarSystem 的`renderbox/`文件夹中，用新构建的文件替换同名文件。使用此版本的库构建和运行太阳系项目。

# 总结

恭喜！你在太阳系科学项目中获得了“A”！

在本章中，我们构建了一个太阳系模拟，可以在虚拟现实中使用 Cardboard VR 观看器和安卓手机查看。本项目使用并扩展了`RenderBox`库，如第五章 *RenderBox Engine*中所讨论的那样。

首先，我们向我们的技能库中添加了一个`Sphere`组件。最初，它是使用纯色光照材质渲染的。然后，我们定义了一个漫反射光照材质，并使用地球图像纹理渲染了这个球体，结果呈现出了一个地球仪。接下来，我们增强了材质以接受两个纹理，为球体的背面/“夜晚”一侧添加了额外的纹理。最后，我们创建了一个未照明的纹理材质，用于太阳。凭借太阳系行星的实际大小和与太阳的距离，我们配置了一个太阳系场景，包括九大行星、地球的月亮和太阳。我们添加了一个星空作为天空圆顶，并为它们适当的自转（日）和公转（年）进行了动画处理。我们还实现了一些交互，通过对 Cardboard 触发事件的响应，将摄像机视图从行星移动到行星。

在下一章中，我们将再次使用我们的球体，这次是为了查看您的 360 度照片库。
