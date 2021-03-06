# Learning Transferable Visual Models From Natural Language Supervision

从自然语言监督中学习可转换的视觉模型

## 摘要
最先进的计算机视觉系统训练预测一组固定的预定对象类别。这种受限的超级视觉形式限制了它们的通用性和可用性，因为需要额外的标记数据来指定任何其他视觉概念。直接从原始文本中学习图像是一种很有前途的选择，它利用了更广泛的监督来源。我们证明，在从互联网上收集的4亿对（图像、文本）数据集上，预测哪个字幕与哪个图像匹配的简单预训练任务是一种有效且可扩展的方法，可以从零开始学习SOTA图像表示。在预训练之后，自然语言被用来引用学习到的视觉概念（或描述新的概念），从而实现模型到下游任务的零镜头传输。我们通过在30多个不同的现有计算机视觉数据集上进行基准测试来研究这种方法的性能，这些数据集跨越了OCR、视频中的动作识别、地理定位和许多类型的细粒度对象分类等任务。该模型可以很容易地转移到大多数任务中，并且通常与完全监督的基线竞争，而不需要任何特定于数据集的训练。例如，我们在ImageNet zero shot上匹配原始ResNet-50的精度，而不需要使用128万个训练示例中的任何一个。

## 1. 介绍和激励工作
在过去几年中，直接从原始文本中学习的预培训方法彻底改变了NLP（Dai&Le，2015；Peters et al.，2018；Howard&Ruder，2018；Rad-ford et al.，2018；Devlin et al.，2018；Raffel et al.，2019）。任务不可知的目标，如自回归和掩蔽语言建模，在计算、模型容量和数据方面已经跨越了许多数量级，稳定地提高了能力。“文本到文本”作为标准化输入-输出接口的发展（McCann等人，2018；Radford等人，2019；Raffel等人，2019）使得任务无关架构能够零炮传输到下游数据集，从而无需专门的输出头或数据集特定的定制。像GPT-3这样的旗舰系统（Brown等人，2020年）现在在许多定制模型的任务中都具有竞争力，同时几乎不需要特定于数据集的训练数据。

这些结果表明，在网络规模的文本集合中，现代预训练方法的总体监督能力优于高质量的人群标记NLP数据集。然而，在计算机视觉等其他领域，在人群标记数据集（如ImageNet）上预训练模型仍然是标准做法（Deng等人，2009）。直接从网络文本中学习的可伸缩预训练方法能否在计算机视觉领域取得类似的突破？先前的工作令人鼓舞。

20多年前，Mori et al.（1999）通过训练一个模型来预测与图像配对的文本文档中的名词和形容词，探索改进基于内容的图像检索。Quattoni等人（2007年）证明，通过训练分类器预测与图像相关的字幕中的单词的权重空间中的流形学习，可以学习更有效的图像表示。Sri-vastava和Salakhutdinov（2012）通过在底层图像和文本标签特征的基础上训练多模态深层Boltzmann机器，探索了深层表征学习。Joulin等人（2016）对这一工作进行了现代化改造，并证明经过训练的CNN能够预测图像标题中的单词，从而学习有用的图像表示。他们将YFCC100M数据集（Thomee et al.，2016）中图像的标题、描述和hashtag元数据转换为一袋单词多标签分类任务，并显示预训练AlexNet（Krizhevsky et al.）。，2012年）对这些标签进行预听写，学习了类似于基于ImageNet的转移任务预培训的表达。Li et al.（2017）随后将这种方法扩展到除单个单词外的短语n-grams预测，并通过基于学习的视觉n-grams词典对目标类进行评分并预测得分最高的目标类，证明了他们的系统零镜头传输到其他图像分类数据集的能力。VirTex（Desai&Johnson，2020）、ICMLM（Bulent-Sariyildiz et al.，2020）和ConVIRT（Zhang et al.，2020）采用了最新的架构和预训练方法，最近展示了基于转换器的语言建模、掩蔽语言建模和对比对象从文本中学习图像表示的潜力。

虽然令人兴奋的概念证明，使用自然语言监督图像表示学习仍然是罕见的。这很可能是因为在com-mon基准测试中表现出来的性能远远低于其他方法。例如，Li等人（2017年）在零拍设置下，ImageNet的准确率仅为11.5%。这远低于目前技术水平88.4%的准确率（Xie等人，2020年）。它甚至低于经典计算机视觉方法50%的准确率（Deng等人，2012）。相反，范围更窄但目标明确的弱监管的使用提高了性能。Mahajan等人（2018）表明，预测Instagram图像上与ImageNet相关的标签是一项有效的预训练任务。当微调到ImageNet时，这些预先训练的模型将精确度提高了5%以上，并改善了当时的整体技术水平。Kolesnikov et al.（2019）和Dosovitskiy et al.（2020）还通过预训练模型预测噪声标记的JFT-300M数据集的类别，证明了在更广泛的转移基准集上的巨大收益。

这一行的工作代表了当前从有限数量的超级“黄金标签”学习和从几乎无限数量的原始文本学习之间的务实中间立场。然而，它并非没有压缩。这两部作品都经过精心设计，并在工艺限制下，分别监理到1000级和18291级。自然语言能够通过其普遍性来表达并监督更广泛的视觉概念。这两种方法都使用静态softmax分类器来执行预测，并且缺乏动态输出的机制。这严重削弱了它们的灵活性，限制了它们的发展“零射击”能力。

这些弱监督模型和最近直接从自然语言学习图像表示的探索之间的一个关键区别是尺度。Mahajan et al.（2018）和Kolesnikov et al.（2019）在数百万到数十亿张图像上训练了加速器年的模型，VirTex、ICMLM和ConVIRT在一到二十万张图像上训练了加速器日。在这项工作中，我们填补了这一空白，并大规模研究了在自然语言监督下训练的图像分类器的行为。借助互联网上大量的公开数据，我们创建了一个4亿对（图像、文本）的新数据集，并演示了一个简化版的ConVIRT从头开始训练，我们称之为CLIP，用于对比语言图像预训练，是学习自然语言的有效方法。我们通过训练一系列八个模型来研究CLIP的可伸缩性，这些模型跨越了几乎2个数量级的计算和观察，传输性能是计算的一个平滑可预测函数（Hestness et al.，2017；Kaplan et al.，2020）。我们发现，CLIP类似于GPT家族，在训练前学习执行一系列任务，包括OCR、地理定位、动作识别等。我们通过在30多个现有数据集上测试CLIP的零镜头传输性能来衡量这一点，并发现它可以与以前的任务特定的监督模型相竞争。我们还用线性探针表示学习分析证实了这些发现，并表明CLIP-out执行了最好的公共可用ImageNet模型，同时计算效率也更高。此外，我们还发现零镜头剪辑模型比同等精度监督的ImageNet模型更具鲁棒性，这表明任务无关模型的零镜头评估更能代表模型的性能。这些结果具有重大的政策和伦理意义，我们在第7节中对此进行了讨论。

## 2. 方法
### 2.1 自然语言监督
我们的方法的核心是从自然语言中包含的监督中学习感知。正如导言中所讨论的，这根本不是一个新概念，但是用于描述这一领域工作的术语是多种多样的，甚至看似矛盾，而且所陈述的动机也是多种多样的。Zhang et al.（2020）、Gomez et al.（2017）、Joulin et al.（2016）和Desai&Johnson（2020）都介绍了从文本与图像配对中学习视觉表示的方法，但分别将其描述为无监督、自监督、弱监督和监督。

我们强调，这项工作的共同点不是所使用的特定方法的任何细节，而是作为训练信号的自然语言的欣赏。所有这些方法都是从自然语言的超视觉学习。尽管早期的工作在使用主题模型和n-gram表示法时与自然语言的复杂性进行了斗争，但深层语境表示学习的改进表明，我们现在有了有效利用这一丰富监督来源的工具（McCann et al.，2017）。

与其他训练方法相比，从自然语言中学习有几个潜在的优势。与用于图像分类的标准众包标记相比，扩展自然语言监督要容易得多，因为它不要求注释采用经典的“机器学习兼容格式”，如规范的1/N多数票“黄金标签”。相反，研究自然语言的方法可以被动地从互联网上大量文本所包含的监督中学习。与大多数无监督或自监督学习方法相比，从自然语言学习也有一个重要的优势，即它不仅“只是”学习一种表征，而且还将这种表征与语言联系起来，从而实现灵活的零镜头转换。在下面的小节中，我们将详细介绍我们确定的具体方法。

### 2.2 创建足够大的数据集
现有工作主要使用了三个数据集，MS-COCO（Lin et al.，2014）、Visual Genome（Krishna et al.，2017）和YFCC100（Thomee et al.，2016）。虽然MS-COCO和Visual Genome是高质量的人群标记数据集，但按照现代标准，它们都很小，每个都有大约100000张训练照片。相比之下，其他计算机视觉系统的训练量高达35亿张Instagram照片（Mahajan等人，2018年）。YFCC100M（1亿张照片）是一个可能的替代方案，但每个图像的元数据都是稀疏的，质量也各不相同。许多图像使用自动生成的文件名（如2016071613957.JPG）作为“标题”或包含相机曝光设置的“说明”。经过过滤，只保留自然语言标题和/或英文描述的图片，数据集缩小了6倍，只有1500万张照片。这与ImageNet的大小大致相同。

自然语言监督的一个主要动机是互联网上公开的大量数据。由于现有的数据集不能充分反映这种可能性，仅考虑这些数据的结果将不会低估这一研究领域的潜力。为了解决这个问题，我们构建了一个新的数据集，这个数据集由4亿对（图像、文本）对组成，这些对是从互联网上各种公开来源收集的。为了尽可能广泛地涵盖一组视觉概念，我们在构建过程中搜索（图像、文本）对，其中文本包含500000个查询中的一个。我们通过每个查询包含20000个（图像、文本）对来大致平衡结果。结果数据集的总字数与用于训练GPT-2的WebText数据集相似。我们将此数据集称为WebImageText的WIT。

### 2.3 选择有效的预训练方法
最先进的计算机视觉系统使用大量的计算机。Mahajan等人（2018年）需要19年GPU来训练他们的ResNeXt101-32x48d，Xie等人（2020年）需要33年TPUv3核心年来训练他们的噪音学生效率NET-L2。当考虑到这两个系统都只被训练来预测1000个ImageNet类时，从自然语言中学习一组开放的视觉概念的任务似乎很艰巨。在我们的ef-forts过程中，我们发现训练效率是成功扩展自然语言监控的关键，我们选择了最终的预训练方法。

我们最初的方法，类似于VirTex，从零开始联合训练图像CNN和文本转换器来预测图像的标题。但是，我们在有效地扩展此方法时遇到了困难。在图2中，我们展示了一个6300万参数的transformer语言模型，它已经使用了ResNet-50图像编码器计算量的两倍，它学习识别ImageNet类的速度比一个简单得多的基线慢了三倍，该基线预测了同一文本的一包字编码。

这两种方法有一个关键的相似之处。他们试着预测每幅图片所附带的文字的确切字词。这是一个困难的任务，因为各种各样的描述，评论和相关的文本，同时出现在图像。最近在图像对比表征学习方面的研究发现，对比目标比其等效预测目标能更好地学习表征（Tian et al.，2019）。其他研究发现，尽管图像的生成模型可以学习高质量的图像表示，但它们需要比具有相同性能的对比模型多出一个数量级的计算量（Chen等人，2020a）。注意到这些发现，我们探索了训练一个系统来解决潜在的更容易的代理任务，即只预测哪个文本作为一个整体与哪个图像配对，而不是该文本的确切单词。从同一袋单词编码基线开始，我们将图2中的预测目标替换为对比目标，并观察到在将零镜头传输到ImageNet的速率上，效率进一步提高了4倍。

给定一批N个（图像，文本）对，CLIP被训练来预测在一批中实际发生的N×N个可能的（图像，文本）对。为此，CLIP通过联合训练图像编码器和文本编码器来学习多模态嵌入空间，以最大化批中N个实对的图像和文本嵌入的余弦相似性，同时最小化N2-N个不正确对的嵌入的余弦相似性。我们优化了这些相似性分数上的对称交叉熵损失。在图3中，我们包含了CLIP实现核心的伪代码。据我们所知，这一批量构建技术和目标首次被引入深度度量学习领域，被称为多类N对缺失Sohn（2016），被Oord et al.（2018）推广为对比表征学习，被称为信息缺失，最近被改编为对比（文本、图像）表征学习医学影像领域，张等（2020）。

由于我们的预训练数据集很大，过度拟合不是主要问题，与Zhang等人（2020）的实现相比，训练片段的细节简化了。我们从零开始训练剪辑，而不必使用ImageNet权值初始化图像编码器或使用预先训练的权值初始化文本编码器。我们去除了表征和对比嵌入空间之间的非线性投影，这一变化由Bach-man等人（2019）提出，并由Chen等人（2020b）推广。我们只使用线性投影将每个编码器的表示映射到多模态嵌入空间。我们没有注意到两个版本在训练效率上的差异，并且推测非线性投影可能与当前仅图像的自监督表示学习方法的细节相适应。我们还从Zhang et al.（2020）中重新移动了文本转换函数tu，该函数从文本中统一采样一个句子，因为CLIP预训练数据集中的许多（图像、文本）对只是一个句子。我们还简化了电视的图像变换功能。从调整大小的图像中随机裁剪正方形是训练过程中唯一使用的数据增强。最后，在训练过程中，将控制softmax中logit范围的温度参数τ作为对数参数化的乘法标量进行直接优化，以避免变为超参数。

### 2.4 选择和缩放模型
我们考虑了两种不同的图像编码器结构。首先，我们使用ResNet-50（He et al.，2016a）作为图像编码器的基本架构，因为它被广泛采用并且性能得到了验证。我们使用He等人（2019）的ResNet-D改进和Zhang（2019）的抗锯齿rect-2模糊池对原始版本进行了几次修改。我们还将全局平均池层替换为注意池机制。注意池被实现为“变压器式”多头部QKV注意的单一层，其中查询以图像的全局平均池表示为条件。对于第二种架构，我们使用最近引入的Vision Transformer（ViT）进行了实验（Dosovitskiy等人，2020）。我们密切关注它们的实现，只需对转换器之前的组合补丁和位置嵌入添加额外的层规范化，并使用稍微不同的初始化方案。
