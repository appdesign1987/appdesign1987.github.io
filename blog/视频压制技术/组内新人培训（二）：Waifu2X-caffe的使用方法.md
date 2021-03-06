
VS处理工具很多，其中很少有单独需要开一篇文章来讲的插件，Waifu2X是其中一个。

本文目录：总体介绍在第一章，原理在第二章，操作在第三章。

如果是组长叫你来看如何配置wf2x的，小白不愿意看原理可以直接看第三章操作方法。

## **一、waifu2X是什么？**
简单来说这是个图像放大工具，就像它的名字一样，人家就是专门设计这个软件来把你的老婆（纸片人）放大到二倍的。

效果大概如下图，由于使用了神经网络算法对人类绘画习惯（主要是CG）进行了学习，在放大老婆这方面，比传统方案高到不知道哪里去。
![效果对比][1]

后面的caffe表示这个版本使用caffe架构实现，并且是目前唯一能用cudnn加速的版本（下文简称为wf2xc或wf2x），要说加速有多快，相比于CPU运算版大概也就快了个一百倍吧。

A卡目前对waifu2x支持不佳，运算效率远低于N卡。

原作者是个日本死宅，读了一篇卷积神经网络（CNN）深度学习的论文（[1501.00092](https://arxiv.org/abs/1501.00092)）之后产生了骚想法，把玩过的小黄油cg都缩小丢到机器里学习后来就产生了wf2x（emmmm），vs版本由holy大大移植。

## **二、为什么要写这篇文章专门介绍wf2x？**

如同我在高压缩向视频教学第三章中提到的（然而本文发表的时点视频第三章并没有做出来），这是一个神奇的工具。虽然原本是用来放大的，但这个功能却很少被使用（在往后的教学中（也许）你会逐步理解为什么随意缩放分辨率是愚蠢的），不过wf2x有一些其他的优良特性。

按照设计上来说，它本是被用来放大图片的，但实际上由于我们做的是收藏向压制，极少改变素材分辨率，即使改变也是降低，从没有将低清素材拉高使用过。

不过由于这个东西的是用深层次卷积神经网络的方式来处理信息（深层次、大量的卷积变换，这个暂且不表），在该过程中，它会以一些人类无法理解的逻辑对2D画面的结构进行分类识别，并重构这张图片。恰好因为这点带来了一些优良的效应，比如它可以在重构的同时顺路降噪。（严格来说它是在为了去JPEG压缩效应（这里特指DCT+量化的部分）而被训练的，降噪只是附带的结果而已。不过恰好由于JPEG压缩原理中使用的块参考、DCT->量化压缩的逻辑与视频编码器的压缩原理高度相似，它同样非常适合用来对付编码器压缩造成的失真。）其降噪效果有些时候甚至比我们认为的NL-MEANS/BM3D等传统"高级降噪算法"更好。（然而该被抹的细节还是会被抹，别做梦）



![请输入图片描述][2]


比如由于waifu2x在RGB模式下转换的特性，过一遍wf2x，属于将原盘的YUV信息进行一次融合再分离。

在这个过程中会使YUV三个平面的信息互相作用，达到一些诸如抗锯齿，dering的神奇效果。（同时由于公式误差造成偏色）

由于其在上述过程中表现出的强大线条整理能力（往往一些肮脏的Chroma平面经过waifu2x后会被整理的异常干净），对提高压缩率有一定帮助（仅在感觉上，没有做过详细测试，理论上整理chroma线条会降低压缩率），所以其最近在我组方案中的使用比重似乎也有逐渐增大的趋势。


为什么要单独开一篇文章讲呢？因为目前vs移植版的wf2x比较娇贵，错误的设置往往会让工作效率降至三到四分之一，甚至使其傲娇崩溃。

##### 关于wf2x的提速原理，简单来说有两点，其一就是wf2x是以某一个大小的区块为单位工作，把原先的每1个像素变成4个(2x)/9个(3x)或16个等等以此类推。然后比如说你指定要放大到3.5倍，那就再进行一次缩小运算。于是乎，通过选择新版本wf2x中的适当模型，可以避免不必要的放大与缩小运算。

##### 另一方面通过新版本中开放的可自定义区块大小的特性，比如1080P分辨率下设置区块大小为960x540(每张1080P图片只需要渲染四次即可合成)，避免如原先500x500区块造成的运算量浪费(该例中横向需渲染4次\纵向3次才能完全覆盖图片，实际渲染分辨率为2000x1500，多进行30%无效运算)。通过这两种方式可以极大提升wf2x运算效率。

以下引用megui吧@空之飞翔之春哥大佬原话

> 这周一，waifu2x原作者重构的waifu2x网络结构，正式放出了预览模型以及在线网站。新模型在提高了放大/降噪效果的同时，还将计算量减少至原来的1/2.4-1/3，大幅提升了速度。在和作者交流的时候期间，发现cuDNN
> v5中的新特性还能额外带来明显提升。配合新架构的pascal显卡，这三个巨大改变将waifu2x从当年的龟速，一下拉到了实时处理的可能上。
> 
> 
> 根据作者的测试数据，GTX1080+cuDNN v5 
> (3x128x128 patch by waifu2x default setting, average of 100 times)
> 1-1-1-1-1-1-1(waifu2x new): 0.002844s (cuDNN v4, equally) 
> 1-6-6-6-6-6-1(waifu2x new): 0.002189s (cuDNN v5, best decision) 
> 1-1-1-6-6-6-1(waifu2x new): 0.002257s
> 
> (3x480x360 patch, average of 20 times) 
> 1-1-1-1-1-1-1(waifu2x new): 0.02715s (cuDNN v4, equally) 
> 1-6-6-6-6-6-1(waifu2x new): 0.02068s 
> 1-1-1-6-6-6-1(waifu2x new): 0.02028s (cuDNN v5, best decision)
> 
> 
> 以上数据显示，将一张480x360的图进行（降噪并）双倍放大到960x720，
> 
> 在GTX1080，最佳优化下，仅需要0.02028s，相当于 49 fps。（以dvd的720x480来看，可以到 25 fps 了）
> 我的GTX960，最佳优化下，仅需要0.1019s，相当于 9.8 fps。旧模型最多 3.6 fps。
> 
> 如果是将一张1080p的图片仅放大到到4k，960上，旧模型需要3.3s左右;而新模型仅需要1.2s。
> GTX上1080更是仅需要0.25s，4fps!!!这速度比编码器快了==
> 
> 
> 提速的方法一共有三点：
> 
> 第一、模型重构。
> 
> 以前的模型都是先用双立方算法放大一倍以后，用卷积神经网络进行修复。而新模型用了今年暂未公布的最新研究的方法，把这一步放大也做进了网络模型中，所以需要处理的像素仅为原来的1/4。与此同时增加了模型复杂度以提高效果。综合算下来性能为原来的2.4x。
> 
> 这次的模型，同时把降噪和放大做成了一个模型，也就是说消耗的时间和不降噪完全相同。而以往需单独在原分辨率下，先做一遍降噪再放大。这里可以说是为原来的1.25x。
> 
> 自此，作者在发布页上公布，新模型的性能为原来的2.4-3.0x!!! 
> “new scale method is 2.4x faster than current scale method and better quality new noise_scale method is
> 3x faster than current noise_scale method and better quality”
> 
> 
> 第二、cuDNN v5中的卷积新算法。
> 
> v5版本实现了一个WINOGRAD算法（去年的一篇论文），大幅降低了卷积的计算量，还提升了计算精度，在GTX1080上，特定网络下能看到51%的提升。gtx960上45%。
> 
> 从作者的测试显示，WINOGRAD将时间从0.02715s，缩短到0.02028s，速度达到了原来的1.3x。
> 
> Maxwell上的提升就要小一些了，GTX960我只能看到8.7%的提升。
> 
> 不过旧版本waifu2x下（单独的降噪模型还是需要旧版），用v5优化最多看到了25%的提升，1080上应该还要多一些（暂没有测试数据）。
> 
> 具体怎么利用好v5本身的优化判断逻辑，以及怎么在caffe版中同时保持对未来cuDNN版本可能的更优方法的兼容性，这个还在研究讨论中，不过提升已经明显看到了。等到具体应用到vapoursynth上，估计还得再等一段时间。
> 
> 
> 第三、Pascal新架构，以及cuDNN对其的针对性优化。
> 
> (3x480x360 patch, average of 20 times) 
> GTX 1080:-------0.02715s (cuDNN v4, equally)
> GTX 1080:-------0.02028s (cuDNN v5, best decision)
> 
> GTX 960:--------0.11130s (cuDNN v4, equally) 
> GTX 960:--------0.10150s(cuDNN v5, best decision)
> 
> 
> 本身的计算性能，1080是960的4.1x，而加上cuDNN v5在1080上的更多加成，这个差距扩大到了5.0x。（960是在cuDNN
> 5.1RC的结果） 以后cuDNN v6 v7...那些版本估计会引入更快的算法和更好的优化，Pascal和Maxwell的差距会拉的更远。

先不要激动，那么在vs中如何设置来利用这个新特性呢？

简单来说，
 1. 只需要降噪而不放大时，设置正确的模型，vs的caffe版中使用mode1（NOUPRGB模型）来避免不必要的运算。
    （反之当你需要放大的时候就需要使用upRGB一体模型来提高效率）
 2. 设置合适的区块大小，在不爆显存的前提下单个区块越大渲染效率越高。同时确保分辨率是区块的整数倍。
 3. 默认wf2x无法吃满GPU全部性能，使用多线程同时跑让性能不要浪费。

然而由于wf2x本身并未多任务并行进行优化，同时运行多任务一个不小心就会让wf2x傲娇，所以操作上也有一些技巧
以下是操作篇。

## **三、具体操作步骤**

首先关于vs环境下wf2x-caffe的安装这里不再赘述，官网提供的github链接中可以下载编译好的版本（最新版中似乎还附带了cublas和curand）；全部安装至plugins64文件夹后，只需做三件事，就能轻松地开始wf2x之旅了，他们分别是；1、确保N卡驱动更新至最新，2、安装对应版本的CUDA Toolkit，3、将对应版本的cudnn64_x.dll安装至plugins64即可。

在运行上，wf2x有一个特性是，在某目录下第一次按某设定运行wf2x时，他会在该文件夹下生成1个“cudnn_data目录”和“3x3 0x0 1x1 1.dat”这两个东西来记录学习结果。并且在上述过程中，第一次运行的最开始（几帧内）显存会有一个暴增并回落的过程（如下图，在标记位置发生），这个过程实际上是wf2x尝试各种内置模式并选取效率最高者的过程，大多数深度学习框架都有这个特性，该过程完成后关闭进程时就会把上述的学习文件从内存里般到硬盘上。

![请输入图片描述][3]

通常我们正式运行的脚本是2-3线程并行的wf2x，而若之前没有现成的学习文件供他们调用，多线程同时尝试初始化就会因缓存爆炸引起崩溃，这是wf2x崩溃的主要原因。

解决方法的具体操作是，

 1. 删除“cudnn_data目录”和“3x3 0x0 1x1 1.dat”，确保该目录下没有学习文件。
 2. 运行（通常是组长提供的）试运行vpy脚本，该脚本中只有单线程wf2x。按F5执行并渲染若干帧（通常20帧以上即可）后关掉该脚本，就会在当前目录下对应区块大小的学习文件。
 3. 这时就可以开始用正式的多线程脚本压制并且不会崩溃了。
 4. 如果依然发生崩溃，通常是由于显存爆炸导致。比如通常以960x540为区块的2线程wf2x会占用超过2G的显存，如果用2G显存的显卡运行就会导致崩溃，这时需要调小区块分辨率，比如设置到480x360，然后回到第一步重新来过。

## **四、其他碎碎念**
关于降噪强度的选择，wf2x的降噪强度一共有四档强度（0、1、2、3）而实际上1和2两档是3的选择性遮罩版。
也就是说比如说3降低100%噪声，那么2就是降低70%，剩下则30%会被留下...emmmmm
所以由于这个特性，1和2档的线条周围会得到"涂掉一些噪点、剩下一些噪点"的比较肮脏的画面。
通常做预处理时我们只是用0或3档，分别将前者视为一个目视无损的信息整理，将后者视为一个强涂抹降噪。
而做尾处理的话则12档也可以被使用。

另外，作为参考，我们提供一些组内实测数据，大家可以检查一下自己的脚本是否设置正确。
（mode=1，scale=1，noise=3，空载）
960m.........0.9FPS
1060(6G)...2.5FPS
1080(8G)...4.2FPS

虽然已经经过大幅提速(你们要知道版本更新前通常是以0.1FPS的速度在跑)，但由于wf2x运行速度仍然较慢，经常会出现GPU吃满喂不饱CPU的情况。这种时候建议用ThrottleStop等工具将CPU降频，可以在损失微量压制速度的情况下大幅减少TDP。

**清凉压缩，健康你我。**

**迷之压制组，我们下期再见。** 

  [1]: https://i1.fuimg.com/706005/c66ca131ba728942.png
  [2]: https://i1.fuimg.com/706005/ec2016e770db18ae.jpg
  [3]: https://i1.fuimg.com/706005/4af25394f181ec20.png
