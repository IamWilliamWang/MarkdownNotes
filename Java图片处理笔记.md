#ImageIO包
##简单读写图片操作
```java
BufferedImage image = ImageIO.read(new File("1.jpg"));
ImageIO.write(image, "png", new File("1.png"));
```
##复杂读图片操作
###ImageReader的创建
```java
Iterator<ImageReader> readers = ImageIO.getImageReadersByFormatName("jpg");
if(readers.hasNext())
    ImageReader reader = readers.next();
```
###ImageInputStream的创建与绑定
```java
InputStream/File source = ...;
ImageInputStream imageInputStream = ImageIO.createImageInputStream(source);
reader.setInput(imageInputStream, true);
```
###ImageReader使用方法
```java
int width = reader.getWidth(imageIndex); // 获得第一张图片的宽
int height = reader.getHeight(imageIndex); // 获得第一张图片的高
BufferedImage image = reader.read(0); // 读取第一张图片
```
###ImageReaderParam的用法
```java
ImageReadParam param = reader.getDefaultReadParam();
param.setSourceSubsampling(3, 3, 0, 0); // 指定抽样因子
BufferedImage image = reader.read(0, param); // 产生原图片1/9大小的图片
```
###ImageReader多图片读取
```java
reader.getNumImages(true); // 获取总共多少张图片
reader.getNumThumbnails(imageIndex); // 获取某张图片有多少缩略图
image = reader.readThumbnail(imageIndex, thumbnailIndex); // 获得第几张图片的第几个缩略图
```
##复杂写图片操作
###ImageWriter写图片
```java
ImageWriter writer = ImageIO.getImageWritersByFormatName("jpg").next();
File file = new File("myimage.jpg");
writer.setOutput(ImageIO.createImageOutputStream(file));
BufferedImage image = ...;
writer.write(image);
```
###IIOImage多图片写入
```java
BufferedImage first_bi = ..., second_bi = ...;
IIOImage first_IIOImage = new IIOImage(first_bi, null, null);
IIOImage second_IIOImage = new IIOImage(second_bi, null, null);
writer.write(first_IIOImage);
if (writer.canInsertImage(1))
    writer.writeInsert(1, second_IIOImage, null);
```
###Metadata的访问
```java
reader.getImageMetadata(imageIndex);
reader.getStreamMetadata();
```
#OpenCV Mat操作
注意事项：
> 最开始要加
`System.loadLibrary(Core.NATIVE_LIBRARY_NAME);`
> VM options必须加
`-Djava.library.path=E:\opencv_3.4.6\build\java\x64;E:\opencv_3.4.6\build\x64\vc15\bin`
##写Mat
###BufferedImage --> Mat
直接mat.put(r, c, image.getRGB(r, c));写入RGB会始终失败。只能一次性从(0, 0)处写入所有的byte[]
```java
BufferedImage image = ...;
int rows = image.getWidth();
int cols = image.getHeight();
int type = CvType.CV_8UC3;
Mat mat = new Mat(rows, cols, type);
byte[] pixels = ((DataBufferByte) image.getRaster().getDataBuffer()).getData();
mat.put(0, 0, pixels);
```
###byte[] --> Mat（速度最快）
```java
byte[] byte = ...;
Mat mat = Imgcodecs.imdecode(new MatOfByte(bytes), Imgcodecs.CV_LOAD_IMAGE_UNCHANGED);
```
##读Mat
###Mat --> byte[]
```java
Mat mat = ...;
MatOfByte matOfByte = new MatOfByte();
Imgcodecs.imencode(".png", mat, matOfByte); //fetal problem: 执行完matOfByte居然变为单通道！
byte[] bytes = matOfByte.toArray();
```
#BufferedImage互转byte[]
##byte[] --转--> BufferedImage
[sRGB转BufferedImage引用于此](https://blog.csdn.net/10km/article/details/51872134)
```java
/**
  * 根据指定的参数创建一个RGB格式的BufferedImage
  * @param matrixRGB RGB格式的图像矩阵
  * @param width 图像宽度
  * @param height 图像高度
  * @return
  * @see DataBufferByte#DataBufferByte(byte[], int)
  * @see ColorSpace#getInstance(int)
  * @see ComponentColorModel#ComponentColorModel(ColorSpace, boolean, boolean, int, int)
  * @see Raster#createInterleavedRaster(DataBuffer, int, int, int, int, int[], java.awt.Point)
  * @see BufferedImage#BufferedImage(java.awt.image.ColorModel, WritableRaster, boolean, java.util.Hashtable)
  */
public BufferedImage CreateRGBImage(byte[] matrixRGB, int width, int height) {
    // 检测参数合法性
    if (null == matrixRGB || matrixRGB.length != width * height * 3)
        throw new IllegalArgumentException("invalid image description");
    // 将byte[]转为DataBufferByte用于后续创建BufferedImage对象
    DataBufferByte dataBuffer = new DataBufferByte(matrixRGB, matrixRGB.length);
    // sRGB色彩空间对象
    ColorSpace colorSpace = ColorSpace.getInstance(ColorSpace.CS_sRGB);
    int[] nBits = {8, 8, 8};
    int[] bOffs = {0, 1, 2};
    ComponentColorModel colorModel = new ComponentColorModel(colorSpace, nBits, false, false,
            Transparency.OPAQUE, DataBuffer.TYPE_BYTE);
    WritableRaster raster = Raster.createInterleavedRaster(dataBuffer, width, height, width * 3, 3, bOffs, null);
    BufferedImage newImg = new BufferedImage(colorModel, raster, false, null);
    return newImg;
}
```
[Raster.createInterleavedRaster](http://jszx-jxpt.cuit.edu.cn/JavaAPI/java/awt/image/Raster.html#createInterleavedRaster(java.awt.image.DataBuffer,%20int,%20int,%20int,%20int,%20int[],%20java.awt.Point))
一般格式的图片转BufferedImage
```java
BufferedImage image=ImageIO.read(new ByteArrayInputSream(...);
ImageIO.write(bufferedImage,"gif",new ByteArrayOutputStream(...));
```
##BufferedImage --转--> byte[]
```java
BufferedImage image = ...;
ByteArrayOutputStream out = new ByteArrayOutputStream();
ImageIO.write(image, "png", out);
byte[] bytes = out.toByteArray();
```
#ByteBuffer用法
```java
/* 分配内存生成ByteBuffer */
int byteCount = 65536;
ByteBuffer buffer = ByteBuffer.allocate(byteCount);
/* 获得容量 */
int capacity = buffer.capacity();
/* 写入byte(s) */
buffer.put(0, (byte)0xFF); // 在0位置写一个字节
buffer.put((byte)0xFF); // 后面写一个字节
buffer.put(new byte[]{(byte)0xFF, (byte)0xFD}); // 后面写两个字节
/* 当前指针位置 */
int currentPosition = buffer.position(); // getPosition
buffer.position(0); // setPosition
/* 剩余空间 */
int remaining = buffer.remaining(); // 还剩余多少空间
/* limit操作 */
buffer.limit(7); // setLimit（limit在写时为capacity，读时为有效数据长度）
int limit = buffer.limit();// getLimit
/* Buffer特有的操作 */
buffer.clear(); // 把position设为0，把limit设为capacity。一般在把数据写入Buffer前调用。
buffer.flip(); // 把limit设为当前position，把position设为0。一般在从Buffer读出数据前调用。
buffer.rewind(); // 把position设为0，limit不变。一般在把数据重写入Buffer前调用。
```
#ColorConvertOp用法
转换BufferedImage的ColorSpace
```java
BufferedImage newImage = new ColorConvertOp(ColorSpace.getInstance(ColorSpace.CS_GRAY), null).filter(srcImg, null); //ColorSpace.CS_GRAY是要转换到的ColorSpace
```
#Raster用法
##获得BufferedImage原始byte[]
```java
/**
  * 获取图像RGB格式数据
  * @param image
  * @return
  */
public static byte[] getMatrixRGB(BufferedImage image){
    if(image.getType() != BufferedImage.TYPE_3BYTE_BGR){ // 如果
        // 转sRGB格式
        BufferedImage rgbImage = new BufferedImage(
                    image.getWidth(), 
                    image.getHeight(),  
                    BufferedImage.TYPE_3BYTE_BGR);
        new ColorConvertOp(ColorSpace.getInstance(ColorSpace.CS_sRGB), null).filter(image, rgbImage);
        // 从Raster对象中获取字节数组
        return (byte[]) rgbImage.getData().getDataElements(0, 0, rgbImage.getWidth(), rgbImage.getHeight(), null);
    } else {
        return (byte[]) image.getData().getDataElements(0, 0, image.getWidth(), image.getHeight(), null);
    }
}
```
##获得BufferedImage原始int\[\]
每个pixels的各通道(band)按照顺序，拼接numBands个byte成一个int
```java
// Raster操作获取某点
int [] allocArray = new int[raster.getWidth() * raster.getHeight() * raster.getNumBands()];
// allocArray不为空就给allocArray赋值并返回allocArray。allocArray可以为null
int [] pixels = raster.getPixels(0, 0, raster.getWidth(), raster.getHeight(), allocArray);
```
##获得BufferedImage单个像素点
```java
// 直接获取ARGB模型的某点值
BufferedImage image = ...;
int xIndex, yIndex = ...;
int argbConcatNumber = image.getRGB(xIndex, yIndex);
```