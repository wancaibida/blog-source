title: Java BufferedImage OutOfMemoryError
author: 大丈夫没问题
tags:
  - java
  - bufferedimage
  - outofmemoryerror
categories:
  - java
date: 2019-12-06 12:10:00
---
最近遇到了BufferedImage OutOfMemoryError的问题，在此记录一下。

事情的起因是项目中有个功能需要将多张图片合并为一张，流程如下：

1. 有三个图片文件，`File A,File B,File C`
2. 通过`ImageIO.read(inputStream)` 将ABC转换成`BufferedImage`类型
3. 通过`java.awt.image.BufferedImage#getWidth()`获取这三张图片中的最大宽度`maxWidth`，
4. 计算另外两张图片等比例拉伸至`maxWidth`后的高度，将这三张图片的高度相加得到`maxHeight`
5. 创建一张`maxWidth * maxHeight`的图片
6. 通过画布将三张图片写入到这张图片里

主要代码如下：

```
...
                val images = mutableListOf<BufferedImage>()
                ...
                images.add(ImageIO.read(inputStream))
                ...
                joinImages(*images.toTypedArray())
...

    fun joinImages(vararg imgs: BufferedImage): BufferedImage {
        val offset = 20

        val aggregateWidth = imgs.maxBy { it.width }!!.width
        val aggregateHeight = imgs.sumBy {
            (it.height * aggregateWidth) / it.width
        } + (imgs.size - 1) * offset

        val newImage = BufferedImage(aggregateWidth, aggregateHeight, BufferedImage.TYPE_INT_ARGB)
        val g2 = newImage.createGraphics()
        val oldColor = g2.color

        g2.paint = Color.white
        g2.fillRect(0, 0, aggregateWidth, aggregateHeight)
        g2.color = oldColor

        var y = 0

        imgs.forEach {
            val height = (aggregateWidth * it.height) / it.width
            val scaled = it.getScaledInstance(aggregateWidth, height, Image.SCALE_DEFAULT)

            g2.drawImage(toBufferedImage(scaled), null, 0, y)
            y += height + offset
        }
        g2.dispose()

        return newImage
    }
```

运行过程中发现第`4`和`17`行代码时不时会抛出OutOfMemoryError错误，研究了下发现有两个问题：

1. 对于jpeg格式的图片，java将其载入到内存时是不会对其进行压缩的，一个像素会占用3个字节的内存，如果图片的尺寸比较大，会占用非常大的内存，比如这张[图片](https://dummyimage.com/2000x2000/000/fff.jpg)，实际文件大小为84kb，载入到内存里后的大小为11mb：

```
    val file = File("/Downloads/fff.jpeg")
    val image = ImageIO.read(FileInputStream(file))

    println(image.width)  //2000
    println(image.height) //2000
    println(ObjectSizeCalculator.getObjectSize(image)) //12000928  -> 11mb
```

2. 合并图片前代码将这三张图片一次性全部加载到内存里，这也会占用比较大的内存。

对于问题1，目前只能规避这问题，加载图片前会预先判断下该图片会占用多少内存，对于会超出内存使用的jpeg图片不予合并。同时在代码第14行，我们对合并后的最大宽度进行限制，避免合并后的图片尺寸过大，占用的内存超出限制。

对于问题2，这三张图片可以按序加载，不用一次性全部加载到内存里，在需要合并时才加载到内存里，同时可以改进获取图片尺寸的代码，不用将图片加载到内存后再获取尺寸。

最终代码如下：

```
    fun getImageSize(file: File): Dimension {
        ImageIO.createImageInputStream(file).use { `in` ->
            val readers = ImageIO.getImageReaders(`in`)
            if (readers.hasNext()) {
                val reader = readers.next()
                try {
                    reader.input = `in`
                    return Dimension(reader.getWidth(0), reader.getHeight(0))
                } finally {
                    reader.dispose()
                }
            }
        }

        return Dimension(0, 0)
    }
    
    
    fun joinImages(vararg files: File): File {
        ...
        val offset = 20
        val dimensions = files.map { getImageSize(it) }
        val aggregateWidth = min(dimensions.maxBy { it.width }!!.width, 800)
        val aggregateHeight = dimensions.sumBy { (it.height * aggregateWidth) / it.width } + (files.size - 1) * offset
        val newImage = try {
            BufferedImage(aggregateWidth, aggregateHeight, BufferedImage.TYPE_INT_ARGB)
        } catch (e: OutOfMemoryError) {
            ...
            throw e
        }
        val g2 = newImage.createGraphics()
        val oldColor = g2.color

        g2.paint = Color.white
        g2.fillRect(0, 0, aggregateWidth, aggregateHeight)
        g2.color = oldColor

        var y = 0

        files.forEach {
            var image = try {
                ImageIO.read(it)
            } catch (e: OutOfMemoryError) {
                ...
                throw e
            }
            val height = (aggregateWidth * image.height) / image.width
            val scaled = image.getScaledInstance(aggregateWidth, height, Image.SCALE_DEFAULT)

            g2.drawImage(toBufferedImage(scaled), null, 0, y)

            image.flush()
            image = null

            y += height + offset
        }
        g2.dispose()

        val joinImageFile = File.createTempFile(UUID.randomUUID().toString(), ".png")

        ImageIO.write(newImage, "png", joinImageFile)

        return joinImageFile
    }
```

按上述代码修改后，再也没有发生OutOfMemoryError。对于问题1，目前发现[apache commons-imaging](https://commons.apache.org/proper/commons-imaging/whyimaging.html)似乎可以解决这问题，有时间去尝试下，到时候再来更新本文。




## 参考：
https://coderanch.com/t/416485/java/Java-BufferedImage-OutOfMemoryError