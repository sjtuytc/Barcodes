# Barcode Evaluation

一维码打印质量评估是对条码质量进行定量化评级的过程，本方案主要基于ISO/IEC 15416（国际标准化组织制定的条码刻印验证规格，下称ISO文件）制定完成。本方案参考了一个非官方实现[github.com/...](https://github.com/francovia/Barcodes)。

## **测量方法综述**

本文档中定义的测量方法旨在最大限度地提高各种基材上条码符号的反射率以及条码和间距宽度测量值的一致性。该方法还旨在适应条码扫描硬件中遇到的不同条件。

测量应使用定义的光源（例如单光波长）和应用规范定义的或根据ISO文件5.2.1 和 5.2.2 确定的尺寸的测量孔径进行。圆形孔径由其直径根据表 1 定义。应用规范还可以定义其他孔径或形状。

抽样方法应基于被测批次或批次内统计上有效的样本量。在质量控制检查之前，应确定可接受的最低等级。在正式质量保证程序或双边协议中没有定义抽样计划的情况下，合适的计划可能基于 ISO 2859-1 中的建议。

## **指标评分策略**

为了完成质量评定工作，我们需要计算以下几个指标：

1. decode: 能否解码
2. symbol contrast (**SC**)： **Rmax-Rmin**
3. minimum reflectance (**R****min**)：最小反射率
4. minimum edge contrast (**ECmin**)：EC的最小值，EC的求法为EC=**Rs**-**Rb**，就是空白区域的反射率减去黑色区域的反射率。
5. modulation (**MOD**)：**ECmin**/**SC**
6. defects：缺陷是在元素和静区中发现的不规则性，并根据元素反射率的不均匀性来衡量。单个元件或静区内的元件反射率不均匀性是最高峰反射率与最低谷反射率之间的差异。 当元素由单个峰或谷组成时，其反射不均匀性为零。 在扫描反射分布中发现的元素反射不均匀性的最高值是最大元素反射不均匀性。 缺陷测量表示为最大元素反射不均匀性 (ERNmax) 与符号对比度的比率。
7. decodability (**V**): 条形码符号的可解码性是衡量其生成与适当参考解码算法相关的准确性的指标。一般较高可解码性的条码会更容易被解码。管理每个条码符号的标称尺寸的规则在特定的符号规范中给出。参考解码算法通过定义一个或多个参考阈值来为打印和读取过程中的错误留出合理的余量。扫描反射曲线的可解码性是打印过程中未消耗的可用余量的一部分，因此可用于扫描过程。在计算扫描反射率分布的可解码性值 V 时，应考虑相关符号规范中参考解码算法要求的测量值。在以下段落中，术语“测量”应指代单个元素宽度，在符号体系中直接在参考解码算法（例如“代码 39”）中使用这些宽度，或两个或更多相邻元素，在符号体系中使用边缘到相似边缘测量进行解码（例如“Code 128”）。

可解码性值的计算参考以下内容：

a) 特定类型测量的平均实现宽度（在下面的公式中称为 A）[例如窄元素，或条码加空间的组合，名义上总共 2（或 3，或 4 ...）个模块]；

​           b) 适用于与 A 相同类型测量的参考阈值（在下面的公式中称为 RT）；

​          c) 实际测量值在参考阈值方向上与A的最大偏差（在下面的公式中称为 M）。

V计算公式的一般形式如下：

​            ![img](https://wdcdn.qpic.cn/MTY4ODg1MzEzMzAyMDk1MA_221385_cVp1CPDEoViLeUbM_1637912780?sign=1637914822-1930035687-0-0518edfb1b56532fe484bd9345fbdb2f)            

其中(RT - M) 是克服印刷误差未使用的剩余边距；(RT - A) 是基于元素的理想测量值的总理论余量。

文中提到最好按照以上顺序完成计算，在下一章我们将讨论如何用算法实现上述指标的计算。当指标计算好以后，按照下面的表格换算成分数。

| **Grade** | **Decode** | **Rmin**      | **SC** | **ECmin** | **MOD** | **Defects** | **V** |      |
| --------- | ---------- | ------------- | ------ | --------- | ------- | ----------- | ----- | ---- |
| 4         | 能解       | ≤0.5**Rmax**  | ≥70 %  | ≥15%      | ≥0.70   | ≤0.15       | ≥0.62 |      |
| 3         | -          | -             | ≥55 %  | -         | ≥0.60   | ≤0.20       | ≥0.50 |      |
| 2         | -          | -             | ≥40 %  | -         | ≥0.50   | ≤0.25       | ≥0.37 |      |
| 1         | -          | -             | ≥20 %  | -         | ≥0.40   | ≤0.30       | ≥0.25 |      |
| 0         | 不能解码   | >0.5 **Rmax** | <20 %  | <15%      | <0.40   | >0.30       | <0.25 |      |

注意打分并不是间断的，是可以进行插值的。比如SC=0.52，对应评分为2.8。

## **测量反射率时的细化要求**

**反射率标准**：测量的反射率值应通过校准和参考公认的国家标准实验室的方式以百分比表示，其中 100% 应对应于硫酸钡或氧化镁参考样品的反射率。

**光源标准**：应在应用规范中指定用于测量的光源，以适应预期的扫描环境。 当应用规范中未指定光源时，应使用与扫描过程中预期使用的光源最接近的光源进行测量。 光源可以包括窄带或宽带照明。 有关选择光源的指南，请参阅ISO标准文件附录 E。

**孔径标准**：测量孔径的标称直径应由用户应用规范指定，以适应预期的扫描环境。 当应用规范中未规定测量孔径时，应使用ISO标准文件表 1 作为指导。 在遇到一系列 X 尺寸的应用中，所有测量都应使用适合所遇到的最小 X 尺寸的孔径。在没有定义的 X 尺寸的情况下，Z 尺寸将被替代。

**反射率测量的光学方法**：反射率测量的参考光学几何结构应包括以下内容：

a) 一个入射光源，它在样品区域与表面的垂线成 45° 的角度是均匀的，并且在一个包含光源的平面上，该平面应既垂直于表面又平行于条；

b) 一个光收集装置，其轴线垂直于表面。

从表面的圆形样品区域反射的光应收集在一个锥体内； 其顶点夹角为15°，以垂直于表面的角度为中心，通过圆形测量孔，1:1放大倍数下其直径应等于试样区域的直径。

所有测量扫描路径所在的区域应包含在与符号条的高度垂直的两条线之间。下线应位于条的平均下边缘上方的距离处符号的图案，而上线应位于符号的条形图案的平均上边缘下方的相同距离处。 该距离应等于平均钢筋高度或测量孔径直径的 10%，以较大者为准。 检查带应延伸至符号的整个宽度，包括静区。

**测量条数要求**：为了提供条形高度不同位置处符号特性变化的影响，应在符号的整个宽度上执行多次扫描，包括具有适当测量孔径的静区和光源 定义的标称波长。 这些扫描应在检查带的高度上大致等距。 每个符号的最小扫描次数通常应为 10 或检查带的高度除以测量孔径直径，以较低者为准。

**反射曲线的要求**：条码符号质量评估应基于扫描反射曲线的分析。 扫描反射率分布图是反射率与跨越符号的线性距离的关系图。 如果扫描速度不是恒定的，测量设备绘制反射率与时间的关系图应准备好补偿加速或减速的影响。 如果绘图不是连续的模拟轮廓，则测量间隔应该足够小，以确保不会丢失重要的细节并且尺寸精度足够。

## **算法设计方案**

我已经开始创建 10 条扫描平行线，将每条线的像素值存储到一个向量中。在一个for循环中，我们可以按照定义轻易算出：最小反射率**R****min**和符号对比度**SC**。

​            ![img](https://wdcdn.qpic.cn/MTY4ODg1MzEzMzAyMDk1MA_46626_2_8t4NsKPDi9AcqS_1637913814?sign=1637914822-999632821-0-484abbceb76b91a696324f6df944b977)            

然后在另一个循环中，为了衡量线段的对比度，我定义了等于符号对比度 SC/2 一半的中线。之后，我计算了Frei-chen 算子（线段检测的算子），以评估沿扫描的前向和后向像素，以减少噪声。 因此，当我接近平均线时，有两个阈值，一个用于边缘的上部，另一个用于边缘的下部，我计算了边缘对比度(**ECmin**)并更新了所建立的边缘数。 

​            ![img](https://wdcdn.qpic.cn/MTY4ODg1MzEzMzAyMDk1MA_904864_BG7Wjv7mI5nE9u3O_1637914116?sign=1637914822-226770931-0-6fe342809f5824d3586bf4fea109f82b)            

我采用了类似的程序来查找缺陷。 最后，如果我建立了一条边，我可以将参数与所需的参数进行比较，并将参数存储到参数的向量中。



## 安装依赖

OpenCV

## 原作者的描述

I've started to create 10 parallel lines, by dividing the number of rows of my ROI, then I've stored pixels values for each line into an scan_profile vector, with a for loop.
I've founded the first two parameters: Minimum Reflectance and Symbol Contrast.
Then in another loop, I've defined the median line equal than the half of the Symbol Contrast SC/2. After that, I've computed frei-chen operator, to evaluate forward and backward pixels along the scan, in order to reduce noise. So, when I am in proximity of mean line, with two thresholds, one for upper part and the other for lower part of the edge, I've computed edge contrast and updated the number of edges founded. A similar procedure I've adopted to find defects. 
At the end, if I've founded an edge I could compared parameters with the desired ones and I've stored parameters into parameter's vector.

##### Overall Symbol Grade

By using comparision between all parameters, for all parallel scans, I've computed the Overall Symbol Grade, then the mean value.
In conclusion, I've printed all parameters, founded previously, to a excel file.

##  

Several images of linear barcodes are used as dataset to:
- Find the ROI (Region Of Interest) with the Barcode and extract some characteristics
- Estimate quality parameters of Barcode according to the specific ISO/IEC 15416

In particolar, the following quality parameters are computed:
1. Symbol Contrast
2. Reflectance
3. Min edge contrast
4. Modulation
5. Defects
6. Overall Symbol Grade

## ROI

### Binarization
In order to produce a correct binarization of the image, I've used Otsu's Threshold, then I've applied a morphology operator composed by a vertical line dimension kernel, in opening configuration, to eliminate other objects not related to the barcode.
After, a new morphology operation, a dilation by a small rectangular kernel (3 rows, 1 col), several times, to connect all the regions of the barcode and creating a big one rectangle.

### Labeling
According to the Flood-fill approach, I can find all the labels. Furthermore, to find exactly the barcode into the image, I've used all object's areas and I've taken the biggest one. Then I've eliminated all the other labels, with areas less than the barcode.

### Position and Orientation
In this section, I've used the function RotatedRect that it gives me back the minimum enclosing rectangle(MER) and In order to find the minimum and maximum position of the barcode, I've checked all the corners of the MER.
While, to find angle, I've computed the difference between the right and left corner along the x direction, i.e. the same side of the rectangle.
At the end, by using warpAffine function, I've created a rotation matrix with the angle founded before, and I've rotated and cutted the image.

### X-dimension
To get a precise estimation of my barcode, I've just cutted my original image around the region of my barcode, with the previous steps. Then, I've applied again the binarization with the Otsu's threshold and the morphology open operation with a vertical line element.
After, I've created for each bar a bounded rect that contains the bar, after the estimation of the contours.
Then to find the X dimensions requested of smallest bar of barcode, I've evaluated all the bars in a for loop and I've extracted the height, width and area.
At the end, I've stored all the parameters in my parameter's vector and I've cutted again the image in the specific ROI.



