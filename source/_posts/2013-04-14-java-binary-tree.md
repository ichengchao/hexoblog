---
layout: post
title: java实现的四则运算解析器
date: 2013-04-14 16:19
tags:
  - java
  - tech
---
学过编译原理的同学都知道有最开始的一步就是词法分析.  
下面就是一次入门级的实践.(看之前需要了解一下什么是二叉树)  

###主要有三个类:###

- Node.java 二叉树的java model

```java
public class Node {
 
    Node     left  = null;
    Node     right = null;
    String   value;
    NodeType nodeType;
    // 是否在括号中
    Boolean  isBlock;
 
    public Node(String value, NodeType nodeType){
        super();
        this.value = value;
        this.nodeType = nodeType;
        this.isBlock = false;
    }
 
    public Node(String value, NodeType nodeType, Boolean isBlock){
        super();
        this.value = value;
        this.nodeType = nodeType;
        this.isBlock = isBlock;
    }
 
    public Node(){
        super();
    }
 
    @Override
    public String toString() {
        return value + ":" + nodeType;
    }
 
    public void setIsBlock(Boolean isBlock) {
        this.isBlock = isBlock;
    }
 
    public boolean isHigerThanOther(Node other) {
        if (other.isBlock) {
            return false;
        }
        if (("*".equals(value) || "/".equals(value)) && ("+".equals(other.value) || "-".equals(other.value))) {
            return true;
        }
        return false;
    }
 
}
```

- NodeType.java 节点的枚举类型,四则运算中有数字和运算符两种

```java
public enum NodeType {
    num, op;
}
```

- Parser.java 关键类,paser的实现

```java
public class Parser {
 
    private char[] ca;
    private int    total;
    private int    point = 0;
 
    public Parser(String s){
        s = s.replaceAll("\\s+", "");
        ca = s.toCharArray();
        total = ca.length;
    }
 
    public Node parse() {
        Node root = getNode();
        while (true) {
            Node node = getNode();
            if (null == node) {
                break;
            }
            if (!NodeType.op.equals(node.nodeType)) {
                throw new RuntimeException("格式错误!");
            }
            Node nextNode = getNode();
            if (nextNode == null) {
                throw new RuntimeException("操作符后面不能为空!");
            }
            node.right = nextNode;
            if (node.isHigerThanOther(root)) {
                node.left = root.right;
                root.right = node;
            } else {
                node.left = root;
                root = node;
            }
        }
 
        return root;
    }
 
    /**
     * 获取单个node
     *
     * @return
     */
    Node getNode() {
        String v = "";
        while (true) {
            if (point == total) {
                break;
            }
            char c = ca[point];
            point++;
            if (c == '(') {
                String sub = subStringInBracket();
                Parser parser = new Parser(sub);
                Node subNode = parser.parse();
                subNode.setIsBlock(true);
                return subNode;
            }
            if (isOp(c)) {
                if ("".equals(v)) {
                    return new Node(String.valueOf(c), NodeType.op);
                } else {
                    point--;
                    return new Node(v, NodeType.num);
                }
            }
            v += c;
        }
        return "".equals(v) ? null : new Node(v, NodeType.num);
    }
 
    public boolean isOp(char c) {
        return c == '+' || c == '-' || c == '*' || c == '/';
    }
 
    /**
     * 获取括号中的字符串
     *
     * @return
     */
    public String subStringInBracket() {
        StringBuilder sb = new StringBuilder();
        if (ca[point - 1] != '(') {
            throw new RuntimeException("4");
        }
        int bracketFlag = 0;
        for (;;) {
            char c = ca[point];
            if (c == ')' && bracketFlag == 0) {
                point++;
                return sb.toString();
            }
            if (c == '(') {
                bracketFlag++;
            }
            if (c == ')' && bracketFlag > 0) {
                bracketFlag--;
            }
            sb.append(c);
            point++;
            if (point == total) {
                throw new RuntimeException("找不到')',格式错误");
            }
        }
    }
 
    /**
     * 递归计算node的值
     *
     * @param node
     * @return
     */
    public Integer getResult(Node node) {
        Integer left = null;
        Integer right = null;
        if (NodeType.num.equals(node.left.nodeType)) {
            left = Integer.valueOf(node.left.value);
        } else {
            left = getResult(node.left);
        }
        if (NodeType.num.equals(node.right.nodeType)) {
            right = Integer.valueOf(node.right.value);
        } else {
            right = getResult(node.right);
        }
 
        String op = node.value;
 
        if ("+".equals(op)) {
            return left + right;
        }
        if ("-".equals(op)) {
            return left - right;
        }
        if ("*".equals(op)) {
            return left * right;
        }
        if ("/".equals(op)) {
            return left / right;
        }
        throw new RuntimeException("3");
    }
}
```

- ParserTest.java 测试

```java
public class ParserTest extends TestCase {
 
    public void test_getResult() {
 
        Parser parser = new Parser("1+2+3");
        assertEquals(6, parser.getResult(parser.parse()).intValue());
 
        parser = new Parser("1*2*3");
        assertEquals(6, parser.getResult(parser.parse()).intValue());
 
        parser = new Parser("4/2/1");
        assertEquals(2, parser.getResult(parser.parse()).intValue());
 
        parser = new Parser("1+2*3-1+100/2");
        assertEquals(56, parser.getResult(parser.parse()).intValue());
 
        parser = new Parser("(1+2)*3/9");
        assertEquals(1, parser.getResult(parser.parse()).intValue());
 
        parser = new Parser("(1+2)*(2+3)/(5-2)");
        assertEquals(5, parser.getResult(parser.parse()).intValue());
 
        parser = new Parser("200*73/(543-178)");
        assertEquals(40, parser.getResult(parser.parse()).intValue());
 
        parser = new Parser("200*73/(543-178)-(2+2)/2*(((4-1)*2+(3-1))+1)");
        assertEquals(22, parser.getResult(parser.parse()).intValue());
 
        parser = new Parser("(2+2)/2*(((4-1)*2+(3-1))+1)");
        assertEquals(18, parser.getResult(parser.parse()).intValue());
 
        parser = new Parser("200*73/(543-178)-(2+2)/2*(((4-1)*2+(3-1))+1)-(1+2)*(2+3)/(5-2)");
        assertEquals(17, parser.getResult(parser.parse()).intValue());
 
        parser = new Parser("(((((1+2)*(2+3)/(5-2)+1)*(1+1))/4-1)*3+4)*10");
        assertEquals(100, parser.getResult(parser.parse()).intValue());
    }
 
}
```

<!--
<script src="https://gist.github.com/ichengchao/ee5f06f0ee03766dc750.js"></script>
-->