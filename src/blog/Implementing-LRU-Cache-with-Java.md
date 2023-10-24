---
title: Implementing LRU Cache
date: 2023-10-24 03:31:57
author: Bhupendra Parihar
tags: ["post", "featured"]
image: /assets/blog/article-2.jpg
description: LRU Cache is a in memory cache with some fixed size and it will only contain item which are recently used. Good to know this type of problem for Interview.
---

## What is Cache?

- Fetching data from a harddisk or database is little expensive, what if we can store the frequently used item in memory(dynamic memory) which gives the result in O(1) time. It is same as we memorize 5+5 = 10, and we have done this so many times, that we dont even calculate these basic calculations. So our brain can do difficult calculations but it also memorize the frequently accessed item, and it also forgot the information if you don't use it for a long time.

## What is LRU Cache?

- LRU stands for Least Recently Used. It is kind of eviction policy. Like our brain, if some information is not used for very long time, our brain forgot it. It is happening because our brain has a limited dynamic(quick time) memory. Like wise, we have a limited dynamic memory(RAM) and our program can only store that much information. So what we want is to elimiate the least recently used item from the memory if our memory is full for the next item.

## What kind of Data Structure is required here?

There are two requirement - one is to access the information in O(1) time, and other is to remove the least recently used item.

- for O(1) time we are going to use HashMap
- and for maintaing a which item is being recently used, we are going to use a 2-way linkedlist

Lets assume the size of our Cache is 4.

![Alt text](/assets/blog/lrucache.svg)

## Source Code

Create a Node class first

```java
public class Node {
    private final String key;
    private String value;

    private Node prev;
    private Node next;

    public Node(String key, String value) {
        this.key = key;
        this.value = value;
    }

    public Node getNext() {
        return this.next;
    }

    public void setNext(Node n) {
        this.next = n;
    }

    public Node getPrev() { return this.prev; }

    public void setPrev(Node n) {
        this.prev = n;
    }

    public String getValue(){
        return this.value;
    }

    public void setValue(String value) {
        this.value = value;
    }

    public String getKey() {
        return this.key;
    }
}
```

Since the above Node object is a two way linked list, we have next and previous.

Now we are going to create a class LRUCache with size property

```java
public class LRUCache {
    private final Map<String, Node> map;
    private final int capacity;

    private Node head = null;
    private Node tail = null;

    public LRUCache(int capacity) {
        this.map = new HashMap<String, Node>();
        this.capacity = capacity;
    }
}
```

Two important function exposed from LRUCache to the outside world is put and get

```java
public void put(String key, String value){
    if(map.containsKey((key))){
        Node node = map.get(key);
        node.setValue(value);

        deleteFromList(node);
        setListHead(node);
    } else {
        if(map.size() >= capacity) {
            map.remove(tail.getKey());
            deleteFromList(tail);
        }

        Node node = new Node(key, value);

        map.put(key, node);
        setListHead(node);
    }
}

 public String get(String key) {
    if(!map.containsKey(key)){
        return null;
    }

    Node node = map.get(key);

    deleteFromList(node);
    setListHead(node);

    return node.getValue();
}
```

As you can see both these function update the Hashmap along with they call two other function named deleteFromList and setListHead to maintain the doubly linked list.

```java
 private void deleteFromList(Node node) {
    if(node.getKey().equals(head.getKey())){
        if(node.getKey().equals(tail.getKey())){
            head = null;
            tail = null;
        }else {
            head = node.getNext();
            node.setNext(null);
            head.setPrev(null);
        }
    } else if(node.getKey().equals(tail.getKey())) {
        tail = node.getPrev();
        tail.setNext(null);
        node.setPrev(null);
    } else {
        Node current = head;
        while(current != null) {
            if(current.getKey().equals(node.getKey())){
                current.getPrev().setNext(current.getNext());
                current.getNext().setPrev(current.getPrev());
                current.setNext(null);
                current.setPrev(null);
                break;
            }
            current = current.getNext();
        }
    }
}

private void setListHead(Node node){
    node.setNext(null);
    node.setPrev(null);

    if(head == null) {
        head = node;
        tail = node;
    } else {
        node.setNext(head);
        head.setPrev(node);
        head = node;
    }
}
```

I have also created a print function which prints the status of LRUCache.

```java
public void print() {
    System.out.println("Printing Map");
    map.forEach((key, value) -> System.out.println(key + " : " + value));

    System.out.println("Printing LinkedList \n");
    Node current = head;
    while(current != null) {
        System.out.print(current.getKey() + " --> ");

        current = current.getNext();
    }
    System.out.println("null");
}
```

Lets test this class in our Main class.

```java
//Main.java

public class Main {
    public static void main(String[] args) {
        LRUCache cache = new LRUCache(4);
        cache.put("A", "Arthur");
        cache.put("B", "Bob");
        cache.put("C","Carter");
        cache.put("D","Derek");

        cache.print();

        cache.put("E", "Ernest");
        cache.put("F", "Frank");
        cache.put("D","Don");

        cache.print();
        System.out.println(cache.get("D"));

        cache.print();
        System.out.println(cache.get("E"));
        System.out.println(cache.get("E"));
        cache.print();
    }
}
```

You can find the code from this github repo [https://github.com/bp-learning/LRUCache](https://github.com/bp-learning/LRUCache)

