---
title: "(DESeq2) Why are some p values set to NA?"
date: 2023-06-28T05:00:38Z
draft: ["false"]
tags: [
  "fetched",
  "生信菜鸟团"
]
categories: ["Acdemic"]
---
(DESeq2) Why are some p values set to NA? by 生信菜鸟团
------
<div><section data-tool="mdnice编辑器" data-website="https://www.mdnice.com"><h2 data-tool="mdnice编辑器"><span></span>引入</h2><p data-tool="mdnice编辑器">在上一期<a href="https://mp.weixin.qq.com/s?__biz=MzUzMTEwODk0Ng==&amp;mid=2247513791&amp;idx=1&amp;sn=f317d4d2c6385f319b7fa8d43be6b7ad&amp;chksm=fa457382cd32fa9435ecc83deb99e20ca4d5d2cec0ecb2011d366e4c3babb63ef1cf7666a485&amp;scene=21&amp;cur_album_id=2545418429466607617#wechat_redirect" data-linktype="2">奇怪的转录组差异表达矩阵之实验分组</a>中，我们谈到DESeq2输出NA的问题，这周我们仍使用上周 GSE126548-分组差异并不大，这个数据集来进行分析</p><blockquote data-tool="mdnice编辑器"><p>本文主要参考bioconductor文档：Analyzing RNA-seq data with DESeq2</p><p>地址：http://bioconductor.org/packages/devel/bioc/vignettes/DESeq2/inst/doc/DESeq2.html</p></blockquote><p data-tool="mdnice编辑器">这里先贴出一般使用DESeq2分析的代码：</p><pre data-tool="mdnice编辑器"><span></span><code>counts &lt;- read.table(<span>"../11.一些奇怪的转录组差异表达矩阵/1.GSE126548-分组差异并不大//GSE126548_HTSeq_count.txt"</span>,sep = <span>'\t'</span>,header = TRUE,row.names = 1)<br>range(counts)<br><span># 0 71407700</span><br><span># 1根据表达量过滤</span><br>keep &lt;- rowSums(counts&gt;0) &gt;= floor(0.75*ncol(counts))<br>filter_count &lt;- counts[keep,] <span>#获得filter_count矩阵</span><br>dim(counts);dim(filter_count)<br><span># [1] 60160     6</span><br><span># [1] 15065     6</span><br><span># 2根据基因过滤</span><br><span># 一般行名不能一样，我们现在行名形式就是gene ENSG 应该没问题</span><br>table(duplicated(rownames(filter_count)))<br><span># FALSE </span><br><span># 15065 </span><br><span># 分组</span><br>group_list &lt;- factor(c(rep(<span>"withBM"</span>,3),rep(<span>"noBM"</span>,3)),levels = c(<span>"withBM"</span>,<span>"noBM"</span>))<br>table(group_list)<br><span># group_list</span><br><span># withBM   noBM </span><br><span># 3      3</span><br><br><span># 所以这里其实相当于只对低表达量过滤</span><br><br><span># R package DESeq, which utilizes a generalized linear model (GLM) and is applied directly to raw read counts. </span><br><span># 默认counts所以要求整数 内部有log2等 不需手动</span><br><span>#1 查看分组信息和表达矩阵数据</span><br>exprSet &lt;- filter_count<br>dim(exprSet)<br><span># 加载包</span><br>library(DESeq2)<br><span>#2 第一步，构建DESeq2的DESeq对象dds(建立DEseq数据矩阵)</span><br>colData &lt;- data.frame(row.names=colnames(exprSet),group_list=group_list)<br>colData<br><br>dds &lt;- DESeqDataSetFromMatrix(countData = exprSet,<br>                              colData = colData,<br>                              design = ~ group_list)<br><br>dds<br><span>#3 第二步，进行差异表达分析</span><br>dds2 &lt;- DESeq(dds)<br><span>#4 提取差异分析结果，trt组对untrt组的差异分析结果</span><br>tmp &lt;- results(dds2,contrast=c(<span>"group_list"</span>,<span>"withBM"</span>,<span>"noBM"</span>))<br>DEG_DESeq2_raw &lt;- as.data.frame(tmp[order(tmp<span>$padj</span>),])<br>head(DEG_DESeq2_raw)  <span># 联系之前的内容DEseq2和edge会存在p值为0的情况</span><br><span># baseMean log2FoldChange     lfcSE      stat       pvalue        padj</span><br><span># ENSG00000257671 2937.01756      -5.662520 1.0831750 -5.227706 1.716262e-07 0.001160022</span><br><span># ENSG00000121552  261.23701       6.711915 1.3209047  5.081301 3.748577e-07 0.001266831</span><br><span># ENSG00000228113  449.33229      -6.395500 1.2846913 -4.978239 6.416533e-07 0.001445645</span><br><span># ENSG00000233271   70.50139      -5.201629 1.0669793 -4.875098 1.087542e-06 0.001837674</span><br><span># ENSG00000126005  359.50238      -5.362662 1.1296268 -4.747286 2.061642e-06 0.002438831</span><br><span># ENSG00000269834   55.09172      -4.616794 0.9745453 -4.737382 2.164964e-06 0.002438831</span><br><br><span># 查看结果</span><br><span># 去除差异分析结果中包含NA值的行</span><br>DEG_DESeq2 = na.omit(DEG_DESeq2_raw)<br><span># 为什么会出现NA？</span><br><span># https://blog.csdn.net/linkequa/article/details/83116789</span><br><br><span># 筛选上下调差异基因，设定阈值</span><br>fc_cutoff &lt;-2.0<br>fdr &lt;- 0.05<span>#p值</span><br>DEG_DESeq2<span>$regulated</span> &lt;- <span>"normal"</span><br>loc_up &lt;- intersect(<span>which</span>(DEG_DESeq2<span>$log2FoldChange</span>&gt;fc_cutoff),<br>                    <span>which</span>(DEG_DESeq2<span>$pvalue</span>&lt;fdr))<br>loc_down &lt;- intersect(<span>which</span>(DEG_DESeq2<span>$log2FoldChange</span>&lt; -fc_cutoff),<br>                      <span>which</span>(DEG_DESeq2<span>$pvalue</span>&lt;fdr))<br>DEG_DESeq2<span>$regulated</span>[loc_up] &lt;- <span>"up"</span><br>DEG_DESeq2<span>$regulated</span>[loc_down] &lt;- <span>"down"</span><br>table(DEG_DESeq2<span>$regulated</span>)<br><span># down normal     up </span><br><span># 175   6389    195 </span><br></code></pre><p data-tool="mdnice编辑器">可以发现去掉包含NA的行后，影响很大</p><figure data-tool="mdnice编辑器"><img data-ratio="0.1261682242990654" data-src="https://mmbiz.qpic.cn/mmbiz_png/iaRJcrq2Los9IXBGwLZIaVOIokB0ypEib928D630WDLLQVibaQoOjQOgjT5GHRcn7Ox9rziaeIv9aATYC87rYxw34A/640?wx_fmt=png" data-type="png" data-w="642" src="https://mmbiz.qpic.cn/mmbiz_png/iaRJcrq2Los9IXBGwLZIaVOIokB0ypEib928D630WDLLQVibaQoOjQOgjT5GHRcn7Ox9rziaeIv9aATYC87rYxw34A/640?wx_fmt=png"></figure><p data-tool="mdnice编辑器">在这个过程中我们使用了DESeq2的默认设置，也就意味着除了手动根据低表达量和基因名进行过滤，我们还使用了DESeq2内部的自动过滤</p><ul data-tool="mdnice编辑器"><li><section>Pre-filtering</section></li></ul><p data-tool="mdnice编辑器">在上面的代码中，我们使用 <code>keep &lt;- rowSums(counts&gt;0) &gt;= floor(0.75*ncol(counts))</code>进行过滤，也就是保留那些样本基因表达量不为0不少于样本数四分之三的基因</p><p data-tool="mdnice编辑器">在官方的文档中对于Pre-filtering，有这么一段描述：</p><blockquote data-tool="mdnice编辑器"><p>http://bioconductor.org/packages/devel/bioc/vignettes/DESeq2/inst/doc/DESeq2.html#pre-filtering</p><p>虽然在运行DESeq2函数之前没有必要对低计数基因进行预过滤，但有两个原因使预过滤变得有用：1减少了dds数据对象的内存大小，并提高了DESeq2中转换和检测函数的速度；2改善可视化效果。</p></blockquote><p data-tool="mdnice编辑器">这是因为DESeq2本身有Independent filtering（更严格）</p><p data-tool="mdnice编辑器">同时，官方也给出了“a minimal pre-filtering”<code>rowSums(counts(dds)) &gt;=10</code> , 也就是保留所有样本基因表达量计数和大于10的基因</p><ul data-tool="mdnice编辑器"><li><section>Independent filtering</section></li></ul><p data-tool="mdnice编辑器"><strong>Filtering criteria：</strong></p><p data-tool="mdnice编辑器">http://bioconductor.org/packages/devel/bioc/vignettes/DESeq2/inst/doc/DESeq2.html#indfilttheory</p><p data-tool="mdnice编辑器"><strong>Independent filtering of results：</strong></p><p data-tool="mdnice编辑器">http://bioconductor.org/packages/devel/bioc/vignettes/DESeq2/inst/doc/DESeq2.html#indfilt</p><p data-tool="mdnice编辑器">DESeq2包的 <code>results</code>函数默认情况下使用归一化计数的平均值作为过滤统计信息来执行独立过滤，找到过滤统计量的阈值，该阈值优化了低于显著性水平α的调整后的p值的数量，未通过过滤阈值的基因的调整后的p值被设置为NA</p><p data-tool="mdnice编辑器">默认的独立过滤是使用genefilter包的 <code>filtered_p</code>函数执行的，<code>filtered_p</code>的所有参数都可以传递给 <code>results</code>函数。过滤阈值和过滤统计量的每个分位数处的拒绝次数可用作结果返回的对象的元数据metadata</p><p data-tool="mdnice编辑器">例如，我们可以通过绘制results对象的 <code>filterNumRej</code>属性来可视化优化。<code>results</code>函数在过滤统计量的分位数（归一化计数的平均值）上最大化拒绝次数（调整后的p值小于显著性水平）。所选择的阈值（垂直线）是过滤的最低分位数，对于该分位数，拒绝次数在拟合过滤分位数上拒绝次数的曲线峰值的1个残差标准偏差内：</p><pre data-tool="mdnice编辑器"><span></span><code>metadata(tmp)<span>$alpha</span><br><span># [1] 0.1</span><br>metadata(tmp)<span>$filterThreshold</span><br><span># 54.28571% </span><br><span># 14.34643 </span><br></code></pre><pre data-tool="mdnice编辑器"><span></span><code>plot(metadata(tmp)<span>$filterNumRej</span>, <br>     <span>type</span>=<span>"b"</span>, ylab=<span>"number of rejections"</span>,<br>     xlab=<span>"quantiles of filter"</span>)<br>lines(metadata(tmp)<span>$lo</span>.fit, col=<span>"red"</span>)<br>abline(v=metadata(tmp)<span>$filterTheta</span>)<br></code></pre><figure data-tool="mdnice编辑器"><img data-ratio="0.7563909774436091" data-src="https://mmbiz.qpic.cn/mmbiz_png/iaRJcrq2Los9IXBGwLZIaVOIokB0ypEib9UBc4LUHpmuosCjAD2yQl4jWibzPkxZoxkekaP2soLPqST6k70G53ic8w/640?wx_fmt=png" data-type="png" data-w="665" src="https://mmbiz.qpic.cn/mmbiz_png/iaRJcrq2Los9IXBGwLZIaVOIokB0ypEib9UBc4LUHpmuosCjAD2yQl4jWibzPkxZoxkekaP2soLPqST6k70G53ic8w/640?wx_fmt=png"></figure><p data-tool="mdnice编辑器">可以通过设置 <code>independentFiltering</code> 为 <code>FALSE</code>关闭独立过滤</p><pre data-tool="mdnice编辑器"><span></span><code><span># Independent filtering can be turned off by setting independentFiltering to FALSE.</span><br>tmpNoFilt &lt;- results(dds2, independentFiltering=FALSE)<br>addmargins(table(filtering=(tmp<span>$padj</span> &lt; .1),<br>                 noFiltering=(tmpNoFilt<span>$padj</span> &lt; .1)))<br><span># noFiltering</span><br><span># filtering FALSE TRUE  Sum</span><br><span># FALSE  6719    0 6719</span><br><span># TRUE     17   23   40</span><br><span># Sum    6736   23 6759</span><br></code></pre><p data-tool="mdnice编辑器">查看关闭过滤后的结果：</p><pre data-tool="mdnice编辑器"><span></span><code>DEG_DESeq2_NoFilt &lt;- as.data.frame(tmpNoFilt[order(tmpNoFilt<span>$padj</span>),])<br><span># &gt; dim(DEG_DESeq2_NoFilt)</span><br><span># [1] 15065     6</span><br><span># &gt; dim(na.omit(DEG_DESeq2_NoFilt))</span><br><span># [1] 14934     6</span><br><span># 为什么还是有NA？</span><br><span># 发现仍为NA的 pvalue就已为NA了</span><br></code></pre><p data-tool="mdnice编辑器">可以发现仍存在一些基因p值为NA，所有这些基因和之前大部分的区别在于，它们的pvalue就已经为NA</p><figure data-tool="mdnice编辑器"><img data-ratio="0.45576923076923076" data-src="https://mmbiz.qpic.cn/mmbiz_png/iaRJcrq2Los9IXBGwLZIaVOIokB0ypEib9o6PLmGhteygS62fklZLHlRFGtv97B1tUplBqSfCDuFfxa9zicx1hdag/640?wx_fmt=png" data-type="png" data-w="1040" src="https://mmbiz.qpic.cn/mmbiz_png/iaRJcrq2Los9IXBGwLZIaVOIokB0ypEib9o6PLmGhteygS62fklZLHlRFGtv97B1tUplBqSfCDuFfxa9zicx1hdag/640?wx_fmt=png"></figure><figure data-tool="mdnice编辑器"><img data-ratio="0.45185185185185184" data-src="https://mmbiz.qpic.cn/mmbiz_png/iaRJcrq2Los9IXBGwLZIaVOIokB0ypEib9OdlbFI4eXIS3TZ2u8WE750Vxgw7IWW32wptsMYePPRFSW7icqAxicQ2w/640?wx_fmt=png" data-type="png" data-w="1080" src="https://mmbiz.qpic.cn/mmbiz_png/iaRJcrq2Los9IXBGwLZIaVOIokB0ypEib9OdlbFI4eXIS3TZ2u8WE750Vxgw7IWW32wptsMYePPRFSW7icqAxicQ2w/640?wx_fmt=png"></figure><ul data-tool="mdnice编辑器"><li><section>How can I get unfiltered DESeq2 results?</section></li></ul><p data-tool="mdnice编辑器">http://bioconductor.org/packages/devel/bioc/vignettes/DESeq2/inst/doc/DESeq2.html#how-can-i-get-unfiltered-deseq2-results</p><blockquote data-tool="mdnice编辑器"><p>Users can obtain unfiltered GLM results, i.e. without <strong>outlier removal</strong> or <strong>independent filtering</strong> with the following call:</p></blockquote><pre data-tool="mdnice编辑器"><span></span><code>dds2.2 &lt;- DESeq(dds, minReplicatesForReplace=Inf)<br>tmpUnFilt &lt;- results(dds2.2, cooksCutoff=FALSE, independentFiltering=FALSE)<br>DEG_DESeq2_UnFilt &lt;- as.data.frame(tmpUnFilt[order(tmpUnFilt<span>$padj</span>),])<br>dim(DEG_DESeq2_UnFilt)<br><span># [1] 15065     6</span><br>dim(na.omit(DEG_DESeq2_UnFilt))<br><span># [1] 15065     6</span><br></code></pre><p data-tool="mdnice编辑器">这时完全属于unfiltered</p><p data-tool="mdnice编辑器">可以发现DESeq2内部过滤不仅仅有<strong>independent filtering</strong>还存在<strong>outlier removal</strong></p><hr data-tool="mdnice编辑器"><h2 data-tool="mdnice编辑器"><span></span>Why are some <em>p</em> values set to NA?</h2><p data-tool="mdnice编辑器">http://bioconductor.org/packages/devel/bioc/vignettes/DESeq2/inst/doc/DESeq2.html#pvaluesNA</p><figure data-tool="mdnice编辑器"><img data-ratio="0.19166666666666668" data-src="https://mmbiz.qpic.cn/mmbiz_png/iaRJcrq2Los9IXBGwLZIaVOIokB0ypEib9CibOVj1yA38UN9p4sicFTYJZ9jkNibdbXYXdPDtib3JFX6mrqR4mpZDKBg/640?wx_fmt=png" data-type="png" data-w="1080" src="https://mmbiz.qpic.cn/mmbiz_png/iaRJcrq2Los9IXBGwLZIaVOIokB0ypEib9CibOVj1yA38UN9p4sicFTYJZ9jkNibdbXYXdPDtib3JFX6mrqR4mpZDKBg/640?wx_fmt=png"></figure><ul data-tool="mdnice编辑器"><li><section>如果在一行中，所有样本的计数都为零，则基础平均值（baseMean）列将为零，<strong>log2 FC、p值和调整后的p值</strong>都将被设置为NA</section></li></ul><ul data-tool="mdnice编辑器"><li><section>如果一行平均归一化计数较低，会被自动独立过滤掉，<strong>只有调整后的p值将被设置为NA</strong></section></li></ul><p data-tool="mdnice编辑器">上述两条都很好理解，我们往期推文无论是使用DESeq2、edgeR还是limma，都或多或少考虑到了这些</p><p data-tool="mdnice编辑器">我们将重点看看outlier removal</p><ul data-tool="mdnice编辑器"><li><section>如果一行包含一个具有极端计数异常值的样本，则<strong>p值和调整后的p值将被设置为NA</strong>。这些异常计数值由Cook距离检测到。自定义离群值过滤和替换离群值计数并进行重新拟合的功能描述如下</section></li></ul><blockquote data-tool="mdnice编辑器"><p>Approach to count outliers</p><p>http://bioconductor.org/packages/devel/bioc/vignettes/DESeq2/inst/doc/DESeq2.html#outlier</p></blockquote><p data-tool="mdnice编辑器">RNA测序数据有时会包含与实验或研究设计明显无关的非常大的计数的孤立实例，这些计数可能被视为异常值。异常值可能产生的原因有很多，包括罕见的技术或实验人工制品，在遗传上不同的样品中读取映射问题，以及真实但罕见的生物事件。</p><p data-tool="mdnice编辑器">在很多情况下，用户主要关注表现一致的基因，这就是为什么默认情况下，DESeq2会<strong>过滤</strong>受这些异常值影响的基因，而<strong>如果有足够的样本，异常值计数将被替换以进行模型拟合</strong>，这两种方式将在下面进行介绍：</p><p data-tool="mdnice编辑器">DESeq函数对每个基因和每个样本进行计算，用一种叫做Cook距离的异常值诊断检测。Cook距离是衡量单个样本对基因拟合系数影响程度的指标，而较大的Cook距离则表示存在异常值。Cook距离的矩阵存储在 <code>assays(dds)[["cooks"]]</code>中。</p><p data-tool="mdnice编辑器"><strong><code>results</code>函数会自动标记那些在具有3个或更多重复样本的情况下，包含高于Cooks距离截止值的基因。这些基因的p值和调整后的p值将被设置为NA。至少需要3个重复样本才能进行标记，因为仅有2个重复样本时很难判断哪个样本是异常值。可以使用 <code>results(dds, cooksCutoff=FALSE)</code>命令关闭这个过滤功能。</strong></p><p data-tool="mdnice编辑器"><strong>当自由度很大——即样本数远大于要估计的参数数时，完全因为一个计数异常值而从分析中移除整个基因是不可取的。当给定样本的重复次数为7次或更多次时，DESeq函数将自动用所有样本的修剪均值来替换大的Cook距离值，该平均值经过该样本的尺寸因子或正则化因子进行缩放。这种方法是谨慎的，它不会导致假阳性，因为它用空假设预测的值替换异常值。这种异常值替换仅在有7个或更多个重复时发生，并且可以使用 <code>DESeq(dds，minReplicatesForReplace = Inf)</code>命令关闭。</strong></p><p data-tool="mdnice编辑器">上述行文提到的两种方式的默认Cooks距离截止值取决于样本大小和要估计的参数数量。默认值是使用F(p，m-p)分布的99%分位数（其中p是参数数量，包括截距，m是样本数）。</p><p data-tool="mdnice编辑器">基因标记的默认值可以使用 <code>results</code>函数的 <code>cooksCutoff</code>参数进行修改。对于异常值替换，在 <code>DESeq</code>中保留原始计数，并将替换计数保存为矩阵，命名为 <code>assays(dds)</code>中的 <code>replaceCounts</code>。请注意，如果在设计中存在连续自变量，则不会自动执行异常值检测和替换，因为我们当前的方法涉及对组内方差进行鲁棒估计，难以简单地扩展到连续协变量。不过，用户可以通过 <code>assays(dds)[["cooks"]]</code>检查Cooks距离，以便在必要时进行手动可视化和过滤。</p><blockquote data-tool="mdnice编辑器"><p>基因标记 "gene flagging"是指DESeq2在RNA测序数据分析中，针对每个基因对所有样本进行异常值检测将存在异常值的样本标记出来。当一个样本的Cooks距离超过F(p,m-p)分布的0.99分位数时，DESeq2会将其标记为异常值。</p></blockquote><p data-tool="mdnice编辑器"><strong>Note on many outliers:</strong> 如果summary（res）（我们上述代码中应是tmp变量）报告了非常多的异常值（例如数百个或数千个），则可以考虑进一步探索，以查看是否应由于质量不佳而去除一个或几个样本。自动异常值过滤/替换在仅有少量异常值的情况下最为有用。当报告的异常值数量有数千个时，可能更有意义地关闭异常值过滤/替换（使用 <code>DESeq</code>函数中的 <code>minReplicatesForReplace = Inf</code>和 <code>results</code>函数中的 <code>cooksCutoff = FALSE）</code>，并进行手动检查：</p><p data-tool="mdnice编辑器">首先可以制作PCA图来检测单个样本的异常值;其次，可以制作Cook’s距离的箱型图，以查看某个样本是否始终比其他样本高</p><pre data-tool="mdnice编辑器"><span></span><code><span>######Approach to count outliers######</span><br><br>par(mar=c(8,5,2,2))<br>boxplot(log10(assays(dds2)[[<span>"cooks"</span>]]), range=0, las=2)<br></code></pre><figure data-tool="mdnice编辑器"><img data-ratio="0.7555555555555555" data-src="https://mmbiz.qpic.cn/mmbiz_png/iaRJcrq2Los9IXBGwLZIaVOIokB0ypEib94xuFeG6bbfmrDqeVhJCaicCMOq7peYSeVichrR5qMuCppfauUM1q1Kag/640?wx_fmt=png" data-type="png" data-w="1080" src="https://mmbiz.qpic.cn/mmbiz_png/iaRJcrq2Los9IXBGwLZIaVOIokB0ypEib94xuFeG6bbfmrDqeVhJCaicCMOq7peYSeVichrR5qMuCppfauUM1q1Kag/640?wx_fmt=png"></figure><p data-tool="mdnice编辑器">当然，这里不存在这种情况</p><hr data-tool="mdnice编辑器"><h2 data-tool="mdnice编辑器"><span></span>小结</h2><p data-tool="mdnice编辑器">在本文中，我们介绍了三种DESeq2结果输出NA的情况：</p><ul data-tool="mdnice编辑器"><li><section>如果在一行中，<strong>所有样本的计数都为零</strong>，则基础平均值（baseMean）列将为零，<strong>log2 FC、p值和调整后的p值</strong>都将被设置为NA</section></li></ul><ul data-tool="mdnice编辑器"><li><section>如果一行<strong>平均归一化计数较低</strong>，会被自动独立过滤掉，<strong>只有调整后的p值将被设置为NA</strong></section></li><li><section><strong>如果一行包含一个具有极端计数异常值的样本</strong>，则<strong>p值和调整后的p值将被设置为NA</strong>。这些异常计数值由Cook距离检测到。自定义离群值过滤和替换离群值计数并进行重新拟合的功能描述如下</section></li></ul><p data-tool="mdnice编辑器">大家可以联系自己的表达矩阵和差异分析结果对感兴趣的基因进行解读</p><p data-tool="mdnice编辑器">同时，我们着重介绍了基因计数异常值的处理，包括小样本（但大于3）中的直接过滤和大样本（大于7，大自由度）中类似处理缺失值的拟合替换</p><p data-tool="mdnice编辑器">关于官方文档，很多内容都非常详尽，感兴趣的同学可以挑自己需要的部分进行阅读</p><p data-tool="mdnice编辑器">http://bioconductor.org/packages/devel/bioc/vignettes/DESeq2/inst/doc/DESeq2.html</p></section><p><br></p><p><mp-style-type data-value="10000"></mp-style-type></p></div>  
<hr>
<a href="https://mp.weixin.qq.com/s/j037Ff8cpvysKUFN3WZm_A",target="_blank" rel="noopener noreferrer">原文链接</a>