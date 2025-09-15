# yolosam
# 环境
yolo和sam环境友好。配置相对简单
1. 首先conda create -n yolo python=3.9 -y创建一个虚拟环境
2. git clone https://github.com/ultralytics/ultralytics.git 到你的工作目录下
3. pip install ultralytics(会自动帮你选择合适的torch版本)
4. git clone git@github.com:facebookresearch/segment-anything.git 到你的工作目录下
5. cd segment-anything; pip install -e .
6. pip install pycocotools onnxruntime onnx
7. 执行yolo_sam.ipynb的第一个cell，检查上面的环境是否安装完整

# 运行
首先你得准备好一个yolo detect训练格式的数据集，具体指南参考第三部分
1. cd ./ultralytics/ultralytics/models/yolo/detect，这里面应该有一个train.py
2. 用下面的指令启动训练：yolo task=detect mode=train model=/path/to/your/weights data=/path/to/your/custom/yolo/dataset batch=7 epochs=2000 imgsz=640 workers=8 device=0 
还有一些可选参数：save_period,pretrained,patience(控制早停)，yolo可以选择的参数很多，更深入细节请查看./ultralytics/ultralytics/cfg/default.yaml
3. 随后就会启动训练，笔者在train时，系统还自动加载了一个小模型yolo11n.pt，trian的结果会在./ultralytics/ultralytics/models/yolo/detect/runs/detect
   也就是你在哪个目录用yolo指令，就会在当前目录生成一个runs目录，无论是训练还是测试，都会默认保存结果在该目录
4. 训练结束后，weights文件保存在./ultralytics/ultralytics/models/yolo/detect/runs/detect/weights下，
5. 运行笔者提供的yolo_sam.ipynb脚本,记得修改sam yolo模型的权重地址，input_image以及pathout这四个路径最后会在你存储yolo_sam.ipynb的同路径下生成一个runs/detect的文件夹，检测和分割结果就储存在那

# 数据集
默认你有如下格式的文件夹
-custom_yolo_dataset
---shoes
-----origin
-----annotations
-----images
-----ImageSets
-----labels
---json2xml.py
---splitdataset.py
---xml2txt.py
1. 使用labelme标注，得到 x.jpg 以及对应的 x.json，将他们放在origin内，如果本身有xml格式标注请跳到4
2. 笔者提供的三个tool文件：json2xml.py(yolo训练要xml格式)，splitdataset.py(自动划分训练集、测试集)，xml2txt.py
3. 执行python json2xml.py，记得修改其中的12，13行，如果你不是jpg格式，请修改23行，其他内容不重要，可以不修改
4. 执行python splitdataset.py，修改第6，16，17，18，19行路径
5. 执行python xml2txt.py,路径涉及比较多，记得都检查一遍
6. 在./custom_yolo_dataset下编写一个shoes.yaml文件，大致如下，路径记得修改，一定用绝对路径！！！，nc和names如果你做别的目标识别，也请记得修改
   笔者会提供给shoes.yaml，你可以只修改其中三个路径

（注意没有//，只是#这个注释符在md中被占用了）下面是一个yaml的模板
   
train: /data/lmy/custom_yolo_data/shoes/train.txt

val: /data/lmy/custom_yolo_data/shoes/val.txt

test: /data/lmy/custom_yolo_data/shoes/test.txt

nc: 1

// # class names

names: ['nail']

7. 现在万事俱备，请你回到第二部分去trian

8. 如果你还不会使用labelme，请进入第四部分

# 标注
1. labelme笔者只在windows下实践过，理论在macos上也可行，如果不行的话，可以寻求其他标注工具，但生成数据集的步骤就不一定能参考第三部分了，需读者自行解决，
2. 下载 anaconda
从官网或者清华源下载并安装到本地
3. 安装labelme
先创造环境,python版本不要太高，容易报错
conda create --name labelme python=3.6 -y
激活环境后，安装必要依赖
conda install pyqt
conda install pillow
安装labelme
pip install labelme
安装成功后，在终端输入labelme,会启动程序。
4. 使用labelme 标注数据
左边栏 open dir打开待标记的图片集。
使用create polygons标点法框出目标物体的轮廓，轮廓首尾相连后会自动跳出label框，输入label名，如有需要可以输入group id，可从同类物体中区分出个体。
我们在标注时，一类物体使用同一个label，整个训练流程前后要保持一致。
完成后crtl s保存到目标文件夹，labelme会自动生成一个与原图文件名一样的json文件。
