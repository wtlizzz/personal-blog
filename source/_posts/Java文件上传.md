title: Java文件上传，下载，压缩，解压
author: Wtli
tags:
  - Java
categories:
  - 后端
date: 2021-07-21 09:44:00
---
Java文件操作。

- 上传
- 下载
- 压缩
- 解压缩

代码比较多，比较全。。
<!--more-->

框架：SpringBoot
上传下载：Multipart
压缩解压缩：ZipOutputStream、ZipInputStream

### 文件上传

#### 环境配置

配置文件：src/main/resources/application.properties

- 支持多部分文件上载
- 定义可以上载的最大文件大小
- 配置所有上传文件将存储到的目录。

```
## MULTIPART (MultipartProperties)
# Enable multipart uploads
spring.servlet.multipart.enabled=true
# Threshold after which files are written to disk.
spring.servlet.multipart.file-size-threshold=2KB
# Max file size.
spring.servlet.multipart.max-file-size=200MB
# Max Request Size
spring.servlet.multipart.max-request-size=215MB
## File Storage Properties
# All files uploaded through the REST API will be stored in this directory
file.upload-dir=/Users/callicoder/uploads
```

<font color='grey'>
Note: Please change the file.upload-dir property to the path where you want the uploaded files to be stored.
</font>

#### 绑定配置

@ConfigurationProperties，此注解能够将配置文件中属性绑定到pojo中。

定义一个名为FileStorageProperties的POJO类来绑定所有文件存储属性。

```
@ConfigurationProperties(prefix = "file")
public class FileStorageProperties {
    private String uploadDir;

    public String getUploadDir() {
        return uploadDir;
    }

    public void setUploadDir(String uploadDir) {
        this.uploadDir = uploadDir;
    }
}
```

@ConfigurationProperties（prefix=“file”）注释在应用程序启动时执行其工作，并将带有prefix file的所有属性绑定到POJO类的相应字段。

如果将来定义其他文件属性，只需在上述类中添加相应的字段，spring boot就会自动将该字段与属性值绑定。


如果报错Springboot出现Not registered via @EnableConfigurationProperties or marked as Spring component。

解决方式：
1. 可以在@ConfigurationProperties添加上@Component注解。
2. 在其他配置类中添加@EnableConfigurationProperties注解。例如在Application运行类中添加。
```
@SpringBootApplication
@EnableConfigurationProperties({
        FileStorageProperties.class
})
public class FileDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(FileDemoApplication.class, args);
    }
}
```

#### 文件管理服务

创建FileStorageService类，负责在文件系统中存储文件并检索它们。

```
@Service
public class FileStorageService {

    private final Path fileStorageLocation;

    @Autowired
    public FileStorageService(FileStorageProperties fileStorageProperties) {
        this.fileStorageLocation = Paths.get(fileStorageProperties.getUploadDir())
                .toAbsolutePath().normalize();
        try {
            Files.createDirectories(this.fileStorageLocation);
        } catch (Exception ex) {
            throw new FileStorageException("Could not create the directory where the uploaded files will be stored.", ex);
        }
    }

    public String storeFile(MultipartFile file) {
        // Normalize file name
        String fileName = StringUtils.cleanPath(file.getOriginalFilename());
        try {
            // Check if the file's name contains invalid characters
            if(fileName.contains("..")) {
                throw new FileStorageException("Sorry! Filename contains invalid path sequence " + fileName);
            }
            // Copy file to the target location (Replacing existing file with the same name)
            Path targetLocation = this.fileStorageLocation.resolve(fileName);
            Files.copy(file.getInputStream(), targetLocation, StandardCopyOption.REPLACE_EXISTING);
            return fileName;
        } catch (IOException ex) {
            throw new FileStorageException("Could not store file " + fileName + ". Please try again!", ex);
        }
    }
}
```

流程：
1. 首先注入配置文件中的Path路径配置。
2. 创建文件存储目录。
2. 通过MultipartFile，找到fileName。
3. Files.copy方法，将文件上传到指定目录中。


#### Api编码

创建FileController类。

```
@RestController
public class FileController {

    private static final Logger logger = LoggerFactory.getLogger(FileController.class);

    @Autowired
    private FileStorageService fileStorageService;
    
    @PostMapping("/uploadFile")
    public UploadFileResponse uploadFile(@RequestParam("file") MultipartFile file) {
        String fileName = fileStorageService.storeFile(file);

        String fileDownloadUri = ServletUriComponentsBuilder.fromCurrentContextPath()
                .path("/downloadFile/")
                .path(fileName)
                .toUriString();

        return new UploadFileResponse(fileName, fileDownloadUri,
                file.getContentType(), file.getSize());
    }

    @PostMapping("/uploadMultipleFiles")
    public List<UploadFileResponse> uploadMultipleFiles(@RequestParam("files") MultipartFile[] files) {
        return Arrays.asList(files)
                .stream()
                .map(file -> uploadFile(file))
                .collect(Collectors.toList());
    }
}
```

创建api，上传单个文件，以及上传多个文件。

> 多看看多写写代码还是好！又学到了Arrays.asList。

Note：

1. 文件存储目录。
	- 如果已创建，就不会重新创建；
    - 如果没有创建，则会自动创建（FileStorageService初始化时Files.createDirectories）。
2. 文件覆盖？StandardCopyOption有三个选项
	- REPLACE_EXISTING，会覆盖之前文件（Windows || Liunx）都可用。
    - COPY_ATTRIBUTES，(Windows || Linux 都不可用)。
    - ATOMIC_MOVE，和上面一样都不可用。

#### 文件下载

```
    @GetMapping("/downloadFile/{fileName:.+}")
    public ResponseEntity<Resource> downloadFile(@PathVariable String fileName, HttpServletRequest request) {
        // Load file as Resource
        Resource resource = fileStorageService.loadFileAsResource(fileName);

        // Try to determine file's content type
        String contentType = null;
        try {
            contentType = request.getServletContext().getMimeType(resource.getFile().getAbsolutePath());
        } catch (IOException ex) {
            logger.info("Could not determine file type.");
        }

        // Fallback to the default content type if type could not be determined
        if(contentType == null) {
            contentType = "application/octet-stream";
        }

        return ResponseEntity.ok()
                .contentType(MediaType.parseMediaType(contentType))
                .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + resource.getFilename() + "\"")
                .body(resource);
    }
```


### 压缩文件

这些核心库是java.util.zip包的一部分，在这里我们可以找到所有与压缩和解压缩相关的实用程序。

#### 压缩单个文件

先说明流程，然后看一下代码。

![upload successful](/images/pasted-100.png)

首先是对inputFile和outputFile分别创建FileInputStream、FileOutputStream。

然后使用FileInputStream来对文件进行读取，在这里就不使用FileOutputStream写文件了，而是使用ZipOutputStream写文件。

对压缩包中的文件需要定义每一个文件（当有多个文件时分别定义）ZipEntry。

```
public class ZipFile {
    public static void main(String[] args) throws IOException {
        String sourceFile = "test1.txt";
        FileOutputStream fos = new FileOutputStream("compressed.zip");
        ZipOutputStream zipOut = new ZipOutputStream(fos);
        File fileToZip = new File(sourceFile);
        FileInputStream fis = new FileInputStream(fileToZip);
        ZipEntry zipEntry = new ZipEntry(fileToZip.getName());
        zipOut.putNextEntry(zipEntry);
        byte[] bytes = new byte[1024];
        int length;
        while((length = fis.read(bytes)) >= 0) {
            zipOut.write(bytes, 0, length);
        }
        zipOut.close();
        fis.close();
        fos.close();
    }
}
```

注意的点：

1. FileInputStream、FileOutputStream路径，可以使用相对路径，也可以使用绝对路径。相对路径，如果直接是文件名则指的项目的根目录，项目中其他目录如下所示：
```
String sourceFile = "src/main/java/com/aircas/client/test1.txt";
```
也可以使用绝对路径，如下所示：
```
String sourceFile = "/Users/lli/Desktop/aircas-workspace0621/aircas-client/src/main/java/com/aircas/client/test1.txt";
```
2. 如果不定义ZipEntry，会报错，就算是一个文件的压缩也需要定义entry。
![upload successful](/images/pasted-101.png)
3. 如果有了FileOutputStream重复的文件，会在代码执行时覆盖之前重名的文件。

#### 压缩多个文件

压缩多个文件，代码如下：

与单个文件的区别为，需要分别读取每一个待压缩文件，然后对每个文件定义自己的zipEntry。

```
public class ZipMultipleFiles {
    public static void main(String[] args) throws IOException {
        List<String> srcFiles = Arrays.asList("test1.txt", "test2.txt");
        FileOutputStream fos = new FileOutputStream("multiCompressed.zip");
        ZipOutputStream zipOut = new ZipOutputStream(fos);
        for (String srcFile : srcFiles) {
            File fileToZip = new File(srcFile);
            FileInputStream fis = new FileInputStream(fileToZip);
            ZipEntry zipEntry = new ZipEntry(fileToZip.getName());
            zipOut.putNextEntry(zipEntry);

            byte[] bytes = new byte[1024];
            int length;
            while((length = fis.read(bytes)) >= 0) {
                zipOut.write(bytes, 0, length);
            }
            fis.close();
        }
        zipOut.close();
        fos.close();
    }
}
```

#### 压缩文件夹

压缩文件夹，就需要使用递归的方式。

```
public class ZipDirectory {
    public static void main(String[] args) throws IOException {
        String sourceFile = "zipTest";
        FileOutputStream fos = new FileOutputStream("dirCompressed.zip");
        ZipOutputStream zipOut = new ZipOutputStream(fos);
        File fileToZip = new File(sourceFile);

        zipFile(fileToZip, fileToZip.getName(), zipOut);
        zipOut.close();
        fos.close();
    }

    private static void zipFile(File fileToZip, String fileName, ZipOutputStream zipOut) throws IOException {
        if (fileToZip.isHidden()) {
            return;
        }
        if (fileToZip.isDirectory()) {
            if (fileName.endsWith("/")) {
                zipOut.putNextEntry(new ZipEntry(fileName));
            } else {
                zipOut.putNextEntry(new ZipEntry(fileName + "/")); 
            }
            zipOut.closeEntry();
            File[] children = fileToZip.listFiles();
            for (File childFile : children) {
                zipFile(childFile, fileName + "/" + childFile.getName(), zipOut);
            }
            return;
        }
        FileInputStream fis = new FileInputStream(fileToZip);
        ZipEntry zipEntry = new ZipEntry(fileName);
        zipOut.putNextEntry(zipEntry);
        byte[] bytes = new byte[1024];
        int length;
        while ((length = fis.read(bytes)) >= 0) {
            zipOut.write(bytes, 0, length);
        }
        fis.close();
    }
}
```

Note:
- 为了压缩子目录，我们递归地遍历它们。
- 每次找到一个目录时，我们都会将其名称附加到子目录的名称中以保存层次结构。
- 我们还为每个空目录创建一个目录条目

### 解压文件

#### 解压zip压缩文件

流程和压缩文件的流程类似：


![upload successful](/images/pasted-102.png)

整体思路就是

1. 先通过ZipInputStream读取压缩文件。
2. 然后循环获取ZipInputStream.getNextEntry()来判断压缩包中有多少文件。
3. 然后通过newFile取生成文件。
4. 最后通过FileOutputStream写入文件。

```
    public static void unzipFile() throws IOException {
        String fileZip = "src/main/java/com/aircas/client/compressed.zip";
        File destDir = new File("src/main/java/com/aircas/client/test1.txt");
        byte[] buffer = new byte[1024];
        ZipInputStream zis = new ZipInputStream(new FileInputStream(fileZip));
        ZipEntry zipEntry = zis.getNextEntry();
        while (zipEntry != null) {
            File newFile = newFile(destDir, zipEntry);
            if (zipEntry.isDirectory()) {
                if (!newFile.isDirectory() && !newFile.mkdirs()) {
                    throw new IOException("Failed to create directory " + newFile);
                }
            } else {
                // fix for Windows-created archives
                File parent = newFile.getParentFile();
                if (!parent.isDirectory() && !parent.mkdirs()) {
                    throw new IOException("Failed to create directory " + parent);
                }

                // write file content
                FileOutputStream fos = new FileOutputStream(newFile);
                int len;
                while ((len = zis.read(buffer)) > 0) {
                    fos.write(buffer, 0, len);
                }
                fos.close();
            }
            zipEntry = zis.getNextEntry();
        }
        zis.closeEntry();
        zis.close();
    }

    public static File newFile(File destinationDir, ZipEntry zipEntry) throws IOException {
        File destFile = new File(destinationDir, zipEntry.getName());

        String destDirPath = destinationDir.getCanonicalPath();
        String destFilePath = destFile.getCanonicalPath();

        if (!destFilePath.startsWith(destDirPath + File.separator)) {
            throw new IOException("Entry is outside of the target dir: " + zipEntry.getName());
        }

        return destFile;
    }
```

Note：

文件什么时候生成？是new File还是FileOutputStream.write。

1. 文件夹是File.mkdirs生成
2. 文件是new FileOutputStream(newFile)生成，然后FileOutputStream.write写入内容。


#### 解压gz压缩文件

简单介绍一下zip文件与gz文件的区别：

tar本身只是将文件捆绑在一起（结果称为tarball），而zip也应用压缩。

通常使用gzip和tar来压缩结果tarball，从而获得与zip类似的结果。

不过，对于相当大的档案来说，有着重要的区别。zip存档是压缩文件的集合。gzip tar是（未压缩文件的）压缩集合。因此，zip存档是一个随机可访问的串联压缩项列表，而.tar.gz是一个存档，它必须在目录可访问之前完全展开。

zip需要注意的是不能跨文件进行压缩（因为每个文件都是独立于存档中的其他文件进行压缩的，所以压缩不能利用不同文件内容之间的相似性），优点是只需查看归档文件的特定部分（依赖于目标文件），就可以访问其中包含的任何文件（因为集合的“目录”与集合本身是分开的）。

.tar.gz需要注意的是必须解压缩整个归档文件才能访问其中包含的文件（因为文件在tarball中）；优点是压缩可以利用文件之间的相似性（因为它压缩整个tarball）。

结论：

zip文件是每个文件独立压缩，ZipInputStream是有一个ZipEntry，作为每个文件的入口。

gz文件则是直接使用GZIPInputStream来进行write，并且不能提前知道文件名，除非全部解压完。

```
    public void unGunzipFile(String compressedFile, String decompressedFile) {
        byte[] buffer = new byte[1024];
        try {
            FileInputStream fileIn = new FileInputStream(compressedFile);
            GZIPInputStream gZIPInputStream = new GZIPInputStream(fileIn);
            FileOutputStream fileOutputStream = new FileOutputStream(decompressedFile);
            int bytes_read;
            while ((bytes_read = gZIPInputStream.read(buffer)) > 0) {
                fileOutputStream.write(buffer, 0, bytes_read);
            }
            gZIPInputStream.close();
            fileOutputStream.close();
            System.out.println("The file was decompressed successfully!");
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }
```

***
附：

java.Multipart文件上传下载[参考网站](https://www.callicoder.com/spring-boot-file-upload-download-rest-api-example/)

java.util.zip压缩和解压缩的[参考网站](https://www.baeldung.com/java-compress-and-uncompress)。

比较zip和rar的[参考网站](https://stackoverflow.com/questions/10540935/what-is-the-difference-between-tar-and-zip)。