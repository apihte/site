+++
title = 'Code.md'
date = 2023-11-09T15:56:46+08:00
draft = true
+++

## 链表逆序

```java
public class Node {
	int id;
	Node next;

	public Node(int id) {
		this.id = id;
	}
}

public Node createList(int size) {
    Node head = null;
    Node newNode = null;
    for (int i = 0; i < size; i++) {
        if (head == null) {
            head = new Node(i + 1);
            newNode = head;
            continue;
        }
        newNode.next = new Node(i + 1);
        newNode = newNode.next;
    }
    return head;
}

public Node reverse(Node head) {
    if (head == null || head.next == null) {
        return head;
    }
    Node temp = head.next;
    Node newHead = reverse(head.next);
    temp.next = head;
    head.next = null;
    return newHead;
}
```

## 多线程顺序输出

```java
public class Printer implements Runnable {
    private Object lock = new Object();

    private int i = 1;
    private int max;

    public Print(int max) {
        this.max = max;
    }

    @Override
    public void run() {
        while (i <= max) {
            synchronized (lock) {
                lock.notify();
                System.out.println(i++);
                try {
                    if (i <= max) {
                        lock.wait();
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

public static void main(String[] args) {
    Printer printer = new Printer(10);
    new Thread(printer, "t1").start();
    new Thread(printer, "t2").start();
}
```

## id 分组

```java
public int getUserGroup(long userId, String group) {
    String newUser = userId + group;
    return newUser.hashcode() % 100;
}
```

## 二分查找

```java
public static void binaraySearch(int[] arr, int value) {
	int right = arr.length;
	int left = 0;
	int mid;

	while (true) {
		mid = (right + left) / 2;
		if (arr[mid] == value) {
			System.out.println("mid = " + mid);
			return;
		} else if (arr[mid] < value) {
			left = mid + 1;
		} else if (arr[mid] > value) {
			right = mid - 1;
		}
		if (left >= right) {
			System.out.println("null");
			return;
		}
	}
}
```



