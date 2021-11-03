# FTPClient中被人忽略的fileType

我们在使用ftpclient时，可能注意到了主动被动模式，然而可能会忽略fileType这个成员变量，默认不设置的话，ftp的fileType值为`ASCII_FILE_TYPE`，这样，在上传二进制文件的时候，会出现上传后的文件和实际文件有区别的情况，需设置为`BINARY_FILE_TYPE`。
# 现象
上传一段音频至ftp，播放下载下来的音频后，杂音较多，而原文件无杂音，于是分析了下ftpClient的代码。
# 分析过程

1. 运行以下demo上传文件
```java
public static void main(String[] args) throws SocketException, IOException {
        FileInputStream is = new FileInputStream("F:/1.mp3");
        FTPClient ftpClient = new FTPClient();
        ftpClient.setControlEncoding("UTF-8");
        ftpClient.connect("127.0.0.1", 21);
        ftpClient.login("root", "123456");
        //ftpClient.setFileType(FTP.BINARY_FILE_TYPE); 设置fileType为二进制文件类型
        ftpClient.storeFile("/2.mp3", is);
        is.close();
        ftpClient.disconnect();
    }
```

2. 以上代码第7行注释，未设置fileType，使用默认值`ASCII_FILE_TYPE`上传文件后，原文件1725KB，上传后文件大小变为1732KB。
2. 查看ftpClient源码，上传文件的方法，可以看到，第13行代码中，有特殊的逻辑判断，使用了`ToNetASCIIOutputStream`类进行处理。
```java
private boolean __storeFile(int command, String remote, InputStream local)
    throws IOException
    {
        OutputStream output;
        Socket socket;

        if ((socket = _openDataConnection_(command, remote)) == null)
            return false;

        output = new BufferedOutputStream(socket.getOutputStream(),
                                          getBufferSize()
                                          );
        if (__fileType == ASCII_FILE_TYPE)
            output = new ToNetASCIIOutputStream(output);
        // Treat everything else as binary for now
        try
        {
            Util.copyStream(local, output, getBufferSize(),
                            CopyStreamEvent.UNKNOWN_STREAM_SIZE, null,
                            false);
        }
        catch (IOException e)
        {
            try
            {
                socket.close();
            }
            catch (IOException f)
            {}
            throw e;
        }
        output.close();
        socket.close();
        return completePendingCommand();
    }
```

4. 下面看3中提到的类的源码，可以看到实现的write方法中，可以看到，如果文件中只出现了\n而不是\r\n同时出现，则存储时，会被自动替换为\r\n，这就导致存储后的文件与源文件不一致。
```java
/*
 * Copyright 2001-2005 The Apache Software Foundation
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package org.apache.commons.net.io;

import java.io.FilterOutputStream;
import java.io.IOException;
import java.io.OutputStream;

/***
 * This class wraps an output stream, replacing all singly occurring
 * &lt;LF&gt; (linefeed) characters with &lt;CR&gt;&lt;LF&gt; (carriage return
 * followed by linefeed), which is the NETASCII standard for representing
 * a newline.
 * You would use this class to implement ASCII file transfers requiring
 * conversion to NETASCII.
 * <p>
 * <p>
 * @author Daniel F. Savarese
 ***/

public final class ToNetASCIIOutputStream extends FilterOutputStream
{
    private boolean __lastWasCR;

    /***
     * Creates a ToNetASCIIOutputStream instance that wraps an existing
     * OutputStream.
     * <p>
     * @param output  The OutputStream to wrap.
     ***/
    public ToNetASCIIOutputStream(OutputStream output)
    {
        super(output);
        __lastWasCR = false;
    }


    /***
     * Writes a byte to the stream.    Note that a call to this method
     * may result in multiple writes to the underlying input stream in order
     * to convert naked newlines to NETASCII line separators.
     * This is transparent to the programmer and is only mentioned for
     * completeness.
     * <p>
     * @param ch The byte to write.
     * @exception IOException If an error occurs while writing to the underlying
     *            stream.
     ***/
    public synchronized void write(int ch)
    throws IOException
    {
        switch (ch)
        {
        case '\r':
            __lastWasCR = true;
            out.write('\r');
            return ;
        case '\n':
            if (!__lastWasCR)
                out.write('\r');
            // Fall through
        default:
            __lastWasCR = false;
            out.write(ch);
            return ;
        }
    }


    /***
     * Writes a byte array to the stream.
     * <p>
     * @param buffer  The byte array to write.
     * @exception IOException If an error occurs while writing to the underlying
     *            stream.
     ***/
    public synchronized void write(byte buffer[])
    throws IOException
    {
        write(buffer, 0, buffer.length);
    }


    /***
     * Writes a number of bytes from a byte array to the stream starting from
     * a given offset.
     * <p>
     * @param buffer  The byte array to write.
     * @param offset  The offset into the array at which to start copying data.
     * @param length  The number of bytes to write.
     * @exception IOException If an error occurs while writing to the underlying
     *            stream.
     ***/
    public synchronized void write(byte buffer[], int offset, int length)
    throws IOException
    {
        while (length-- > 0)
            write(buffer[offset++]);
    }

}

```

5. 下面看上传前后的文件对比图，16进制显示文件内容，可以看到，就是上面分析的结果。

![image.png](https://cdn.nlark.com/yuque/0/2020/png/2070000/1596984051139-d9cd965a-38ca-460b-99dd-50517adcb402.png#align=left&display=inline&height=584&margin=%5Bobject%20Object%5D&name=image.png&originHeight=584&originWidth=1019&size=81207&status=done&style=none&width=1019)

6. 由于文件存储时，被添加了新的字符，导致此音频文件播放的时候，解码声音时出现了脏数据，导致出现了杂音。
# 结论
如果上传ftp中的文件不是文本类型的，添加代码`ftpClient.setFileType(FTP.BINARY_FILE_TYPE)`，保证上传文件与原文件相同。另外，这并不是ftpClient的缺陷，这是为了兼容linux，windows等不同平台设计的。
