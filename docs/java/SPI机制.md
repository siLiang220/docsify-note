SPI是由第三方实现或扩展的API,用于动态加载服务的机制，java 中的SPI机制主要思想是将装配的控制权移到控制程序之外，在模块化设计中尤其重要，核心思想就是解耦。

---
## SPI 简介
SPI 四要素：

- **SPI接口：** 为服务提供者实现类约定的接口或者抽象类。
- **SPI实现类** 为提供服务的实现类。
- **SPI配置：** java SP机制约定的配置文件，提供查找服务实现类的逻辑。配置文件必须置于META-INF/services目录中，并且文件名应与服务提供者接口的万千限定名保持一致。文件中每一行都有一个实现服务类的详细信息，同样是与服务提供这类的完全限定名称。
- **ServiceLoader：** java SPI的核心类，用于加载SPI实现类。`ServiceLoader`中有各种方法类获取特定的实现、迭代它们或重新加载服务。

## 示例

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1667810765568.png)

### SPI接口
```java
package io.github.spi;  
  
public interface ObjectStorage {  
  
    String fileUpload();  
  
    String getType();  
}
```

### SPI实现类
```java
package io.github.spi;  
  
public class S3Storage implements ObjectStorage{  
    @Override  
    public String fileUpload() {  
        return "S3 file upload";  
    }  
  
  
    @Override  
    public String getType() {  
        return "s3";  
    }  
}
```

### SPI配置
```text
io.github.spi.OssStorage  
io.github.spi.S3Storage
```

### SPI机制加载组件
```java
package io.github.spi;  
  
import org.springframework.beans.factory.InitializingBean;  
import org.springframework.beans.factory.annotation.Value;  
import org.springframework.stereotype.Component;  
  
import javax.naming.spi.InitialContextFactory;  
import java.util.HashMap;  
import java.util.Iterator;  
import java.util.Map;  
import java.util.ServiceLoader;  
  
@Component  
public class ObjectStorageFactory implements InitializingBean {  
  
//    @Value("${file.storage.type}")  
    private String type = "oss";  
    Map<String,ObjectStorage> objectStorageMap = new HashMap<>(2);  
    private ObjectStorage objectStorage;  
    public void fileUpload(){  
        objectStorage.fileUpload();  
    }  
  
    @Override  
    public void afterPropertiesSet() {  
        ServiceLoader<ObjectStorage> storages = ServiceLoader.load(ObjectStorage.class);  
        Iterator<ObjectStorage> ObjectStorageIterator = storages.iterator();  
        while (ObjectStorageIterator.hasNext()){  
            ObjectStorage ObjectStorage = ObjectStorageIterator.next();  
            String type = ObjectStorage.getType();  
            objectStorageMap.put(type,ObjectStorage);  
        }  
        //系统默认支持的文件存储类型  
        objectStorage = objectStorageMap.get(type);  
    }  
}
```

