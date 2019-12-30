## 使用SVN进行操作

### 依赖

```xml
<dependency>
    <groupId>org.tmatesoft.svnkit</groupId>
    <artifactId>svnkit</artifactId>
    <version>1.10.1</version>
</dependency>
```

### 创建与释放客户端管理器

```java
  /**
   * 根据用户名和密码创建SVN客户端管理器
   *
   * @param username 用户名
   * @param password 密码
   * @return 客户端管理器
   */
  public static SVNClientManager getSVNClientManager(String username, String password) {
    if (StringUtil.isBlank(username)) {
      throw new IllegalArgumentException("SVN username must not null");
    }

    DefaultSVNOptions options = new DefaultSVNOptions();
    SVNClientManager svnClientManager = SVNClientManager
        .newInstance(options, username, password);
    svnClientManager.setIgnoreExternals(false);
    return svnClientManager;
  }
```

```java
 /**
   * 释放SVN客户端连接
   *
   * @param svnClientManager svn 客户端管理器
   */
  public static void releaseSVNClientManager(SVNClientManager svnClientManager) {
    if (ObjectUtil.isNull(svnClientManager)) {
      return;
    }
    svnClientManager.dispose();
  }

```

### checkout

```java
  /**
   * checkout 文件夹
   *
   * @param svnClientManager svn客户端管理器 {{@link #getSVNClientManager(String, String)}}
   * @param svnUrl svn地址
   * @param directorPath 本地保存路径
   * @return 版本号
   */
  public static long checkout(SVNClientManager svnClientManager, String svnUrl,
      String directorPath) {
    File dir = new File(directorPath);
    IoUtil.createDirector(dir);
    SVNUpdateClient updateClient = svnClientManager.getUpdateClient();
    try {
      return updateClient
          .doCheckout(SVNURL.parseURIEncoded(svnUrl), dir, SVNRevision.HEAD, SVNRevision.HEAD,
              SVNDepth.INFINITY, true);
    } catch (SVNException e) {
      throw new RuntimeException(e);
    }
  }

```

### update

```java
  /**
   * 更新
   *
   * @param svnClientManager svn客户端管理器
   * @param filepath 文件路径
   * @return 文件版本号
   */
  public static long update(SVNClientManager svnClientManager, String filepath) {
    if (StringUtil.isBlank(filepath)) {
      throw new IllegalArgumentException("update file must not null");
    }
    return update(svnClientManager, new File(filepath));
  }

  /**
   * 更新
   *
   * @param svnClientManager svn客户端管理器
   * @param file 需更新的文件
   * @return 文件版本号
   */
  public static long update(SVNClientManager svnClientManager, File file) {
    if (ObjectUtil.isNull(file)) {
      throw new NullPointerException("update file must not null");
    }
    File[] files = new File[1];
    files[0] = file;
    return update(svnClientManager, files)[0];
  }

  /**
   * 更新
   *
   * @param svnClientManager svn客户端管理器
   * @param filesets 更新的文件集合
   * @return 文件版本号列表
   */
  public static long[] update(SVNClientManager svnClientManager, String[] filesets) {
    if (CollectionUtil.isEmpty(filesets)) {
      throw new IllegalArgumentException("update files must has one or more");
    }
    File[] files = new File[filesets.length];
    for (int i = 0; i < filesets.length; i++) {
      files[i] = new File(filesets[i]);
    }
    return update(svnClientManager, files);
  }

  /**
   * 更新文件
   *
   * @param svnClientManager svn客户端管理器
   * @param files 需要更新的文件列表
   * @return 文件版本号列表
   */
  public static long[] update(SVNClientManager svnClientManager, File[] files) {
    if (CollectionUtil.isEmpty(files)) {
      throw new IllegalArgumentException("update files must has one or more");
    }
    if (ObjectUtil.isNull(svnClientManager)) {
      throw new NullPointerException("SVNClientManager must not null");
    }
    SVNUpdateClient updateClient = svnClientManager.getUpdateClient();
    try {
      return updateClient.doUpdate(files, SVNRevision.HEAD, SVNDepth.INFINITY, true, false);
    } catch (SVNException e) {
      throw new RuntimeException(e);
    }
  }
```

### commit

```java
  /**
   * 提交
   *
   * @param svnClientManager svn客户端管理器
   * @param filepath 提交的文件
   * @param commitMessage 提交的文本描述
   * @return 新的版本号, -1:没有数据提交
   */
  public static long commit(SVNClientManager svnClientManager, String filepath,
      String commitMessage) {
    if (StringUtil.isBlank(filepath)) {
      throw new IllegalArgumentException("commit file must not null");
    }
    return commit(svnClientManager, new File(filepath), commitMessage);
  }

  /**
   * 提交
   *
   * @param svnClientManager svn客户端管理器
   * @param file 提交的文件
   * @param commitMessage 提交的文本描述
   * @return 新的版本号, -1:没有数据提交
   */
  public static long commit(SVNClientManager svnClientManager, File file, String commitMessage) {
    if (ObjectUtil.isNull(file)) {
      throw new IllegalArgumentException("commit file must not null");
    }
    File[] files = new File[1];
    files[0] = file;
    return commit(svnClientManager, files, commitMessage);
  }

  /**
   * 提交
   *
   * @param svnClientManager svn客户端管理器
   * @param paths 提交的路径
   * @param commitMessage 提交的文本描述
   * @return 新的版本号, -1:没有数据提交
   */
  public static long commit(SVNClientManager svnClientManager, File[] paths, String commitMessage) {
    if (ObjectUtil.isNull(svnClientManager)) {
      throw new NullPointerException("SVNClientManager must not null");
    }
    if (CollectionUtil.isEmpty(paths)) {
      throw new IllegalArgumentException("commit file set must not null");
    }
    SVNCommitClient commitClient = svnClientManager.getCommitClient();
    try {
      SVNCommitInfo svnCommitInfo = commitClient
          .doCommit(paths, false, commitMessage, null, null, false, true, SVNDepth.INFINITY);
      return svnCommitInfo.getNewRevision();
    } catch (SVNException e) {
      throw new RuntimeException(e);
    }
  }
```

### 使用方式

```java
SVNClientManager svnClientManager = null;
try {
    svnClientManager = getSVNClientManager("myUsername", "myPassword");
    // checkout
    long checkoutVersion = checkout(svnClientManager, "http://ip:port/url", "C://dir");
    // update
    long updateVersion = update(svnClientManager, "path to update file");
    // commit
    long commitVersion = commit(svnClientManager, "path to commit file or director", "commit message");
} finally {
    releaseSVNClientManager(svnClientManager);
}
```

