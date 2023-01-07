---
title: "线性表C实现"
date: "2020-11-30"
categories:
  - 数据结构

toc: true
---

线性表的顺序存储结构是指用一组地址连续的存储单元依次存放线性表的元素。为了使用顺序结构实现线性表，程序通常会采用数组来保存线性表中的数据元素。

<!--more-->


```c
#define ListSize 100    // 自定义长度100
#define DataType int    // 自定义节点类型

// 顺序表结构
typedef struct {
    DataType data[ListSize]; 	// 数组data存放表结点
    int length;					// 线性表的当前表长（实际存储元素的个数）
} SeqList;


int main () {
    SeqList a = {{1,2,3,4,5}, 5};
    show(a);

    insert(&a, 1, -1);
    show(a);

    delete(&a, 1);
    show(a);

    convert(&a);
    show(a);

	return 0;
}

// 第i个元素插入v，原有>=i的元素向后移1个， i从1开开始
void insert(SeqList *L, int i, DataType v) {
    if (i <= 0 || i > L->length) {
        printf("插入位置错误");
        exit(0);
    }
    if (L->length == ListSize) {
        printf("表已满");
        exit(0);
    }

    for (int j = L->length; j >= i; j--) {
        L->data[j] = L->data[j-1];
    }
    L->data[i - 1] = v;
    L->length++;

    return;
}

// 删除第i个元素
DataType delete(SeqList *L, int i) {
    if (i <= 0 || i > L->length) {
        printf("元素不存在");
        exit(0);
    }

    DataType v = L->data[i - 1];
    for (int j = i - 1; j < L->length - 1; j++) {
        L->data[j] = L->data[j+1];
    }
    L->length--;
    return v;
}

// 反转顺序表
void convert(SeqList *L) {
    int middle = L->length / 2;
    DataType temp;

    for (int i = 0; i <= middle; i++) {
        temp = L->data[i];
        L->data[i] = L->data[L->length - 1 - i];
        L->data[L->length - 1 - i] = temp;
    }
    return;
}

// 打印
void show(SeqList L)
{
    printf("长度：%d", L.length);
    printf("，数据：");
    for (int k = 0; k < L.length; k++) {
        printf("%d ", L.data[k]);
    }
    printf(" \n");
}
```
