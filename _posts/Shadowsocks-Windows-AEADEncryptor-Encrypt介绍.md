---
title: 'Shadowsocks-Windows-AEADEncryptor#Encrypt介绍'
abbrlink: f322
date: 2018-06-18 20:24:39
tags:
	- Shadowsocks
---


最近自己在写代理软件，参考了Shadowsocks-python，Shadowsocks-Windows, v2ray的代码，正好先把C#版本的代码整理上来，虽然写得有点乱:)
<!-- more -->

```csharp
public override void Encrypt(byte[] buf, int length, byte[] outbuf, out int outlength)
        {
            Debug.Assert(_encCircularBuffer != null, "_encCircularBuffer != null");

            _encCircularBuffer.Put(buf, 0, length);//存放到环形缓冲区
            outlength = 0;
            Logging.Debug("---Start Encryption");
            if (! _encryptSaltSent) {
                _encryptSaltSent = true;
                // Generate salt
                byte[] saltBytes = new byte[saltLen];
                randBytes(saltBytes, saltLen);
                InitCipher(saltBytes, true, false);
                Array.Copy(saltBytes, 0, outbuf, 0, saltLen);
                outlength = saltLen;
                Logging.Debug($"_encryptSaltSent outlength {outlength}");
            }

            if (! _tcpRequestSent) {
                _tcpRequestSent = true;
                // The first TCP request
                int encAddrBufLength;
                byte[] encAddrBufBytes = new byte[AddrBufLength + tagLen * 2 + CHUNK_LEN_BYTES];
                
                //先将请求头加密
                byte[] addrBytes = _encCircularBuffer.Get(AddrBufLength);
                ChunkEncrypt(addrBytes, AddrBufLength, encAddrBufBytes, out encAddrBufLength);

                Debug.Assert(encAddrBufLength == AddrBufLength + tagLen * 2 + CHUNK_LEN_BYTES);
                Array.Copy(encAddrBufBytes, 0, outbuf, outlength, encAddrBufLength);
                outlength += encAddrBufLength;
                Logging.Debug($"_tcpRequestSent outlength {outlength}");
            }

            // handle other chunks
			//处理TCP payload。
            while (true) {
                uint bufSize = (uint)_encCircularBuffer.Size;
                if (bufSize <= 0) return;

                //限制数据包长度
                var chunklength = (int)Math.Min(bufSize, CHUNK_LEN_MASK);
                byte[] chunkBytes = _encCircularBuffer.Get(chunklength);
                int encChunkLength;
                byte[] encChunkBytes = new byte[chunklength + tagLen * 2 + CHUNK_LEN_BYTES];
                ChunkEncrypt(chunkBytes, chunklength, encChunkBytes, out encChunkLength);
                Debug.Assert(encChunkLength == chunklength + tagLen * 2 + CHUNK_LEN_BYTES);
                Buffer.BlockCopy(encChunkBytes, 0, outbuf, outlength, encChunkLength);
                outlength += encChunkLength;
                Logging.Debug("chunks enc outlength " + outlength);
                // check if we have enough space for outbuf
                if (outlength + TCPHandler.ChunkOverheadSize > TCPHandler.BufferSize) {
                    Logging.Debug("enc outbuf almost full, giving up");
                    return;
                }
                bufSize = (uint)_encCircularBuffer.Size;
                if (bufSize <= 0) {
                    Logging.Debug("No more data to encrypt, leaving");
                    return;
                }
            }
        }
```
_encCircularBuffer是一个环形缓冲区，所有的数据都先存进去然后等到满足了数据长度之后通过ByteCircularBuffer#get()来获取。
对于第一个加密请求包，开头的32字节是salt，也即saltLen == 32

注意这个`byte[] encAddrBufBytes = new byte[AddrBufLength + tagLen * 2 + CHUNK_LEN_BYTES];`
AddrBufLength是Socks5请求头中的ATYP + dst + port的长度，加上AEAD的两个认证tag（各16字节） + 两字节的LengthField。

```csharp
private void ChunkEncrypt(byte[] plaintext, int plainLen, byte[] ciphertext, out int cipherLen)
        {
            if (plainLen > CHUNK_LEN_MASK) {
                Logging.Error("enc chunk too big");
                throw new CryptoErrorException();
            }

            // encrypt len
            //加密长度，2 bytes的长度加上16字节的Auth tag
            byte[] encLenBytes = new byte[CHUNK_LEN_BYTES + tagLen];
            uint encChunkLenLength = 0;
            byte[] lenbuf = BitConverter.GetBytes((ushort) IPAddress.HostToNetworkOrder((short)plainLen));

            /*abstract函数，由AEADMbedTLSEncryptor
             *AEADOpenSSLEncryptor, AEADSodiumEncryptor具体实现
             */
            cipherEncrypt(lenbuf, CHUNK_LEN_BYTES, encLenBytes, ref encChunkLenLength);

            Debug.Assert(encChunkLenLength == CHUNK_LEN_BYTES + tagLen);
            IncrementNonce(true);

            // encrypt corresponding data
			//这里加密Socks请求头ATYP + dst + port
            byte[] encBytes = new byte[plainLen + tagLen];
            uint encBufLength = 0;
            cipherEncrypt(plaintext, (uint) plainLen, encBytes, ref encBufLength);
            Debug.Assert(encBufLength == plainLen + tagLen);
            IncrementNonce(true);

            // construct outbuf
            Array.Copy(encLenBytes, 0, ciphertext, 0, (int) encChunkLenLength);
            Buffer.BlockCopy(encBytes, 0, ciphertext, (int) encChunkLenLength, (int) encBufLength);
            cipherLen = (int) (encChunkLenLength + encBufLength);
			//最后将加密之后的数据都复制到`encChunkLenLength`
        }


```

准备开始加密，这时：

outbuf
```
+----------------+------------+
| salt 32        |空数据      |
```

调用`ChunkEncrypt`加密之后`encAddrBufBytes`中的内容：（当然，加密之后的数据不可见，但为了简单起见，这里还是直接说是SocksHeader之类的词）
```
+---------------+--------+----------------+--------+
| lengthField 2 | tag 16 |  socks header  | tag 16 |
```

将`encAddrBufBytes`内容复制到`outbuf`之后:
```
+---------+---------------+--------+----------------+--------+
| salt 32 | lengthField 2 | tag 16 |  socks header  | tag 16 |

```
然后就会进入`Encrypt``中的`while`循环继续处理ByteCircularBuffer中的剩余数据，最后要保证总体数据长度不超过`CHUNK_LEN_MASK(0x3FFFu)`

最后
```
+---------+---------------+--------+----------------+--------+------------+-----------+
| salt 32 | lengthField 2 | tag 16 |  socks header  | tag 16 |TCP payload | tag   16  |
```
当Socks header发送到Server之后，在认证完之后协议格式变简单了:
```
+---------------+--------|------------+-----------+
| lengthField 2 | tag 16 |TCP payload | tag   16  |
```
这里开始就无限转发数据了:)

AEADEncryptEncrypt#Encrypt()的函数调用栈
![AEADEncryptor#Encrypt Callback stack](https://chaochaogege.net/images/AEADEncryptor.png)

举个例子	
![一个例子](https://chaochaogege.net/images/AEADEncryptor-Encrypt-SocksHeader.png)
第一个字节0x03表示域名，后面的14个字节代表`api.github.com``，1,187分别是十六进制的0x01,0xbb代表443端口，[18]代表的则是TLS握手的建立连接了字段了。[18]以及[18]之后的都是TCP payload.

在`ChunkEncrypt`之中
encBytes的长度34 == socksHeader(18) + auth tag(16)
![](https://chaochaogege.net/images/ChunkEncrypt.png)


觉着写得挺乱的，还不如直接comments都写到代码上