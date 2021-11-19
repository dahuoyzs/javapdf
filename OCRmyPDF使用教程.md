# OCRmyPDF  使用教程



学习这个项目的主要目的无非就是想对 扫描件PDF 进行OCR识别。并生产一个可以编辑的PDF  一些图片如何可以保存，当然更好。一些页面不规整，如果能调整一些当然更好。如果要转换的PDF数量比较多，当然可以并行处理会更快一些。OCR时当然也要注意一下语言的区分。如果可以把生产的文件提交变小，那更是爽。

经过上方的需求 读完本文章后 可以把组合一下命令，放置文章开头。方便复制。

## 便捷命令

单文件

```shell

#                中文                txt                   pdf
wsl ocrmypdf -l chi_sim --sidecar output.txt input.pdf output.pdf

#                中文       矫正      清理      修正           输入     输出
wsl ocrmypdf -l chi_sim --deskew --clean --rotate-pages input.pdf output.pdf

#               强制         中文       矫正      清理      修正                  输入     输出
wsl ocrmypdf --force-ocr -l chi_sim --deskew --clean --rotate-pages input.pdf output.pdf
```

多文件

```shell
#              CPU最高占用80%   3个Job并行   中文              矫正     清理       修正               输入     
wsl parallel --tag --load 80% -j 3 ocrmypdf  -l chi_sim --deskew --clean --rotate-pages '{}' 'output/{}' ::: *.pdf

#           CPU最高占用80%   3个Job并行            强制      中文         矫正    清理         修正             输入     
wsl parallel --tag --load 80% -j 3 ocrmypdf  --force-ocr -l chi_sim --deskew --clean --rotate-pages '{}' 'output/{}' ::: *.pdf
```



### parallel 常见参数

- --eta：显示任务完成的预计剩余时间。
- -j 2 或 -jobs 2：同时运行的命令数，在本例中设置为2。
- --load 80%：最大cpu负载。在上面的命令中，我们指定最多可以运行80％的CPU。
- --noswap：如果服务器处于大量内存负载下，则不会启动新作业，以便在存储新信息之前从内存中删除信息。
- 后面的命令使用重定向和管道符时：">>" ">" "|" 需要加上双引号;

### 前提知识

PDF全称Portable Document Format，译为可移植文档格式，是一种电子文件格式。这种文件格式与操作系统平台无关，也就是说，PDF文件不管是在Windows，Unix还是在苹果公司的Mac OS操作系统中都 是通用的。这一性能使它成为在Internet上进行电子文档发行和数字化信息传播的理想文档格式。

PDF/A是PDF格式的一种，是为长期保存文件而设计的。基本上就是屏蔽了一些不适合的功能，如Javascript，音频、视频等等。　



> 正文开始



## 官方教程

> [OCRmyPDF documentation](https://ocrmypdf.readthedocs.io/en/latest/index.html)



## 安装

这些平台具有一行程序安装:

| Debian, Ubuntu              | `apt install ocrmypdf`      |
| --------------------------- | --------------------------- |
| Windows Subsystem for Linux | `apt install ocrmypdf`      |
| Fedora                      | `dnf install ocrmypdf`      |
| macOS                       | `brew install ocrmypdf`     |
| LinuxBrew                   | `brew install ocrmypdf`     |
| FreeBSD                     | `pkg install py38-ocrmypdf` |
| Conda (WSL, macOS, Linux)   | `conda install ocrmypdf`    |



## Windows 系统安装

比较简单的方法是先安装Ubuntu子系统 WSL   

WSL 安装并升级完成后`apt install ocrmypdf` 即可  然后再安装 语言包  `apt-get install tesseract-ocr-chi-sim`

> 注意使用语言包是   在参数上加  -l chi_sim    为中文   注意 - 替换为下划线  



## 使用

添加OCR层并转换为PDF/A

```
ocrmypdf -l chi_sim input.pdf output.pdf
```

添加OCR层并输出标准PDF 

```
ocrmypdf -l chi_sim --output-type pdf input.pdf output.pdf
```

创建PDF/ a与所有颜色和灰度图像转换为JPEG

```
ocrmypdf -l chi_sim --output-type pdfa --pdfa-image-compression jpeg input.pdf output.pdf
```



## 页面旋转

OCR将尝试自动纠正每一页的旋转。这可以帮助修复混合了横向和纵向页面的扫描工作。

```
ocrmypdf --rotate-pages myfile.pdf myfile.pdf
```

您可以增加(减少)参数 `--rotate-pages-threshold` 以使页面旋转更积极(更不积极)。阈值数是OCR引擎对文档图像应该更改与保持不变的置信度的比率。默认值相当保守;在某些文件中，它可能根本不会尝试旋转，除非它非常确信当前的旋转是错误的。较低的`2.0`值将产生更多的旋转和更多的假阳性。使用`-v1`运行以查看每个页面的置信级别，以查看是否有更好的文件值。

如果页面“稍微偏离水平方向”，就像一幅歪斜的图片，那么你就需要`--deskew`。`--rotate-pages`是在主角错误时使用的。



## 除英语外的OCR语言

OCRmyPDF假定文档是英文的，除非另有说明。如果使用错误的语言，OCR的质量可能会很差。

```
ocrmypdf -l fra LeParisien.pdf LeParisien.pdf
ocrmypdf -l eng+fra Bilingual-English-French.pdf Bilingual-English-French.pdf
```

必须为所有指定的语言安装语言包。请参见安装其他语言包。

不幸的是，Tesseract OCR引擎没有能力检测未知的语言。



## 生成PDF和包含OCR文本的文本文件

这将生成一个名为“output.pdf”的文件和一个名为“output.txt”的文本文件。

```
ocrmypdf -l chi_sim --sidecar output.txt input.pdf output.pdf
```



## OCR图像，而不是pdf

### 使用 Tesseract

如果你从图像开始，你可以直接使用Tesseract将图像转换为pdf

```
tesseract my-image.jpg output-prefix pdf
```

```
# 当有多个图像时
tesseract text-file-containing-list-of-image-filenames.txt output-prefix pdf
```

Tesseract的PDF输出相当不错——在某些情况下，OCRmyPDF在内部使用它。然而，OCRmyPDF有许多在Tesseract中不可用的特性，如图像处理、元数据控制和PDF/A生成。



### 使用 img2pdf

您还可以使用像img2pdf这样的程序将图像转换为pdf，然后通过管道将结果运行ocrmypdf。-告诉ocrmypdf读取标准输入。

```
img2pdf my-images*.jpg | ocrmypdf - myfile.pdf
```

推荐使用img2pdf，因为它在不转码图像的情况下生成pdf非常出色。



### 使用 OCRmyPDF (仅限单个图像)

为了方便，OCRmyPDF  也可以将单个图像转换为pdf。如果图像的分辨率(每英寸点，DPI)没有设置或不正确，可以使用`--image-dpi`覆盖它。(因为1英寸是2.54厘米，1 dpi = 0.39 dpcm)。

```
ocrmypdf --image-dpi 300 image.png myfile.pdf
```

如果您有多个图像，您必须使用img2pdf将图像转换为PDF。



### 不推荐

我们警告不要使用ImageMagick或Ghostscript将图像转换为PDF，因为它们可能会转换图像或生成下采样图像，有时没有警告。



## 图片处理

如果需要，OCRmyPDF会对PDF的每个页面执行一些图像处理。对每个页面应用相同的处理。建议用户在图像处理后检查文件，因为这些命令可能会删除所需的内容，特别是从质量较差的扫描。



- `--rotate-pages` 尝试确定每个页面的正确方向，并在必要时旋转页面.
- `--remove-background` 尝试从灰度或彩色图像中检测和去除有噪声的背景。单色图像被忽略。这不应该用于包含彩色照片的文档，因为它可能会删除它们.
- `--deskew` 将纠正的页面以一个倾斜的角度扫描，通过旋转他们回到原位。歪斜的确定和校正使用[Postl的行和方差](http://www.leptonica.org/skew-measurement.html)算法来执行，正如在[Leptonica](http://www.leptonica.org/index.html)中实现的那样。
- `--clean` 使用[unpaper](https://www.flameeyes.eu/projects/unpaper)在OCR之前清理页面，但不改变最终输出。这使得OCR不太可能在背景噪音中寻找文本.
- `--clean-final` 使用unpaper在OCR之前清理页面，并将页面插入最终输出。你会想要检查每一页，以确保unpaper没有删除重要的东西.



> 在许多情况下，图像处理会将PDF页面栅格化为图像，可能会降低质量。



### 示例:OCR和正确的文档倾斜(弯曲扫描)

Deskew:

```
ocrmypdf --deskew input.pdf output.pdf
```

可以组合图像处理命令。给出选项的顺序并不重要。OCRmyPDF总是以相同的顺序应用图像处理管道的步骤(旋转、删除背景、deskew、清洁)。

```
ocrmypdf --deskew --clean --rotate-pages input.pdf output.pdf
```





## 不用OCR我的PDF

如果您设置`--tesseract-timeout 0`，那么OCRmyPDF将应用其图像处理而不执行OCR，如果您只想应用图像处理或PDF/A转换。

```
ocrmypdf --tesseract-timeout=0 --remove-background input.pdf output.pdf
```

### 优化图像而不执行OCR

您还可以优化所有图像而不执行任何OCR

```
ocrmypdf --tesseract-timeout=0 --optimize 3 --skip-text input.pdf output.pdf
```

### 仅对某些页面执行OCR

您可以要求OCRmyPDF只对某些页面应用OCR。

```
ocrmypdf --pages 2,3,13-17 input.pdf output.pdf
```

连字符表示页码的范围，逗号分隔页码。如果您喜欢使用空格，请引用所有的页码:`--pages '2, 3, 5, 7'`。

如果您的页码列表包含重复或重叠的页面，OCRmyPDF将发出警告。OCRmyPDF目前不考虑文档页码，例如使用罗马数字的书的介绍部分。它只是计算从开始到现在虚拟纸张的数量。

不管`--pages`，的参数是什么，OCRmyPDF都会优化文件中的所有页面并将其转换为PDF/A，除非禁用这些选项。在这个例子中，我们希望只对标题进行OCR，否则尽可能少地更改PDF:

```
ocrmypdf --pages 1 --output-type pdf --optimize 0 input.pdf output.pdf
```



## 重做现有的OCR

要用其他OCR软件或以前版本的OCRmyPDF和/或Tesseract在文件OCRed上重做OCR，可以使用`--redo-ocr`参数。(通常情况下，如果被要求用OCR修改文件，OCRmyPDF将退出并出现错误。)

这对于那些想要利用Tesseract 4.0的准确性改进的用户是很有帮助的，他们以前使用Tesseract和OCRmyPDF的早期版本来occred文件。

```
ocrmypdf --redo-ocr input.pdf output.pdf
```

这种方法将在不栅格化、降低质量或删除矢量内容的情况下取代OCR。如果一个文件包含纯数字文本和OCR的混合，数字文本将被忽略，OCR将被替换。因此，这种模式与图像处理选项不兼容，因为它们会改变文件的外观。

在某些情况下，无法检测或替换现有的OCR。例如，由OCRmyPDF v2.2或更早版本生成的文件在内部表示为具有可见文本，并在顶部绘制不透明图像。无法检测到这种情况。

如果`--redo-ocr`不能工作，您可以使用`--force-ocr`，它将强制对所有页面进行光栅化，可能会降低质量或丢失矢量内容。



## 提高OCR质量

[图像处理](https://ocrmypdf.readthedocs.io/en/latest/cookbook.html#image-processing)特征可以提高OCR的质量。

旋转页面和反斜有助于确保在OCR开始前页面方向正确。删除背景和/或清理页面也可以提高结果。可以指定'—过采样DPI '参数，以便在尝试OCR之前将图像重新采样到更高的分辨率;这也可以改善结果。

如果输入图像的分辨率不正确，OCR质量将受到影响(因为将检查可能的fo的像素大小范围.



## PDF优化

默认情况下，OCRmyPDF将尝试在OCR完成后对pdf中的图像执行无损优化。即使没有找到OCR文本，也会执行优化。

`--optimize N`(简写为`-O`)参数控制优化，其中` N `的范围从0到3(包括0到3)，类似于GCC编译器中的优化级别。

| Level          | Comments                                                     |
| -------------- | ------------------------------------------------------------ |
| `--optimize 0` | 禁用优化.                                                    |
| `--optimize 1` | 支持无损优化，如将图像转换为更有效的格式。还可以压缩PDF中其他未压缩的对象，使PDF中的“对象流”更有效. |
| `--optimize 2` | 所有这些，并支持有损优化和颜色量化。                         |
| `--optimize 3` | 所有这些，并支持更积极的优化和目标较低的图像质量。           |

当JBIG2编码器可用和安装`pngquant`时，优化得到改进。如果缺少其中任何一个组件，那么某些类型的图像就不能被优化。

可用的优化类型可能会随着时间的推移而扩展。默认情况下，OCRmyPDF压缩pdf中的数据流，并将低效的压缩模式更改为更现代的版本。像`qpdf`这样的程序可以用来更改编码，例如检查PDF的内部结构。

```
ocrmypdf --optimize 3 in.pdf out.pdf  # 让它小
```







# 批量处理

本文提供了有关在多个文件上运行 OCRmyPDF 或将其配置为由文件系统事件触发的服务的信息。

## 批次作业

考虑使用优秀的[GNU 并行同时](https://www.gnu.org/software/parallel/)将 OCRmyPDF 应用于多个文件。

两者都将尝试使用所有可用的处理器。为了最大限度地实现并行性，而不会使系统与流程过载，请考虑使用并行来限制同时运行两个作业。`parallel`  `ocrmypdf`  `parallel -j 2`

此命令将运行当前目录中命名的所有 ocrmypdf 所有文件，并将它们写入之前创建的文件夹。它不会搜索子方向。`*.pdf`  `output/`

该参数要求在打印邮件时将文件名作为前缀打印为前缀，以便人们能够将任何错误跟踪到生成这些错误的文件中。`--tag`

```
parallel --tag -j 2 ocrmypdf '{}' 'output/{}' ::: *.pdf
```

OCRmyPDF 在分析和收集 PDF 信息之前会自动修复 PDF。

## 目录树

这将穿过目录树，并在所有文件上运行 OCR，以制作的方式打印输出

```
find . -printf '%p' -name '*.pdf' -exec ocrmypdf '{}' '{}' \;
```

或者，用码头集装箱（将体积安装到储存 PDF 的容器中）：

```
find . -printf '%p' -name '*.pdf' -exec docker run --rm -v <host dir>:<container dir> jbarlow83/ocrmypdf '<container dir>/{}' '<container dir>/{}' \;
```

这每次只运行一个过程。此变体用于创建目录列表并并行运行，再次更新已到位的文件。`ocrmypdf``find``parallel``ocrmypdf`

```
find . -name '*.pdf' | parallel --tag -j 2 ocrmypdf '{}' '{}'
```

在 Windows 批次文件中，使用

```
for /r %%f in (*.pdf) do ocrmypdf %%f %%f
```





以上是把官网的教程做了一部分抽取，把比较关键的一些内容抽取了出来。并翻译了一下。