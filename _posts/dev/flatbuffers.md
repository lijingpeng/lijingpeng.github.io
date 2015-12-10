title: FlatBuffers使用简介
date: 2015-11-20 10:01:56
tags: FlatBuffers
category: dev
---

### 概述
---

Google在今年6月份发布了跨平台序列化工具FlatBuffers，提供了C++/Java/Go/C#接口支持，这是一个注重性能和资源使用的序列化类库。相较于Protocol Buffers，其更适用于移动设备，FlatBuffers提供更高的性能以及更低的资源需求。

特点
- 不需要打包/解包。它的结构化数据都以二进制形式保存，不需要数据解析过程，数据也可以方便传递
- 省内存、性能好
- 强类型系统，在编译阶段就能预防一些bug的产生
- 跨平台（C++11/Java/Go/C#）

### FlatBuffers和Protocol Buffers以及Json的比较：
- FlatBuffers的功能和Protocol Buffers很像，他们的最大不同点是在使用具体的数据之前，FlatBuffers不需要解析/解包的过程。同时，在工程中使用时，FlatBuffers的引用比Protocol Buffers方便很多，只需要包含两三个头文件即可
- JSON作为数据交换格式，被广泛用户各种动态语言之间（当然也包括静态语言）。它的优点是易于理解（可读性好），同时它的最大的缺点那就是解析时的性能问题了。而且因为它的动态类型特点，你的代码可能还需要多写好多类型、数据检查逻辑。

### FlatBuffers的使用步骤
编写一个用来定义数据结构的schema(IDL，接口定义)文件，如下所示：

```FlatBuffers
  //flatbuffers test struct

  namespace Jason.Flat.Test;

  enum Color : byte { Red = 1, Green, Blue }

  union Any { TextureData, Texture }

  table TestAppend {
      test_num:int;
      test_num2:int;
  }

  table TextureData {
      image_size:int (id:0);
      image_data:[ubyte] (id:1);
      image_test:short(id:3);
      test_num2:int(id:2);
  }

  table Texture {
      num_textures:short;
      textures:[TextureData];
      num_test:short = 30;
      num_test1:short (deprecated);
          num_test2:short;
      test_append:TestAppend;
  }

  root_type Texture;

```
将上述代码保存为TestFlat.fbs文件之后，即可用flatc来编译了

- 使用FlatBuffer编译器flatc生成数据结构源代码（C++头文件或者Java类）
登录GitHub，下载所需版本的源码及工程文件，在build目录下有VS2010的工程文件（当然，你也可以选择利用CMake自己本地创建工程），打开配置好flatc工程的传入参数，如下：
-c -o ./ ./TestFlat.fbs
运行flatc工程，即可在当前工程目录下生成TestFlat_generated.h头文件，这个头文件中包含了我们所需的所有结构体、枚举类型等以及相应的存取方法和验证方法

- 使用FlatBufferBuilder类创建flat的二进制buffer
以下代码展示了如何利用FlatBufferBuilder创建相应buffer：

```c++
 //read serialized buffer
 flatbuffers::FlatBufferBuilder builder_data;

 int test_append = 300;
 auto name_test = builder_data.CreateString("TestAppend");
 auto testApp = CreateTestAppend(builder_data, test_append, test_append);

 int image_size = 12;
 unsigned char inv_data[] = { 11, 2, 4, 2, 10, 3, 5 ,7, 10, 39, 45, 23 };
 auto name = builder_data.CreateString("TextureData");
 auto image_data = builder_data.CreateVector(inv_data, image_size);

 int image_test = 900;
 auto texture_data = CreateTextureData(builder_data, image_size, image_data, image_test, image_test);

 //flatbuffers::FlatBufferBuilder builder_tex;
 int texture_num = 1;

 auto name_tex = builder_data.CreateString("Texture");

 vector<flatbuffers::Offset<TextureData>> tex_vec;
 tex_vec.push_back(texture_data);
 auto tex_data = builder_data.CreateVector(tex_vec);

 int num_text = 100, num_text2 = 200, num_text3 = 300;
 auto texture = CreateTexture(builder_data, texture_num, tex_data, num_text, num_text2, testApp);
 builder_data.Finish(texture);
```

要使上述正确运行，除了引用C++基本库之外，需在文件头部添加以下代码：

```c++
#include "flatbuffers/flatbuffers.h"
#include "idl.h"
#include "util.h"
#include "TestFlat_generated.h"
using namespace Jason::Flat::Test;
```
上述代码的编写中规中矩，其中CreateString和CreateVector都是FlatBufferBuilder类的成员函数，分别用于创建适用于FlatBuffer内存结构的字符串数据以及向量数据。其余的方法，如CreateTextureData、CreateTexture均是由flatc根据IDL文件（TestFlat.fbs）自动生成的头文件中用于创建相应结构体的函数。最后一句builder_data.Finish(texture)用于优化对齐写入builder_data的内存结构。

- 保存buffer到本地或者直接通过网络发送
保存buffer到本地的代码，如下：

std::cout << builder_data.GetSize() << std::endl;
flatbuffers::SaveFile("texture.bin", reinterpret_cast<char *>(builder_data.GetBufferPointer()), builder_data.GetSize(), true);
将数据保存到名为texture.bin的二进制文件中，其中通过builder_data.GetBufferPointer()获取内存指针，builder_data.GetSize()获取内存大小，最后一个参数用于制定是否生成二进制文件。

- 接收并buffer并读取数据内容
读取二进制文件的代码如下：

```c++
 string binaryfile;
 bool ok = flatbuffers::LoadFile("texture.bin", false, &binaryfile);

 flatbuffers::Verifier tex_verify(builderOut.GetBufferPointer(), builderOut.GetSize());
 bool verify_flag = VerifyTextureBuffer(tex_verify);

 flatbuffers::FlatBufferBuilder builderOut;
 TextureBuilder* texBuilder = new TextureBuilder(builderOut);
 builderOut.PushBytes(reinterpret_cast<unsigned char*>(const_cast<char *>(binaryfile.c_str())), binaryfile.size());
 std::cout << builderOut.GetSize() << std::endl;

 auto model = GetTexture(builderOut.GetBufferPointer());

 int outNum = model->num_textures();
 const flatbuffers::Vector<flatbuffers::Offset<TextureData>>* outTex = model->textures();
 TextureData* outTexData = (TextureData *)outTex->Get(0);
 int outSize = outTexData->image_size();
 const flatbuffers::Vector<unsigned char>* outData = outTexData->image_data();
 int x = outData->Get(5);
 int len = outData->Length();
 delete texBuilder;
```
上述代码中VerifyTextureBuffer用于验证读取的内存是否为FlatBuffers的内存块，是则返回true，不是则返回false。通过GetTexture获取指针之后，结构体中的变量均可以通过相应方法（各方法名请查看自动生成的头文件）获取。

### 总结
利用FlatBuffers来进行数据保存及传输的优点显而易见，它利用自身特殊的编码格式，能一定程度上减少内存的占用，优化读取的性能。更重要的是，对于数据结构的向前向后兼容提供了很好的扩展性，方便又高效：

- 要想让数据结构具有可扩展性，需将数据结构定义为table，它是数据扩展的基础，FlatBuffers中的struct类型不支持扩展
- 如果想在后续的版本中删除数据结构中的某些字段，只要在将要删除的字段后面添加(deprecated)即可，当然需要保证删除的字段在之前版本的程序中不会引起程序崩溃（该删掉的字段在上一版本的程序中获取到的会是个空指针或空值，只需保证程序在获取到空值或空指针之后不会出现异常即可）
- 如果想在后续版本中向数据结构中添加某些字段，需添加到table中最后一个字段的后面，若是想table中随意位置添加字段，需如上面TextureData 的定义，给每个字段指明添加id:n（n从0开始）
目前FlatBuffers还不是很完善，碰到问题可以到FlatBuffers Issues Tracker去提交或则寻找答案。

From: http://www.jianshu.com/p/6eb04a149cd8

