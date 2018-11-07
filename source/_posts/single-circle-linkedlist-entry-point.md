title: 如何求单链表的环入口点 - 龟兔赛跑法 - 动画解释
date: 2018/11/6 13:42:00
tag:
	- Algorithm

---

当单向链表中存在环的时候，遍历此链表会发生无限循环，无法到达末尾（入环后链表就不存在末尾了）的情况，所以在可能发生这种情况的时候，需要检查链表中是否存在一个环。

检查是链表是否存在环的方法就是「**龟兔赛跑**」法：乌龟和兔子同时从头节点开始遍历链表，兔子遍历的速度大于乌龟的速度，如果链表中存在环，兔子和乌龟就会先后进环，由于兔子的速度比乌龟快，他们必然会在环内相遇。

下面的动图解释了这一过程：

<!--more-->

![](http://data.kchen.cc/mac_af-b9a73ed3596db8a22fbc97f6f6abc35f.gif-origin)

这个算法的美妙之处在于，乌龟和兔子相遇的地方和环入口点的位置是有关系的。

根据动画所示，我们令兔子的遍历速度为2，乌龟的遍历速度为1，则他们的速度差也为1。设乌龟进环的时候已经遍历了 {% math %}x{%endmath%} 个节点，那么此时兔子也已经在环内遍历了 {% math %}x{%endmath%} 个节点。若令环的大小为 {% math %}y{%endmath%}，兔子和乌龟在环内的遍历就是一次追及问题，兔子需要追上乌龟的距离为 {% math %}y-x{%endmath%}。由于兔子和乌龟的速度差为1，所以追及时间 {% math %}t=(y-x)\div1=y-x{%endmath%}，那么乌龟和兔子相遇的点距离环入口也就是 {% math %}x{%endmath%} 了。此时只需要将兔子放回起点，并把兔子的遍历速度换成1，则乌龟将会和兔子在环入口处再次相遇。

最后附上 Java 算法：

```java
LinkedListNode solve(LinkedListNode head) {
    LinkedListNode rabbit = head;
    LinkedListNode turtle = head;

    while (rabbit != null && rabbit.next != null) {
        rabbit = rabbit.next.next;
        turtle = turtle.next;
        if (rabbit == turtle) break;
    }

    if (rabbit == null || rabbit.next == null) return null;

    rabbit = head;
    while (rabbit != turtle) {
        rabbit = rabbit.next;
        turtle = turtle.next;
    }
    return rabbit;
}
```