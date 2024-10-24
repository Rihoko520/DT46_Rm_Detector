# DT46_Rm_Detector

## 项目简介

**DT46_Rm_Detector** 是一个用于装甲板识别的计算机视觉项目。项目由三个主要模块构成：识别模块、视频流处理模块和参数调整模块。

## 使用说明
1. **安装依赖**: 确保安装了 **OpenCV** 和其他必要的 **Python** 库。
```bash
git clone https://github.com/Rihoko520/DT46_Rm_Detector.git
pip install opencv-python numpy math
```
2. **运行程序**: 
   - 根据需要选择模式：
     - 模式 0: 视频流调试
     - 模式 1: 仅运行检测
     - 模式 2: 静态图调试
   - 运行 **cam.py** 文件，程序将根据选择的模式开始处理图像。
3. **实时调整参数**: 在运行模式0和模式2时，可以使用滑动条实时调整参数，以达到最佳识别效果。

### 1. 识别模块 (**detector**)

#### **Detector** 类

**Detector** 类是识别模块的核心，负责处理图像并检测灯条和装甲板。该类包含多种参数设置以及实现识别算法的核心逻辑。

#### 参数字典

在 **Detector** 类中，使用以下参数字典来控制识别的细节：

1. **灯条参数字典** (**light_params**):
   - **light_distance_min**: 最小灯条之间的距离。
   - **light_area_min**: 最小灯条面积，过滤掉小于此值的灯条。
   - **light_angle_min**: 灯条的最小角度，用于过滤不符合方向的灯条。
   - **light_angle_max**: 灯条的最大角度。
   - **light_angle_tol**: 灯条的角度容差，允许的角度误差范围。
   - **line_angle_tol**: 线的角度容差，影响灯条之间角度的判断。
   - **height_tol**: 灯条高度的容差，用于比较灯条的高度。
   - **width_tol**: 灯条宽度的容差，用于比较灯条的宽度。
   - **cy_tol**: 中心点的y轴容差，允许的y轴误差范围。

2. **装甲板参数字典** (**armor_params**):
   - **armor_height/width_max**: 装甲板高度与宽度的最大比值。
   - **armor_height/width_min**: 装甲板高度与宽度的最小比值。
   - **armor_area_max**: 装甲板的最大面积，限制识别出的装甲板的大小。
   - **armor_area_min**: 装甲板的最小面积，过滤小于此值的装甲板。

3. **图像参数字典** (**img_params**):
   - **resolution**: 图像的分辨率，影响图像的处理速度和质量。
   - **val**: 二值化处理中的阈值，用于将图像转换为黑白图像。

4. **颜色参数字典** (**color_params**):
   - **armor_color**: 装甲板颜色映射，定义不同颜色装甲板的**RGB**值。
   - **armor_id**: 装甲板 **ID** 映射，定义不同颜色装甲板的 **ID**。
   - **light_color**: 灯条颜色映射，定义灯条的**RGB**值。
   - **light_dot**: 灯条中心点颜色映射，定义灯条中心点的颜色。

5. **模式参数字典** (**mode_params**):
   - **display**: 显示模式，0表示不显示图像，1表示显示图像。
   - **resize_in**： 输入图像调整， 0: 不调整, 1: 调整
   - **resize_out**： 输出图像调整， 0: 不调整, 1: 调整
   - **color**: 颜色模式，0表示识别红色装甲板，1表示识别蓝色装甲板，2表示识别全部装甲板。

#### 辅助函数

1. **`darker(self, img)`**:
   - 功能: 将图像的亮度降低，返回暗化后的图像。
   - 过程: 将输入图像转换为 **HSV** 颜色空间，并将 **V** 通道的值乘以 0.5，从而降低亮度。最后再转换回 **BGR** 颜色空间。

2. **`process(self, img)`**:
   - 功能: 对输入图像进行预处理。
   - 过程:
     - 若图像需要调整大小，则使用 **self.resize** 参数进行缩放。
     - 调整图像大小并存储在 **img.resized** 中。
     - 降低图像亮度并存储在 **img.darken** 中。
     - 复制暗化后的图像到 **img.draw** 中。
     - 对暗化后的图像进行二值化处理，并将结果存储在 **img.binary** 中。

3. **`adjust(self, rect)`**:
   - 功能: 调整灯条和装甲板的旋转矩形属性。
   - 过程: 解包矩形的中心、宽高和角度，确保宽度小于高度，并标准化角度到 -90° 到 90° 之间。

4. **`project(self, polygon, axis)`**:
   - 功能: 计算多边形在指定轴上的投影。
   - 过程: 使用点积计算多边形的最小值和最大值。

5. **`is_coincide(self, a, b)`**:
   - 功能: 检查两个多边形是否重叠。
   - 过程: 遍历多边形的每条边，计算法向量并进行投影。如果两个多边形在某个方向上的投影不重叠，则返回 **False**。

6. **`is_close(self, rect1, rect2, light_params)`**:
   - 功能: 检查两个矩形是否接近。
   - 过程: 计算中心点之间的距离和角度差，判断是否在容忍范围内。返回值表示两个灯条是否可以被认为是相邻的。

7. **`is_armor(self, lights)`**:
   - 功能: 判断一组灯条是否构成一个有效的装甲板，并在满足条件时创建 **Armor** 对象。
   - 过程:
     - 验证传入的灯条数量是否满足条件，至少需要 2 个灯条。
     - 遍历灯条组合，检查每对灯条的颜色是否相同。
     - 计算灯条之间的距离，确保它们足够接近。
     - 使用 **self.is_close** 方法检查灯条的相对位置和角度，判断是否符合装甲板的特征。
     - 如果灯条组合满足条件，创建 **Armor** 对象，并将灯条的矩形属性作为参数传入。
     - 将新创建的 **Armor** 对象添加到 **self.armors** 列表中，以便后续处理。
       - **Armor** 对象的属性
         1. **color**:
          - 功能: 存储装甲板的颜色，初始化为 **None**，后续可以通过其他方法进行赋值。
          - 类型: **None** 或具体颜色值。
         2. **rect**:
          - 功能: 存储装甲板的矩形属性，包含几何信息，如中心点、宽度、高度和旋转角度。
          - 类型: 旋转矩形对象，通常由构成装甲板的灯条的矩形信息传递而来。
   - 返回值: 如果灯条组合构成有效的装甲板，返回 **True**；否则返回 **False**。

8. **`id_armor(self, armor)`**:
   - 功能: 为装甲板分配 **ID**。
   - 过程: 根据颜色和位置特征为装甲板分配唯一的 **ID**，并将信息存储在 **self.armors_dict** 中。

9. **`draw_lights(self, img_draw)`**:
   - 功能: 在图像上绘制检测到的灯条。
   - 过程: 遍历 **self.lights** 列表，使用 **cv2.drawContours** 函数绘制每个灯条的边界框。

10. **`draw_armors(self, img_draw)`**:
    - 功能: 在图像上绘制检测到的装甲板。
    - 过程: 遍历 **self.armors** 列表，绘制每个装甲板的边界框和中心点。

11. **`display(self, img)`**:
    - 功能: 显示检测结果图像。
    - 过程: 使用 **OpenCV** 的 **cv2.imshow** 函数显示处理后的图像。
    - 若图像需要调整大小，则使用 **self.resize** 参数进行缩放。

#### 主要函数

1. **`process(self, img)`**:
   - 功能: 对输入图像进行预处理，调整大小、亮度和二值化，更新 **Img** 对象的各个属性。

2. **`find_lights(self, img, img_binary, img_draw)`**:
   - 功能: 查找图像中的灯条。
   - 过程:
     - 使用 **cv2.findContours** 查找轮廓。
     - 根据灯条的参数（如最小面积、角度等）过滤出符合条件的灯条，通过 **self.adjust** 调整每个灯条的矩形属性。
     - 对过滤后的灯条进行 **is_coincide** 重叠检测，确保每个灯条是独立的。
     - 根据灯条的颜色和位置创建 **Light** 对象，并添加到 **self.lights** 列表中。
      - **Light** 对象的属性
      1. **color**:
        - 功能: 指示灯条的颜色，通常依据颜色参数字典中的定义。
        - 类型: 0 --> 红色，1 --> 蓝色。
      2. **rect**:
        - 功能: 存储灯条的旋转矩形属性，包含角度和尺寸信息。
        - 类型: 一个对象，通常是一个包含中心点、宽度、高度和旋转角度的矩形属性。 
     - 调用 **self.draw_lights(img_draw)** 绘制灯条结果。

3. **`find_armor(self, img_draw)`**:
   - 功能: 查找图像中的装甲板。
   - 过程:
    - 遍历 **self.lights** 列表，调用 **self.is_armor** 检查灯条组合。
    - 如果发现符合条件的组合，使用 **self.id_armor** 方法为装甲板分配 **ID**，并将其存储在 **self.armors_dict** 字典中。
    - 调用 **self.draw_armors(img_draw)** 绘制装甲板结果。

#### 检测函数：**`Detector.detect(self, frame)`**

**detect** 函数是 **Detector** 类中的主要方法，负责整个检测流程。其大致流程如下：

1. **创建 Img 对象**:
   - 首先将输入的 **frame** 转换为 **Img** 对象。**Img** 对象用于存储图像的不同处理状态，包含以下属性：
     - **raw**: 原始图像，保存输入的 **frame**。
     - **resized**: 调整后的图像，保存经过尺寸调整的图像。
     - **draw**: 绘制后的图像，通常用于显示检测结果的图像。
     - **darken**: 暗化后的图像，保存经过亮度调整处理后的图像。
     - **binary**: 二值化后的图像，保存经过阈值处理后的黑白图像。

2. **图像处理**:
   - 调用 **self.process(img)** 方法，对图像进行预处理。此方法会更新 **Img** 对象的属性：
     - **img.resized** 被设置为调整后的图像，使用 **cv2.resize** 方法根据 **img_params["resolution"]** 进行调整。
      - 若无需进行调整，即 **img_params["resize"]** 为 **0**，则**img.resized** 为 **img.raw**。
     - **img.darken** 被设置为暗化后的图像，通过亮度降低处理调整图像的亮度。
     - **img.draw** 被设置为 **img.darken** 的副本，用于后续的绘制操作。
     - **img.binary** 通过 **cv2.threshold** 方法进行二值化处理，生成黑白图像，方便后续的轮廓检测。

3. **查找灯条**:
   - 使用 **self.find_lights(img.darken, img.binary, img.draw)** 方法查找图像中的灯条。该方法会根据灯条的参数过滤出符合条件的灯条，并将其存储在 **self.lights** 列表中。灯条的检测依赖于 **img.binary** 中的轮廓信息。

4. **查找装甲板**:
   - 通过调用 **self.find_armor(img.draw)** 方法，识别出装甲板。该方法会根据灯条的位置信息，组合灯条并判断是否为装甲板。检测到的装甲板将被存储在 **self.armors** 列表中。

5. **显示检测结果**:
   - 如果显示模式为 1，调用 **self.display(img)** 方法显示检测后的图像和二值化图像。此时，**img.draw** 中的内容将展示检测到的灯条和装甲板。
![装甲板检测示例](/photo/example.jpg)
    
6. **打印装甲板信息**:
   - 最后，打印出装甲板的信息字典 **self.armors_dict**，包含装甲板的 **ID**、中心位置和尺寸信息。
```bash
 {'443': {'armor_id': (128, 0, 128), 'height': 101, 'center': [443, 364]}, '264': {'armor_id': (255, 255, 0), 'height': 31, 'center': [264, 241]}, '366': {'armor_id': (255, 255, 0), 'height': 35, 'center': [366, 229]}}
```
### 2. 视频流处理模块 (**cam**)

- **功能**: 该模块用于处理来自摄像头的视频流，或读取指定的静态图像文件。根据用户的选择，实时识别装甲板，并在图像上绘制检测结果，以便于观察和分析。

- **主要功能**:
  - **支持多种输入源**: 用户可以选择使用摄像头实时捕捉视频流，或者从本地视频文件中读取图像进行处理。
  - **实时检测与可视化**: 模块会实时检测视频流或图像中的装甲板，并在图像上绘制检测到的结果，包括灯条、装甲板的轮廓及其颜色等信息。
  - **动态参数调整**: 提供滑动条界面，用户可以实时调整灯条和装甲板的检测参数，以优化检测效果。

#### 使用方法

1. **参数设置**:
  - 根据需求修改 **mode_params** 字典中的参数，以选择所需的检测模式和颜色识别。
  - **mode_params** 的可修改值
    1. **display**:
      - **类型**: 整数
      - **可修改值**:
        - `0`: 不显示图像
        - `1`: 显示图像（默认值）
    2. **resize_in**:
      - **类型**: 整数
      - **可修改值**:
        - `0`: 输入图像不调整
        - `1`: 输入图像调整（默认值）
    3. **resize_out**:
      - **类型**: 整数
      - **可修改值**:
        - `0`: 输出图像不调整（默认值）
        - `1`: 输出图像调整
    4. **color**:
      - **类型**: 整数
      - **可修改值**:
        - `0`: 识别红色装甲板
        - `1`: 识别蓝色装甲板
        - `2`: 识别全部装甲板（默认值）
  - 可以通过修改 **url** 和 **image_path** 变量来指定视频文件路径或静态图像路径。

2. **运行模块**:
   - 运行 **cam.py** 文件，模块将自动获取第一个可用的摄像头索引，或使用指定的视频文件进行处理。
   - 根据设置的 **mode** 值选择不同的运行模式:
     - **模式 0**: 视频流调试，实时捕捉视频并进行检测，带有滑动条调整参数。
     - **模式 1**: 仅运行检测，使用摄像头捕捉图像并进行检测。
     - **模式 2**: 静态图调试，读取指定的静态图像文件并进行检测。

3. **交互操作**:
   - 在运行过程中，用户可以通过滑动条动态调整参数，以优化检测结果。
   - 按下 **q** 键可以退出当前模式，关闭程序。

### 3. 参数调整模块 (**adjust**)

- **功能**: 提供滑动条界面，允许用户实时调整识别算法的参数，以优化检测效果。用户可以根据不同的环境和需求，动态修改各项参数。
- **主要功能**:
  - 创建滑动条界面，调整灯条和装甲板的相关参数。
  - 包含多种参数调整功能，如最小灯条距离、面积、角度容差等。
- 可调参数详细说明
  1. **灯条参数** (**light_params**):
    - **`light_distance_min`**: 
      - 类型: 整数
      - 功能: 最小灯条之间的距离，过滤过于接近的灯条。
    - **`light_area_min`**: 
      - 类型: 整数
      - 功能: 最小灯条面积，过滤小物体和噪声。
    - **`light_angle_tol`**: 
      - 类型: 整数
      - 功能: 灯条角度容差，允许一定的角度变化。
    - **`line_angle_tol`**: 
      - 类型: 整数
      - 功能: 线的角度容差。
    - **`height_tol`**: 
      - 类型: 整数
      - 功能: 灯条高度容差，允许高度变化。
    - **`width_tol`**: 
      - 类型: 整数
      - 功能: 灯条宽度容差，允许宽度变化。
    - **`cy_tol`**: 
      - 类型: 整数
      - 功能: 灯条中心点的 Y 轴容差。

  2. **装甲板参数** (**armor_params**):
    - **`armor_height/width_max`**: 
      - 类型: 浮点数
      - 功能: 装甲板高度与宽度的最大比值。
    - **`armor_area_max`**: 
      - 类型: 整数
      - 功能: 装甲板的最大面积。
    - **`armor_area_min`**: 
      - 类型: 整数
      - 功能: 装甲板的最小面积。

  3. **图像参数** (**img_params**):
    - **`val`**: 
      - 类型: 整数
      - 功能: 处理图像时的参数值，例如阈值。

## 贡献

欢迎任何形式的贡献，如报告问题、提交补丁或改善文档等。
