---
title: java utils files
date: 2021-05-10 15:45:25
banner_img: /img/basic/java-binner.jpg
index_img: /img/basic/java-index.jpg
tags: 
 - apache
categories:
 - basic-component
 - apache
---

# File Utils

| write(final File file, final CharSequence data, final Charset encoding)write(final File file, final CharSequence data, final Charset encoding, final boolean append) write(final File file, final CharSequence data, final String encoding)write(final File file, final CharSequence data, final String encoding, final boolean append) | 将字符序列写入到文件file：待写入的文件，路径不存在时，自动新建；data：写入的字节内容；append：是否追加；encoding：文件内容编码，这在网络通信时防止中文乱码非常有用。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| writeByteArrayToFile(final File file, final byte[] data)writeByteArrayToFile(final File file, final byte[] data, final boolean append)writeByteArrayToFile(final File file, final byte[] data, final int off, final int len)writeByteArrayToFile(final File file, final byte[] data, final int off, final int len,final boolean append) | 将字节数组写入到文件file：待写入的文件，路径不存在时，自动新建；data：写入的字节内容；append：是否追加；off：字节数组写入的起始位置；lem：写入的长度 |
| writeLines(final File file, final Collection<?> lines)writeLines(final File file, final Collection<?> lines, final boolean append)writeLines(final File file, final Collection<?> lines, final String lineEnding)writeLines(final File file, final Collection<?> lines, final String lineEnding,final boolean append)writeLines(final File file, final String encoding, final Collection<?> lines)writeLines(final File file, final String encoding, final Collection<?> lines,final boolean append)writeLines(final File file, final String encoding, final Collection<?> lines,final String lineEnding)writeLines(final File file, final String encoding, final Collection<?> lines,final String lineEnding, final boolean append) | 将集合中的内容一次性写入文件file：待写入的文件，路径不存在时，自动新建；lines：要写入的行，null 表示插入空行append：内容是否追加lineEnding：要使用的行分隔符，null 表示使用系统默认值encoding：要使用的编码，{@code null}表示平台默认值 |
| writeStringToFile(final File file, final String data, final Charset encoding)writeStringToFile(final File file, final String data, final Charset encoding,final boolean append)writeStringToFile(final File file, final String data, final String encoding)writeStringToFile(final File file, final String data, final String encoding,final boolean append) | 将字符串写入到文件file：待写入的文件，路径不存在时，自动新建；data：写入的字节内容；append：是否追加；encoding：文件内容编码 |
| byte[] readFileToByteArray(final File file)                  | 将文件内容读入字节数组。文件始终处于关闭状态。               |
| String readFileToString(final File file, final String encoding)String readFileToString(final File file, final Charset encoding)String readFileToString(final File file) | 将文件的内容读入字符串。文件始终处于关闭状态。 file：要读取的文件，不能是 nullencoding:要使用的编码，null表示使用平台默认值 |
| List<String> readLines(final File file, final Charset encoding)List<String> readLines(final File file, final String encoding) | 逐行读取文件内容到字符串列表。文件始终处于关闭状态。 file：要读取的文件，不能是 nullencoding:要使用的编码，null表示使用平台默认值 |
| LineIterator lineIterator(final File file)LineIterator lineIterator(final File file, final String encoding) | 为文件打开一个 InputStream 的行迭代器，完成迭代器之后，应该关闭流以释放内部资源。LineIterator implements Iterator：可以很方便的一行一行读取文件内容 |
| copyDirectory(final File srcDir, final File destDir)copyDirectory(final File srcDir, final File destDir,final boolean preserveFileDate)copyDirectory(final File srcDir, final File destDir,final FileFilter filter)copyDirectory(final File srcDir, final File destDir,final FileFilter filter, final boolean preserveFileDate) | 将指定目录下所有子孙目录和文件复制到指定的目标目录下，如果目标目录不存在，则创建该目录。如果目标目录确实存在，则此方法将源目录与目标目录合并，源目录优先。注意只能是目录，如果是文件则异常。srcDir：源目录，不能为 nulldestDir：目标目录，不能为 nullpreserveFileDate：副本的文件日期是否与原件相同filter：要应用的筛选器，即可以细粒度的控制复制哪些文件或者目录，null 表示复制所有目录和文件 |
| copyFile(final File srcFile, final File destFile)copyFile(final File srcFile, final File destFile,final boolean preserveFileDate)copyFile(final File input, final OutputStream output) | 此方法将指定源文件的内容复制到指定的目标文件，如果目标文件不存在，则会创建包含目标文件的目录，如果目标文件存在，则此方法将覆盖它。srcFile：源文件，不能为 nulldestFile：目标文件，，不能为 nullpreserveFileDate：副本的文件日期是否与原件相同output：将文件复制到字节输出流，方法在内部使用缓冲输入。 |
| copyFileToDirectory(final File srcFile, final File destDir)copyFileToDirectory(final File srcFile, final File destDir, final boolean preserveFileDate) | 将指定源文件的内容复制到指定目标目录中同名的文件中。如果目标目录不存在，则创建该目录。如果目标文件存在，则此方法将覆盖它。srcFile：源文件，不能为 nulldestDir：目标目录，不能为 nullpreserveFileDate：副本的文件日期是否与原件相同 |
| copyInputStreamToFile(final InputStream source, final File destination) | 将字节输入流复制到目标文件中，不存在时自动创建               |
| copyToDirectory(final File src, final File destDir)copyToDirectory(final Iterable<File> srcs, final File destDir) | 将源文件或目录及其所有内容复制到指定目标目录中同名的目录中。如果目标目录不存在，则创建该目录。如果目标目录确实存在，则此方法将源目录与目标目录合并，源目录优先。src：源文件或者目录destDir：目标目录，不能为 null |
| copyToFile(final InputStream source, final File destination) | 将字节输入流复制打目标文件中，如果目标目录不存在，则将创建该目录。如果目标已存在，则将覆盖该目标。 |
| copyURLToFile(final URL source, final File destination)copyURLToFile(final URL source, final File destination,final int connectionTimeout, final int readTimeout) | 将 URL 网络资源复制到目标文件中，可以用于下载，未设置超时时间时，可能出现永久阻塞connectionTimeout：连接超时时间，单位毫秒readTimeout：读取超时时间，单位毫秒 |
| deleteDirectory(final File directory)                        | 递归删除目录。注意只能是目录，如果是文件，则异常。           |
| deleteQuietly(final File file)                               | 安全删除文件或者递归删除目录，不会抛出任何异常。             |
| forceDelete(final File file)                                 | 强制删除文件或者递归删除目录                                 |
| forceMkdir(final File directory)                             | 生成一个目录，包括任何必需但不存在的父目录。如果已存在具有指定名称的文件，但它不是目录，则会引发IOException。如果目录无法创建（或不存在），则抛出IOException。 |
| moveDirectory(final File srcDir, final File destDir)         | 移动目录。当目标目录在另一个文件系统上时，执行“复制并删除”。srcDi r要移动的目录 destDir 目标目录 |
| moveDirectoryToDirectory(final File src, final File destDir, final boolean createDestDir) | 将目录移动到另一个目录。*将目录移动到另一个目录。*@param src要移动的文件*@param destDir目标文件*@param createDestDir如果为true，则创建目标目录，否则如果为false，则抛出IOException |
| moveFile(final File srcFile, final File destFile)            | 移动文件。当目标文件位于另一个文件系统上时，请执行“复制并删除”。 |
| moveFileToDirectory(final File srcFile, final File destDir, final boolean createDestDir) | 将文件移动到目录。                                           |
| moveToDirectory(final File src, final File destDir, final boolean createDestDir) | 将文件或目录移动到目标目录。*当目标位于另一个文件系统上时，请执行“复制并删除”。 |
| forceMkdirParent(final File file)                            | 为给定文件生成任何必需但不存在的父目录。如果无法创建父目录，则引发IOException。 |
| File getFile(final File directory, final String... names)File getFile(final String... names) | 获取文件对象directory：父目录names：子孙目录名称             |
| File getTempDirectory()String getTempDirectoryPath()         | 返回系统临时目录。底层就是 System.getProperty("java.io.tmpdir") |
| File getUserDirectory()String getUserDirectoryPath()         | 返回用户的主目录，底层就是 System.getProperty("user.home")   |
| boolean isFileNewer(final File file, final Date date)boolean isFileNewer(final File file, final File reference)boolean isFileNewer(final File file, final long timeMillis) | 测试指定文件的最后修改时间是否在指定时间之后，底层是 file.lastModified() > timeMillis |
| boolean isFileNewer(final File file, final Date date)boolean isFileNewer(final File file, final File reference)boolean isFileNewer(final File file, final long timeMillis) | 测试指定文件的最后修改时间是否在指定时间之前，底层是 file.lastModified() < timeMillis |
| long sizeOf(final File file)BigInteger sizeOfAsBigInteger(final File file) | 返回指定文件或目录的大小。如果提供的{@link File}是一个常规文件，则返回该文件的长度。如果参数是目录，则递归计算目录的大小。如果某个目录或子目录受到安全限制，则不会包括其大小。请注意，不会检测到溢出，如果发生溢出，则返回值可能为负。 |
| long sizeOfDirectory(final File directory)BigInteger sizeOfDirectoryAsBigInteger(final File directory) | 递归计算目录的大小（所有文件的长度之和）。sizeOfDirectory 只统计目录的大小，单位为 字节，如果是文件则报错。 |
| String byteCountToDisplaySize(final long size)String byteCountToDisplaySize(final BigInteger size) | 将文件字节大小转为可视化的 KB、MB、GB 等形式的字符串，一共有：bytes、KB、MB、GB、TB、PB、EB. |
| boolean contentEquals(final File file1, final File file2)    | 比较两个文件的内容以确定它们是否相等。（注意只能是文件，如果是目录，则异常）此方法检查两个文件的长度是否不同，或者它们是否指向同一个文件，然后对内容进行逐字节比较。如果文件的内容相等或两者都不存在，则为true；否则为false |
| File[] convertFileCollectionToFileArray(final Collection<File> files) | 将文件集合转为文件数组，底层就是:files.toArray(new File[files.size()]); |
| Collection<File> listFiles(final File directory, final String[] extensions, final boolean recursive)Collection<File> listFiles(final File directory, final IOFileFilter fileFilter, final IOFileFilter dirFilter) | 查找给定目录（及其子目录）中与扩展名数组匹配的文件。 directory 要搜索的目录 extensions：要过滤的扩展数组，例如{“java”、“xml”}，为 null，返回所有文件。 ecursive：true 表示搜索所有子目录fileFilter：文件过滤器，IOFileFilter 是一个接口，常用的实现类有：SuffixFileFilter(文件后缀过滤器)、PrefixFileFilter(文件前缀过滤器)、TrueFileFilter(总是返回true的文件过滤器)、FalseFileFilter(总是返回false的文件过滤器)dirFilter：与上面同理 |
| FileInputStream openInputStream(final File file)             | 为指定文件打开 FileInputStream，提供比简单调用new FileInputStream（file更好的错误消息。在方法结束时，要么成功打开流，要么抛出异常。*如果文件不存在，则引发异常。*如果文件对象存在但是一个目录，则引发异常。*如果文件存在但无法读取，则引发异常。 |
| FileOutputStream openOutputStream(final File file)FileOutputStream openOutputStream(final File file, final boolean append) | 打开指定文件的{@link FileOutputStream}，检查并创建父目录（如果不存在）。在方法结束时，要么成功打开流，要么抛出异常。*如果父目录不存在，将创建它。*如果文件不存在，将创建该文件。*如果文件对象存在但是一个目录，则引发异常。*如果文件存在但无法写入，则引发异常。*如果无法创建父目录，则引发异常。 |
| File toFile(final URL url)File[] toFiles(final URL[] urls)   | 将 URL 转为 File 对象，注意只能对本地文件生成的 URL 才有效，对网络上的 URL 直接返回 null. |
| URL[] toURLs(final File[] files)                             | 将 File 对象转为 URL 对象。                                  |