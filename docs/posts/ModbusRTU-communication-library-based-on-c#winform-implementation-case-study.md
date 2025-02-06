---
date:
  created: 2024-02-01
tags:
  - Modbus
authors: [Tenax]
description: >
  Learning ModbusRTU communication library implemented in c#winform.
---

# 基于 c#winform 实现的 ModbusRTU 通信库案例学习

<!-- more -->

## 通用读取

用 01H 功能码读取输出线圈（01 Coil Status/0x ）举例

点击读取按钮 btn_Read_Click

开始 if 条件判断，进入 CommonVerify 函数
CommonVerify 函数中会检查是否连接正常和站地址、起始地址、长度格式配置是否符合要求

判断成功 true 后进入 if 语句，

1. 首先将站地址、起始地址、长度从对应的文本框中提取出来，去除空格，并转换成对应的`byte`和`short`类型;再将数据类型和存储区、数据格式提取出来，转换成对应`Enum`类型中的类型
2. 进入 swtich 语句，根据对应的数据类型（dataType）进入 case（各种针对不同类型数据的读取方法）。若不支持的则 Addlog

进入这个读取方法，在这个读取方法中，

1. 我们首先会创建一个`byte`类型的叫做 result 的数组
2. 进入 switch 语句通过传来的存储区值判断进入那个 case（针对各种输入输出线圈寄存器的读取和写入的方法）。若不支持的则 Addlog

进入方法

---

<!--拼接报文-->

首先创建一个 SendCommand 列表（泛型集合）用来储存的是`byte`类型的元素；

将功能码、线圈地址、线圈数量、CRC 添加到 SendCommand 列表中

**CRC16**

```c#
        private byte[] CRC16(byte[] pucFrame, int usLen)
        {
            int i = 0;
            byte[] res = new byte[2] { 0xFF, 0xFF };
            ushort iIndex;
            while (i < usLen)
            {
                iIndex = (ushort)(res[0] ^ pucFrame[i++]);
                res[0] = (byte)(res[1] ^ aucCRCHi[iIndex]);
                res[1] = aucCRCLo[iIndex];
            }
            return res;
        }
```

首次进入 CRC16 校验方法中，i=0，再创建一个叫 res 的`byte`类型的数组`[0xFF，0xFF]`

每当 i<uslen，

`iIndex = (ushort)(res[0] ^ pucFrame[i++]);`将`res[0]`和`pucFrame[i++]`**按位异或**
例如：`res[0] = 0xFF` 和 `pucFrame[0] = 0x01` 按位异或

```yaml
0xFF = 1111 1111
0x01 = 0000 0001
-------------------
结果 = 1111 1110  = 0xFE
```

`res[0] = (byte)(res[1] ^ aucCRCHi[iIndex]);`将`res[1]`和`aucCRCHi[iIndex]`按位异或

...

<!--发送并接收报文-->

首先创建一个叫 receive 的`byte`类型的数组

再将 length 转换成 byteLength 保存（这里的 length 转换成 byteLength 的意思是，要读取的线圈或者寄存器的长度转换成要来保存的字节长度）

进入 if 语句，判断条件是`SendAndReceive`函数，传入 SendCommand 列表和刚创建的`byte`类型的数组

```c#
        private bool SendAndReceive(byte[] send, ref byte[] receive)
        {
            //加锁
            hybirdLock.Enter();

            try
            {
                //发送报文
                this.serialPort.Write(send, 0, send.Length);

                //定义一个Buffer
                byte[] buffer = new byte[1024];

                //定义一个内存
                MemoryStream stream = new MemoryStream();

                //定义一个开始时间
                DateTime start = DateTime.Now;

                //这么处理的原因是为了防止一次性读不完整

                //循环读取缓冲区的数据，如果大于0，就读出来，放到内存里，如果等于0，说明就读完了

                //如果每次都读不到，要设置一个超时时间

                while (true)
                {
                    Thread.Sleep(10);

                    if (this.serialPort.BytesToRead > 0)
                    {
                        int count = this.serialPort.Read(buffer, 0, buffer.Length);

                        stream.Write(buffer, 0, count);
                    }
                    else
                    {
                        if (stream.Length > 0)
                        {
                            break;
                        }
                        else if ((DateTime.Now - start).TotalMilliseconds > this.ReceiveTimeOut)
                        {
                            return false;
                        }
                    }
                }

                receive = stream.ToArray();

                return true;
            }
            catch (Exception e)
            {
                return false;
            }
            finally
            {
                //解锁
                hybirdLock.Leave();
            }

        }
```

发送传来的 SendCommand 列表，将数据写入串口的 **输出缓冲区**，定义一个`buffer` 作为**临时缓冲区**大小有 1024 个字节，定义一个内存流，定义一个开始时间

进入 while 循环，让当前线程暂停 **10 毫秒**后进入 if 循环，

1. 判断`this.serialPort.BytesToRead > 0` ，`BytesToRead` 是串口对象 (`SerialPort`) 的属性，表示当前 **输入缓冲区中未读取的字节数**。`BytesToRead > 0` 表示缓冲区中有数据可读。
2. **读取数据到 `buffer`**：从串口输入缓冲区中读取最多 `buffer.Length` 字节（此处是 1024 字节）到 `buffer` 数组中，返回值保存在`count`中， 是实际读取的字节数
3. **将有效数据写入内存流**：`buffer` 是之前创建的临时缓存区，`count` 表示本次读取的实际数据长度，因此只将 `buffer` 中前 `count` 个字节写入内存流 (`MemoryStream`)。

当**输入缓冲区中没有未读取的字节数**进入 else，进入 if 循环，

判断内存流中的数据长度是否大于 0（说明**输入缓冲区中的字节数已经读取完**），符合则 break 跳出去循环

else if 循环：

- **触发条件**：
  1. 进入 `else` 分支：表示当前串口缓冲区 **没有数据可读**（`BytesToRead == 0`）。
  2. 进一步检查 **是否已超时**：从开始等待到当前时刻的时间差超过 `ReceiveTimeOut`。
- **结果**：若超时，直接返回 `false`，表示接收失败。

最后将写入到内存流的数据转换成数组保存在 receive 中

`catch (Exception e) { return false; }` 捕获所有异常：无论代码中抛出何种异常（如串口断开、数据格式错误等），都会被此 `catch` 块捕获。返回 `false`：发生异常时，方法直接返回 `false`，表示操作失败。

`finally { hybirdLock.Leave(); }` **始终执行**：无论是否发生异常，`finally` 块中的代码 **一定会执行**。

<!--验证报文-->

进入 if 循环，判断返回来的 receive 中的 CRC 是否正确，并且 receive 的长度是否等于 5+ byteLength（）（为什么是 5？假如接收报文的格式是从站地址+功能码+字节计数+数据+CRC，那 CRC 占 2 个，站地址+功能码+字节计数三个，数据就是 byteLength 的长度）

```c#
        private bool CheckCRC(byte[] value)
        {
            if(value == null) return false;

            if(value.Length <= 2) return false;

            int length = value.Length;
            byte[] buf = new byte[length - 2];
            Array.Copy(value, 0, buf, 0, buf.Length);

            byte[] CRCbuf = CRC16(buf, buf.Length);
            if (CRCbuf[0] == value[length - 2] && CRCbuf[1] == value[length - 1])
            {
                return true;
            }
            return false;
        }
```

简单的来说，这 CheckCRC 的方法就是把 receive 中的 CRC 重新验算了一遍，并判断是否正确

...

进入 if 循环，判断 receive 数组中的第 1 个是否是站地址，第 2 个是否是功能码，第 3 个是否是字节计数，

创建一个叫 result 的字节类型数组，大小为 byteLength，

并将 receive 中的数组开始索引为 3 拷贝长度为 byteLength 个长度到 result 中，成功后返回

---

进入 if 循环，如果返回的 result 不为 null 则

AddLog（**AddLog 中拼接字符串时用的转换方法是把字节数组转换成布尔数组，再把各种类型数组转换成字符串**），

```c#
        private void AddLog(int level, string info)
        {
            ListViewItem lst = new ListViewItem("  " + DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss"),level);

            lst.SubItems.Add(info);

            //让最新的数据在最上面
            this.lst_Info.Items.Insert(0, lst);
        }
```

这个 AddLog 方法简单来说就是将 imageList 中的图片和时间拼接出来放在 ListView 控件中

最后跳出 switc case 循环

## 通用写入

用 05H 功能码预置单线圈（01 Coil Status/0x ）举例

跟通用读取对比，大致的思路相同，只记录不同之处

```c#
            switch (storeArea)
            {
                case StoreArea.输出线圈0x:

                    bool[] values = BitLib.GetBitArrayFromBitArrayString(setValue);
                    if (values.Length == 1)
                    {
                        result = modbus.PreSetSingleCoil(devId, start, values[0]);
                    }
                    else
                    {
                        result = modbus.PreSetMultiCoils(devId, start, values);
                    }
                    break;
                case StoreArea.输入线圈1x:
                case StoreArea.输出寄存器4x:
                case StoreArea.输入寄存器3x:
                    AddLog(1,"写入失败，不支持该存储区");
                    return;
            }
```

在写入布尔的方法中`BitLib.GetBitArrayFromBitArrayString(setValue)`会将字符串按照指定的分隔符转换成布尔数组，将要写入的值转换成布尔值，如果要写入的值是一个就用写入单个线圈的方法，多个则用写入多个线圈的方法

进入`PreSetSingleCoil`的方法中，`SendCommand.Add(value?(byte)0xFF:(byte)0x00);`表示传进来的值如果是 true 就 0xFF 0x00 代表置位，如果不是就 0x00 0x00 代表复位

接下来是验证报文步骤里会有个`ByteArrayEquals`字节数组的对比方法

```c#
        private bool ByteArrayEquals(byte[] b1, byte[] b2)
        {
            if(b1 == null || b2 == null) return false;

            if (b1.Length != b2.Length) return false;

            for (int i = 0; i < b1.Length; i++)
            {
                if (b1[i] != b2[i])
                {
                    return false;
                }
            }
            return true;
        }
```

因为 05H 功能码预置单线圈中发送报文格式和接收报文的格式是相同的，所以我们要对比发送的报文和接收到的报文是否相同

## 通用写入-0FH 预置多线圈

预置多线圈说明需要写入多个值，所以使用`PreSetMultiCoils`预置多个线圈的方法

拼接报文的时候会用到`GetByteArrayFromBoolArray`这个方法将布尔数组转换成字节数组

```c#
        private byte[] GetByteArrayFromBoolArray(bool[] value)
        {
            int byteLength = value.Length%8==0 ? value.Length / 8 : value.Length / 8 + 1;

            byte[] result  = new byte[byteLength];

            for (int i = 0; i < result.Length; i++)
            {
                //获取每个字节的值

                int total = value.Length < 8 * (i + 1) ? value.Length - 8 * i : 8;

                for (int j = 0; j < total; j++)
                {
                    result[i] = SetBitValue(result[i], j, value[8 * i + j]);
                }
            }

            return result;
        }

        private byte SetBitValue(byte src, int bit, bool value)
        {
            return value ? (byte)(src | (byte)Math.Pow(2, bit)) : (byte)(src & ~(byte)Math.Pow(2, bit));
        }

```

首先计算转换成字节数组所以要的容量大小，

然后在遍历字节数组的大小，在遍历 bool 数组计算 value 的大小，

通过`SetBitValue`方法将某个字节某个位置置位或复位

当 `value == true`：

```c#
(byte)(src | (byte)Math.Pow(2, bit))
```

- `Math.Pow(2, bit)` 计算 `2^bit`，例如 `bit = 3` 时，`Math.Pow(2, 3) = 8`（即 `00001000`）。
- `(byte)Math.Pow(2, bit)` 变成 `byte` 类型，例如 `bit = 3` 时，它的二进制为 `00001000`。
- `src | (byte)Math.Pow(2, bit)` 用 **按位或（OR）** 操作，把 `bit` 位置 1。

示例：

```c#
src = 0b00000010  // 初始值
bit = 3           // 需要设置的位
// Math.Pow(2,3) 结果为 8，即 0b00001000
结果 = 0b00000010 | 0b00001000 = 0b00001010
```

当 `value == false`：

```c#
(byte)(src & ~(byte)Math.Pow(2, bit))
```

- `Math.Pow(2, bit)` 计算 `2^bit`，例如 `bit = 3` 时，它的值是 `8`，二进制 `00001000`。
- `~(byte)Math.Pow(2, bit)` 取反，比如 `00001000` 取反变成 `11110111`。
- `src & ~(byte)Math.Pow(2, bit)` 用 **按位与（AND）** 操作，把 `bit` 位置 0。

示例：

```c#
src = 0b00001010  // 初始值
bit = 3           // 需要清除的位
// ~(0b00001000) 结果是 0b11110111
结果 = 0b00001010 & 0b11110111 = 0b00000010
```
