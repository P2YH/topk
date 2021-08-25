1. 尽早释放无用对象的引用
2. 尽量避免使用String，使用StringBuffer代替
3. 尽量少用静态变量
4. 避免集中创建对象尤其是大对象，如果可以尽量使用流处理
5. 运用对象池技术提高系统性能
6. 不要在经常调用的方法中创建对象，尤其是在循环中。
7. 优化配置：
   1. -Xms，-Xmx相等
   2. NewSize,MaxNewSize相等
   3. 调整Heap 和 PermGen space大小