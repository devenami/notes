## 通过java代码对zip文件进行压缩或解压操作

### 解压操作

#### 步骤

1. 创建`ZipFile`文件对象,标识`.zip`文件
2. 创建`ZipInputStream`数据流对象用于标识`.zip`文件的输入流
3. 循环读取`ZipEntry`， 拷贝此条目到解压后的文件中

#### 代码

```java
  /**
   * 解压zip文件
   *
   * @param zipFilepath zip文件路径
   * @param unzipFilepath 解压路径
   */
  public void unzip(String zipFilepath, String unzipFilepath) {

    // 待解压的文件目录
    File file = new File(zipFilepath);
    ZipInputStream zipInputStream = null;
    try {
      ZipFile zipFile = new ZipFile(file);
      zipInputStream = new ZipInputStream(new FileInputStream(file));
      ZipEntry entry = null;
      while ((entry = zipInputStream.getNextEntry()) != null) {
        InputStream inputStream = null;
        OutputStream outputStream = null;
        try {
          // 压缩文件内, 单条记录完整的文件夹名+文件名
          String entryName = entry.getName();
          System.out.println(entryName);

          File outFile = new File(unzipFilepath + File.separator + entryName);
          if (!outFile.getParentFile().exists()) {
            outFile.getParentFile().mkdirs();
          }

          // 如果是目录的话, 直接创建目录
          if (entry.isDirectory()) {
            outFile.mkdirs();
            continue;
          }

          if (!outFile.exists()) {
            outFile.createNewFile();
          }

          // 拷贝文件内容
          copy(zipFile.getInputStream(entry), new FileOutputStream(outFile));
        } finally {
          if (outputStream != null) {
            outputStream.close();
          }
          if (inputStream != null) {
            inputStream.close();
          }
        }
      }
    } catch (IOException e) {
      throw new RuntimeException(e);
    } finally {
      // 关闭zip文件流
      if (zipInputStream != null) {
        try {
          zipInputStream.close();
        } catch (IOException ignore) {
        }
      }
    }
  }

  /**
   * 从输入流拷贝数据到输出流
   *
   * @param src 输入流
   * @param dest 输出流
   */
  private void copy(InputStream src, OutputStream dest) throws IOException {
    int len = 0;
    byte[] buffer = new byte[4096];
    while ((len = src.read(buffer)) != -1) {
      dest.write(buffer, 0, len);
    }
  }

```

### 压缩操作

#### 步骤

1. 创建`ZipOutputStream`输出流对象
2. 判断当前读取的文件是文件还是目录
3. 如果是文件,那么使用当前文件的名称创建`ZipEntry`对象, 并将该文件拷贝到`ZipOutputStream`流中
4. 如果是目录， 则递归目录分别创建文件

#### 代码

```java
  /**
   * 压缩文件为zip
   *
   * @param srcFilepath 源文件或文件夹路径
   * @param destFilepath zip文件路径
   */
  public void toZip(String srcFilepath, String destFilepath) {
    File srcFile = new File(srcFilepath);
    if (!srcFile.exists()) {
      throw new IllegalArgumentException("被压缩的文件或文件夹不存在");
    }
    // 如果目标文件不存在,创建文件
    File zipFile = new File(destFilepath);
    if (zipFile.exists()) {
      zipFile.delete();
    }
    if (zipFile.getParentFile().exists()) {
      zipFile.getParentFile().mkdirs();
    }
    ZipOutputStream outputStream = null;
    try {
      zipFile.createNewFile();
      outputStream = new ZipOutputStream(new FileOutputStream(zipFile));
      compress(outputStream, srcFile, "");
    } catch (IOException e) {
      throw new RuntimeException(e);
    } finally {
      try {
        outputStream.close();
      } catch (IOException ignore) {
      }
    }
  }

  /**
   * 递归压缩文件
   *
   * @param outputStream zip文件输出流
   * @param srcFile 被压缩的文件
   * @param filenamePrefix zip文件内部的条目前缀
   */
  private void compress(ZipOutputStream outputStream, File srcFile, String filenamePrefix)
      throws IOException {
    if (srcFile.isDirectory()) {
      File[] files = srcFile.listFiles();
      // 先写一个目录
      filenamePrefix = filenamePrefix.length() == 0 ? srcFile.getName()
          : filenamePrefix + srcFile.getName();
      if (files == null) {
        return;
      }
      // 递归写入所有存在的文件
      for (File file : files) {
        compress(outputStream, file, filenamePrefix + "/");
      }
    } else {
      // 将单个文件写入到zip中
      outputStream.putNextEntry(new ZipEntry(filenamePrefix + srcFile.getName()));
      try (InputStream inputStream = new FileInputStream(srcFile)) {
        copy(inputStream, outputStream);
      }
    }
  }
```

