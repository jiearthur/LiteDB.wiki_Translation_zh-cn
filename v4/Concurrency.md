LiteDB v4 同时支持线程安全和进程安全：

- 你可以在每次使用时创建 `LiteRepository`, `LiteDatabase` 或 `LiteEngine` 的一个新实例 (进程安全)
- 你可以在你的线程间共享单个 `LiteRepository`, `LiteDatabase` 或 `LiteEngine` 实例 (线程安全)

在第一个选项 (进程安全) 中，总是会从数据文件断开。每次使用时，先打开数据文件，然后锁定文件 (读或写模式)，然后执行你的操作，最后关闭数据文件。锁使用 `FileStream.Lock` 实现，可同时用于读/写模式。在这种方式下，最好总是使用 `using` 声明来关闭数据文件。

在第二个选项 (线程安全) 中，LiteDB 使用 .NET 类 `ReaderWriterLockSlim` 来控制并发。使用此类，可以管理多个读操作和一个独占的写操作。所有线程共享相同的实例，每个方法内控制并发性。 

## 建议

单实例比多实例快得多。在多实例环境下，每个实例必须做昂贵的数据文件操作：打开、锁定、解锁、读、关闭。每个实例也有它们自己的缓存控制，而且这些只是用于一个单独的操作，在关闭数据文件时，所有缓存页面都会被抛弃。而在单实例中，缓存中的所有页面在所有读线程之间共享。

如果你的应用以单进程 (像移动应用、asp.net 网站) 模式运行，更倾向于使用一个单数据库实例，并在所有线程间共享。

你可以使用 `Exclusive` 模式 (在连接字符串中)。使用独占模式可以防止数据文件一直检查头部页面来确定是否有外部变化，同时，独占模式不用在文件上使用 Lock/Unlock，而只需要在内存 (使用 `ReaderWriterLockSlim` 类) 中。
