+++
title = 'Code.md'
date = 2023-11-09T15:56:46+08:00
draft = true
+++

链表逆序

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

