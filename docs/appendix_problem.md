# LabVIEW 的一些问题

这一节，记录笔者在使用 LabVIEW 过程中遇到的一些问题，以及 LabVIEW 一些不太完善的方面。

## LabVIEW 的 unicode 问题

笔者最近试图去查看自己很久之前写的一些 VI，但是却发现那些 VI 要么无法打开，要么打开后，里面的中文字符全部成了乱码。造成个这问题的根本原因是因为 LabVIEW 不支持 unicode 编码。

先简略的介绍一下 unicode 的背景：

计算机是在美国被发明的，所以当时很自然的就只考虑了处理英文。最早出现的一个，也是最著名的一个关于在计算机中表达字符的标准，是 ASCII 标准（American Standard Code for Information Interchange，美国信息互换标准代码）。它定义了128个字符，包括英文字母大小写，数字，常用的标点和一些特殊符号。当时世界上所有的计算机都在使用 ASCII 方案来保存英文文字。这个方案最大的问题是它只包含了英文字母，于是其它国家，组织和公司纷纷开始扩展这个标准，用以支持其它字符，比如制表符、数学运算符号、中文字符、日文字符等等。在中国，最常用的标准，包括 GB2312，GBK，GB18030 等都是对 ASCII 的中文扩展。这些扩展出的标准有一个很麻烦的问题：同一个数值在不同的标准中被定义了不同的含义。比如，某一数值在中文的标准下可能是一个中文字符，在韩文的标准下，可能就是一个完全不相关的制表符。这就造成了，在中文环境下开发的软件，运行到韩文系统下，显示的就完全是乱码。如果有人需要在一个系统下同时运行一个中文软件和一个韩文软件，就只能 有一个软件可以正确显示文字。

为了解决这个问题，1990年开始，计算机业界开始研发一种新的编码标准，它可以覆盖全世界所有的字符，也就是说，任何一个字符都有自己独占的编码，在任何系统下都会保持这个编码，这样就不会出现换个系统就乱码的问题了。这就是 unicode 编码，也叫万国码、单一码。unicode 规定了字符集，但是这套字符集也还存在多种不同的编码格式。Windows 采用了 UTF-16LE 格式的 unicode 编码（UTF全称为 Unicode Transformation Format），使用 16 位的(双字节)数据表示一个字符。但是当前最流行的 unicode 编码格式却是 UTF-8，这是一种变长的编码方式，根据字符的常用程度，使用不同长度的编码来表示这个字符，编码长度有可能是 1 到 6 个字节。目前大多数的 unicode 文档采用的都是 UTF-8 编码。

经过几十年的推广和发展，现在已经很少有不支持 unicode 的主流软件了。但是，非常遗憾的是 LabVIEW 始终没有支持 unicode。在整个2010年代，NI 公司的重点放在了开发支持 unicode 的 LabVIEW NXG。计划用它彻底取代 LabVIEW，所以没有投入足够的资源去改进 LabVIEW。然而，NI 公司又放弃了 LabVIEW NXG。不支持 unicode 的 LabVIEW 成了使用者的唯一选择。

虽然 Windows 很早就开始支持 unicode 了，但是为了兼容那些还没有支持 unicode 的软件，Windows 系统保留了一个默认字符集的设置，对于非 unicode 的软件，Windows 会使用默认的字符集来解释那些字符编码。国内使用的电脑几乎都把默认的字符集设置为中文字符集，所以，一个软件是不是支持 unicode，对于绝大多数用户来说，并不能感觉到什么差别。笔者之前也从来没意识到 LabVIEW 不支持 unicode 会有什么问题。

直到最近，笔者才发现了这个问题。笔者家里有两台电脑，一台操作系统安装了 Windows；另一台操作系统是中文 Deepin Linux。两台电脑上都安装了社区版的 LabVIEW 2021。笔者发下，在 Windows 系统下编写的 VI，如果包含中文常量或注释，拿到 Linux 系统下打开，看到的就全部是乱码；反之亦然，在 Linux 系统下，在 VI 上添加中文注释，到 Windows 系统下打开看到的也是乱码。更严重的是，如果项目中包含有中文名的子VI，或是库中、类中有中文名的VI，拿到另一个系统下就根本无法打开了。

造成这些问题的根源在于 LabVIEW 没有直接支持 unicode，它总是采用操作系统的默认字符编码。在中文 Windows 系统下使用 GB18030 中文编码；在 Deepin Linux 系统下使用 UTF-8 编码。

在 Linux 系统下，由于系统和 LabVIEW 采用的都是 UTF-8 编码，所以一个 VI 在不同 Linux 版本下的行为和显示内容都是一致的。但 Windows 系统本身采用的是 unicode，为应用程序提供的默认编码，也就是 LabVIEW 使用的字符编码却不是 unicode。假设，在一个项目中有两个 VI，分别是“界面.vi”和“任务1.vi”，其中“任务1.vi”是“界面.vi”的子 VI。在操作系统层面，有两个文件：“界面.vi”和“任务1.vi”，它们的名字都是使用 unicode 编码保存的。但是在“界面.vi”内部，它记录了自己要调用“任务1.vi”，用的却是非 unicode 编码（比如在中文 Windows 系统下使用 GB18030 中文编码）保存的子 VI 的名字。所以 LabVIEW 中用于记录这个子 VI 的名字的一段二进制数据与操作系统中记录这个子 VI 文件名使用的二进制数据是不同的。每次 LabVIEW 需要操作系统装载一个子 VI 文件时，还需要做一次编码转换，把文字转为系统使用的 unicode 编码。如果保存 VI，和读取 VI 都是在中文 Windows 中，编码始终一致，不会有任何问题，VI 名字总能被正确转换。但是把之前在中文 Windows 系统下保存的项目拿到非中文 Windows（默认语言编码不再是 GB18030）下打开，文件名编码转换这一步就会出错（用其它编码解释 GB18030 格式的数据），系统得不到正确的 VI 名，也就无法找到正确的 VI。同理，中文 Windows 下保存的 VI，如果有中文名，拿到 Linux 系统下打开会出错；反过来也是一样会出错。

总之，由于 LabVIEW 在 Windows 下没有使用 unicode，造成了中文（或其它非英语文字）在不同系统下显示行为的不一致，切换系统就会出现乱码，甚至 VI 无法被加载。目前，笔者也没有找到好的办法解决这个问题。笔者只能尽量在 LabVIEW 中只使用英文，不使用中文以及特殊字符。