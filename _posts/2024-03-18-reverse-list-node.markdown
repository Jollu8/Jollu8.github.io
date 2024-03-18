---
layout: post
title: "Reverse Linked List"
subtitle: "Эволюция решения задачи обращения односвязного списка: от классического подхода к элегантности"
date: 2024-03-18
author: "Jollu8"

tags:
  - Algorithms
  - LinkedList
  - C++
---


Вот знаете, даже простые задачи со своим решением восхищают. Когда задача становится простой? Когда вы решаете задачи такого шаблона, как щелкаете арахис.

В свободное время я обычно решаю задачки. Решил пройти все задачи по односвязному списку.

```cpp
struct ListNode {
    int val;
    ListNode *next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode *next) : val(x), next(next) {}
};
```

Есть такая задача на LeetCode `206. Reverse Linked List`. Она действительно забавная.

Вот знаете, я в 2022 году решил эту задачу вот так:

```cpp
ListNode* reverseList(ListNode* head) {
    ListNode *p=NULL;
    ListNode *c=head;
    ListNode *n=NULL;
    while(c!=NULL){
        n=c->next;
        c->next=p;
        p=c;
        c=n;
    }
    head=p;
    return head;
}
```

Но в прошлом году решил вот так:

```cpp
ListNode* reverseList(ListNode*& head) {
    if(!head) return head;

    stack<ListNode*>s;
    while(head){
        s.push(head);
        head = head->next;
    }

    head = s.top();
    s.pop();
    auto tmp = head;

    while(!s.empty()) {
        auto curr = s.top();
        s.pop();
        tmp->next = curr;
        tmp = tmp->next;
    }
    tmp->next = nullptr;
    return head;
}
```

Я специально выбрал другой подход, чтобы попробовать.

Вот, на днях очередной раз решаю задачу (другую задачу). После решения задачи я обычно смотрю решения, чтобы посмотреть, как решили эту задачу другие участники.

В этот момент я наткнулся на объяснение своего решения уважаемого @votrubac (ник участника LeetCode).

Вы даже сможете такое сразу представить на уме. Так как его решение настолько элегантно, что я полюбил сразу.

Вот и само его решение:

```cpp
ListNode *reverse(ListNode *h) {
    ListNode *cur = h, *prev = nullptr;

    while (cur) {
        swap(cur->next, prev);
        swap(prev, cur);
    }
    return prev;
}
```

Разве не красиво?
