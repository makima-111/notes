# Jelly_cs

## 一、概况

* 技术栈：
  * java
  * jsp
  * spring
* 项目功能：
  * jelly客服系统

## 二、笔记

* 主界面：
  * path：authorization/index
  * jsp地址：/admincs/index
  * 点击左侧分项后
    * 发送get请求
    * tomcat接收path
    * 返回相应页面
  * 系统管理->账号管理->菜单->添加页面
* 参考页面
  * 玩家数据上传（上传数据）
  * 兑换码生成
    * questionquery/rewardcode_create

## 三、更新

### 1、20210610身份信息上传系统

#### 需求

* 上传图片和用户信息

* 图片存到s3

* 信息发给mfctrl

* 流程：

  1. 输入信息
  2. 上传图片并展示（存到服务器）
  3. 存储信息（传至s3->发送请求）

* 接口

  > 1. 接收参数
  >     uid             必须
  >     gameToken      必须
  >     cardType        必须  认证类型 0:身份证 1:军人 2:港澳台身份证 3:外国护照 4:其他证件 5:渠道认证(此类情况，根据渠道，可能无法获取用户性别，甚至出生日期)
  >     attachment      必须  证件照路径地址 
  >     username        可空
  >     sex             可空，默认-1 0：女   1：男
  >     birthday        可空, 默认 20000101
  >
  > 2. 请求实例
  >    
  >
  > curl -v --request POST 'http://oper.test.microfun.cn/gateway2/certificationStrict/customerCertify.do' --header 'Content-Type: application/x-www-form-urlencoded' --data 'uid=99&gameToken=51bcd0&cardType=2&attachment=www.baidu.com&username=test-name&sex=1&birthday=20100101'
  >
  > 线上没有部署
  > 查看数据方式，登录数据库：
  >
  > psql -h 172.16.0.41 -p 5432 -U op_common -d oper_db
  > 密码：mf_oper#common#2017
  >
  > select * from user_certify_nppa_gov where uid = 99 and game_token='51bcd0';
  >
  > 3. 返回结果
  > status是0，表示成功，1表示失败
  >
  > {
  >     "msg": "参数错误",
  >     "status": "1"
  > }
  >
  > {
  >     "msg": "suc",
  >     "status": "0"
  > }

#### 实现

##### 页面布局

```
用户ID(必填)：box
证件类型（必填）：box
用户昵称：box
性别：select
```

```html
<label class="control-label x100" style="margin-top:10px">奖励个数：</label>
            <input type="text" id="num" name="num" size="20" value="${num}">

<select  class="control-label x100" data-toggle="selectpicker" id="count" name="count" data-rule="required" onchange="changeDbk()">
                <option value="1">1种</option>
                <option value="2">2种</option>
                <option value="3">3种</option>
                <option value="4">4种</option>
            </select>

<label>问题更新截止时间：</label>
            <input type="text" name="endTimeUC" value="${endTimeUC}" data-toggle="datepicker" data-rule="datetime"
                   data-pattern="yyyy-MM-dd HH:mm:ss" size="20">&nbsp;

<button type="button" onclick="reward_create()" class="btn-red" style="margin-left:2%; height:30px; width:140px;" >上传数据</button>
```

```java
//处理二进制数据
String code = FileUtils.getParameter(myrequest,"code"); 
String productname = FileUtils.getParameter(myrequest,"productname"); 
String protype = FileUtils.getParameter(myrequest,"protype"); 
```

```java
import java.awt.Component;   
import java.awt.Graphics2D;   
import java.awt.GraphicsConfiguration;   
import java.awt.GraphicsDevice;   
import java.awt.GraphicsEnvironment;   
import java.awt.Image;   
import java.awt.MediaTracker;   
import java.awt.image.BufferedImage;   
import java.awt.image.ColorModel;   
import java.awt.image.PixelGrabber;   
import java.io.ByteArrayInputStream;   
import java.io.IOException;   
import java.io.InputStream;   
import java.io.OutputStream;   
import java.util.Iterator;   
import javax.imageio.ImageIO;   
import javax.imageio.ImageReader;   
import javax.imageio.stream.MemoryCacheImageInputStream;   
import net.jmge.gif.Gif89Encoder;   
import org.apache.commons.logging.Log;   
import org.apache.commons.logging.LogFactory;   
import com.sun.imageio.plugins.bmp.BMPImageReader;   
import com.sun.imageio.plugins.gif.GIFImageReader;   
import com.sun.imageio.plugins.jpeg.JPEGImageReader;   
import com.sun.imageio.plugins.png.PNGImageReader;   
/**  
 * @author Erick Kong  
 * @see ImageUtil.java  
 * @createDate: 2007-6-22  
 * @version 1.0  
 */  
public class ImageUtil {   
    
 public static final String TYPE_GIF = "gif";   
 public static final String TYPE_JPEG = "jpeg";   
 public static final String TYPE_PNG = "png";   
 public static final String TYPE_BMP = "bmp";   
 public static final String TYPE_NOT_AVAILABLE = "na";   
 private static ColorModel getColorModel(Image image)   
   throws InterruptedException, IllegalArgumentException {   
  PixelGrabber pg = new PixelGrabber(image, 0, 0, 1, 1, false);   
  if (!pg.grabPixels())   
   throw new IllegalArgumentException();   
  return pg.getColorModel();   
 }   
 private static void loadImage(Image image) throws InterruptedException,   
   IllegalArgumentException {   
  Component dummy = new Component() {   
   private static final long serialVersionUID = 1L;   
  };   
  MediaTracker tracker = new MediaTracker(dummy);   
  tracker.addImage(image, 0);   
  tracker.waitForID(0);   
  if (tracker.isErrorID(0))   
   throw new IllegalArgumentException();   
 }   
 public static BufferedImage createBufferedImage(Image image)   
   throws InterruptedException, IllegalArgumentException {   
  loadImage(image);   
  int w = image.getWidth(null);   
  int h = image.getHeight(null);   
  ColorModel cm = getColorModel(image);   
  GraphicsEnvironment ge = GraphicsEnvironment   
    .getLocalGraphicsEnvironment();   
  GraphicsDevice gd = ge.getDefaultScreenDevice();   
  GraphicsConfiguration gc = gd.getDefaultConfiguration();   
  BufferedImage bi = gc.createCompatibleImage(w, h, cm.getTransparency());   
  Graphics2D g = bi.createGraphics();   
  g.drawImage(image, 0, 0, null);   
  g.dispose();   
  return bi;   
 }   
 public static BufferedImage readImage(InputStream is) {   
  BufferedImage image = null;   
  try {   
   image = ImageIO.read(is);   
  } catch (IOException e) {   
   // TODO Auto-generated catch block   
   e.printStackTrace();   
  }   
  return image;   
 }   
    
 public static BufferedImage readImage(byte[] imageByte) {   
  ByteArrayInputStream bais = new ByteArrayInputStream(imageByte);   
  BufferedImage image = readImage(bais);   
  return image;   
 }   
 public static void encodeGIF(BufferedImage image, OutputStream out)   
   throws IOException {   
  Gif89Encoder encoder = new Gif89Encoder(image);   
  encoder.encode(out);   
 }   
 /**  
  *   
  * @param bi  
  * @param type  
  * @param out  
  */  
 public static void printImage(BufferedImage bi, String type,   
   OutputStream out) {   
  try {   
   if (type.equals(TYPE_GIF))   
    encodeGIF(bi, out);   
   else  
    ImageIO.write(bi, type, out);   
  } catch (IOException e) {   
   // TODO Auto-generated catch block   
   e.printStackTrace();   
  }   
 }   
 /**  
  * Get image type from byte[]  
  *   
  * @param textObj  
  *            image byte[]  
  * @return String image type  
  */  
 public static String getImageType(byte[] textObj) {   
  String type = TYPE_NOT_AVAILABLE;   
  ByteArrayInputStream bais = null;   
  MemoryCacheImageInputStream mcis = null;   
  try {   
   bais = new ByteArrayInputStream(textObj);   
   mcis = new MemoryCacheImageInputStream(bais);   
   Iterator itr = ImageIO.getImageReaders(mcis);   
   while (itr.hasNext()) {   
    ImageReader reader = (ImageReader) itr.next();   
    if (reader instanceof GIFImageReader) {   
     type = TYPE_GIF;   
    } else if (reader instanceof JPEGImageReader) {   
     type = TYPE_JPEG;   
    } else if (reader instanceof PNGImageReader) {   
     type = TYPE_PNG;   
    } else if (reader instanceof BMPImageReader) {   
     type = TYPE_BMP;   
    }   
    reader.dispose();   
   }   
  } finally {   
   if (bais != null) {   
    try {   
     bais.close();   
    } catch (IOException ioe) {   
     if (_log.isWarnEnabled()) {   
      _log.warn(ioe);   
     }   
    }   
   }   
   if (mcis != null) {   
    try {   
     mcis.close();   
    } catch (IOException ioe) {   
     if (_log.isWarnEnabled()) {   
      _log.warn(ioe);   
     }   
    }   
   }   
  }   
  if (_log.isDebugEnabled()) {   
   _log.debug("Detected type " + type);   
  }   
  return type;   
 }   
 private static Log _log = LogFactory.getLog(ImageUtil.class);   
}

```



