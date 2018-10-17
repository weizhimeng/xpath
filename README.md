# xpath
xpath用法以及爬虫中的高级应用

XPath即为XML路径语言（XML Path Language），它是一种用来确定XML文档中某部分位置的语言。可以这样理解，文档为一个树结构，xpath就是一级一级的寻找到需要的节点。可广泛应用于爬取页面，再结合正则表达式，完成数据的提取和清洗。

## xpath基础语法
|表达式|描述|
|:---|:---|
|nodename|选取此节点的所有子节点。| 
|/|从根节点选取。(为绝对路径，不推荐使用，过长，并且容易出错。)| 
|//|从匹配选择的当前节点选择文档中的所有节点，而不考虑它们的位置。|
|.|选取当前节点。| 
|..|选取当前节点的父节点。| 
|@|选取属性。| 
|*|匹配任何元素节点。| 
|@*|匹配任何属性节点。| 
|node()|匹配任何类型的节点。| 

基础语法和linux命令差不多，以下为一些实例。 

|表达式|结果|
|:---|:---|
|bookstore|选取 bookstore 元素的所有子节点。| 
|/bookstore|选取根元素 bookstore。| 
|bookstore/book|选取属于 bookstore 的子元素的所有 book 元素。| 
|//book|选取所有 book 子元素，而不管它们在文档中的位置。| 
|bookstore//book|选择属于 bookstore 元素的后代的所有 book 元素，而不管它们位于 bookstore 之下的什么位置。| 
|//@lang|选取名为 lang 的所有属性。| 
|/bookstore/book[1]|选取属于 bookstore 子元素的第一个 book 元素。| 
|/bookstore/book[last()]|选取属于 bookstore 子元素的最后一个 book 元素。| 
|/bookstore/book[last()-1]|选取属于 bookstore 子元素的倒数第二个 book 元素。| 
|/bookstore/book[position()<3]|选取最前面的两个属于 bookstore 元素的子元素的 book 元素。| 
|//title[@lang]|选取所有拥有名为 lang 的属性的 title 元素。| 
|//title[@lang='eng']|选取所有 title 元素，且这些元素拥有值为 eng 的 lang 属性。| 
|/bookstore/book[price>35.00]|选取 bookstore 元素的所有 book 元素，且其中的 price 元素的值须大于 35.00。| 
|/bookstore/book[price>35.00]/title|选取 bookstore 元素中的 book 元素的所有 title 元素，且其中的 price 元素的值须大于 35.00。| 
|/bookstore/*|选取 bookstore 元素的所有子元素。| 
|//*|选取文档中的所有元素。| 
|//title[@*]|选取所有带有属性的 title 元素。|

## xpath模糊定位
#### starts-with 
匹配一个属性开始位置的关键字，例如： 
```
//a[starts-with(@href,'http://v')]
```
匹配href属性以http://v开头的a节点。 

#### end-with
匹配一个属性结束位置的关键字，用法和上面相同。 

#### contains
匹配一个包含关键词的属性，最常用，例如：
```
//a[contains(@id,'i')]
```
匹配id里包含i的a节点。

## 以上为xpath的基础用法，可应对大多数的情况，但有时候所需要的节点属性不容易定位或属性有很多相同的节点，就需要使用以下的Axes（轴）了。

## XPath Axes（轴）

|轴名称|结果|
|:---|:---|
|ancestor|选取当前节点的所有先辈（父、祖父等）。|
|ancestor-or-self|选取当前节点的所有先辈（父、祖父等）以及当前节点本身。|
|attribute|选取当前节点的所有属性。|
|child|选取当前节点的所有子元素。|
|descendant|选取当前节点的所有后代元素（子、孙等）。|
|descendant-or-self|选取当前节点的所有后代元素（子、孙等）以及当前节点本身。|
|following|选取文档中当前节点的结束标签之后的所有节点。|
|namespace|选取当前节点的所有命名空间节点。|
|parent|选取当前节点的父节点。|
|preceding|选取文档中当前节点的开始标签之前的所有节点。|
|preceding-sibling|选取当前节点之前的所有同级节点。|
|self|选取当前节点。|

以下为实例：

|例子|结果|
|:---|:---|
|child::book|选取所有属于当前节点的子元素的 book 节点。|
|attribute::lang|选取当前节点的 lang 属性。|
|child::*|选取当前节点的所有子元素。|
|attribute::*|选取当前节点的所有属性。|
|child::text()|选取当前节点的所有文本子节点。|
|child::node()|选取当前节点的所有子节点。|
|descendant::book|选取当前节点的所有 book 后代。|
|ancestor::book|选择当前节点的所有 book 先辈。|
|ancestor-or-self::book|选取当前节点的所有 book 先辈以及当前节点（如果此节点是 book 节点）|
|child::*/child::price|选取当前节点的所有 price 孙节点。|

组合千变万化，复杂也难记，不过大多情况下只需要使用几个简单的轴就行。之后的运算符，因为大多与平常所用的差不多，便不再赘述。之后将叙述在python爬虫中如何应用xpath并提高效率。 


## python中的应用 
```python
response_xpath = html.etree.HTML(response.text).xpath('xpath语句')
```
以上为自己最常用的使用xpath的格式。在python中，得到的xpath结果若为单个，类型为lxml.etree._Element，但若是有多个结果，则为list类型。而html.etree.HTML(response.text)表达式运算后的结果为lxml.etree._Element类型，也就意味着我们可以将xpath处理后的数据再次进行xpath处理（如果前一次处理后有多个结果需先遍历再进行处理）。这在爬虫中被广泛使用，比如处理表格的数据：
```python
trs = response.xpath('//table[@id="ex"]//tr')
    for tr in trs:
        date = ''.join(tr.xpath('.//td[@width="180"]//text()'))
        content = ''.join(tr.xpath('.//td[@width="500"]//text()'))
```

## 在selenium中xpath也是常用的一种定位方式，这也是爬虫常用的一种爬取方式。


## 最后便是偷懒方法了，chrome自带的xpath选择器只能显示第一个匹配的内容，并且有时还不准确，需要我们在代码里多次调试，尤其是只显示第一个匹配内容这个，很容易埋下隐患，使的处理后的数据莫名多了些不想干的信息，甚至导致出错。这里推荐一个拓展程序xpath helper，输入xpath语句后会显示所有匹配到的结果并且页面上的相关地方会变成黄色，可观性非常不错，如下图：















