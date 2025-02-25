# nvidia-smi卡顿

如果显卡数量在4张以上，在nvidia-smi信息后会非常的慢，非常的卡。尤其在只在乎计算量服务器的时候。

我试过把8张卡 tesla K80 显卡一个个拆下来 8张、7张 6/5/4/3/2/1 试试nvidia-smi 结果速度都一样。

需要4到5分钟时间，甚至都不出直接死机。

解决方法

但是使用nvidia-smi之后可以显示GPU信息，只是疑惑Persistence-M为什么是off呢。。。然后就将其状态改为ON试试，就可以了。显卡打开长启动，关闭节能模式

```
nvidia-smi -pm 1
```

进入正题如下命令，可以加速nvidia-smi的加载速度，打开GPU持久模式

```
sudo nvidia-persistenced --persistence-mode
```

————————————————

